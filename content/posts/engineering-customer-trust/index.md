---
title: "Engineering for Customer Trust: Keeping Product Reviews Accurate and Current"
date: 2026-09-15T18:04:32+02:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - AWS
  - System Architecture
tags:
  - AWS
  - Queues
  - Serverless
  - NoSQL
  - Caching
---

{{< img src="images/article-header.png" alt="A product listing page excerpt with reviews" caption="A product listing with reviews">}} 


In the highly-subjective e-commerce world, social proof plays a major role in shaping buyer decision.

As a customer, when I land on a Product Listing Page (PLP) with twenty different products matching my search criteria, I really don't want to open ten tabs to find the best one, I would prefer leveraging on the experience of customers who have bought same or similar products instead.
Research data has shown that surfacing social proof early in the customer journey potentially signals high transparency, reduces cognitive load and accelerates decision-making which ultimately improves core business metrics such as Click-Through Rate (CTR).

This article covers the engineering thinking and method behind my team's quest to make this happen, seamlessly.
As the lead engineer on this initiative, I'll walk through how a single user's review moves through the pipeline: from the moment it arrives to the moment its aggregate score is refetched and the system of record is updated. This system of record is where the PLPs source review data from.

---

Before I go into the details, I must say that this is a relatively technical article. It discusses infrastructure concepts implemented using mostly AWS Cloud tooling.

As a primer, the articles linked below give a great introduction to some of the tools we used as well as the respective roles they play in this achieving the objectives of the overall system we're about to discuss. 

- Event Pub/Sub - [AWS EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)

- Serverless Functions - [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
  
- NoSQL Database - [AWS DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
  
- Queues - [AWS Simple Queue Service (SQS)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)

- In-memory Caches - [Redis (via AWS Elasticache)](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html)

---

### 1. What Are We Solving For?

In the context of this domain, review ratings (stars) behave like product attributes. They sit next to price and availability on the product card, they appear on listing pages where dozens of products render at once. They can be sorted, filtered and searched like any other facet. 

However, unlike price and availability, they live in a rate-limited 3rd Party API out of our direct control.

The goal is to own this review data in order to enable these capabilities, while keeping latency low and the data as fresh as needed, across every country (store) we operate in - 19 at the time of this writing.


{{< img src="images/architecture-diagram.png" alt="architecture diagram" caption="service components are hosted on AWS cloud">}} 


### 2. The Backbone at a Glance

As seen in the above diagram, three Lambda runtimes share one codebase. 
- An **HTTP** lambda receives webhooks and admin calls
- A **worker** lambda drains an SQS queue
- A **cron** lambda runs the scheduled aggregation.

They're composed per-entry-point with dependency injection, maintaining one deploy and one source of truth for the domain logic, rather than three services that drift out of sync. 
Redis caches auth tokens from the reviews provider (a 3rd-party service) across all of them, and DynamoDB is the durable store.

### 3. Bootstrapping the Integration: Webhook registration
First we expose our webhook intake URL to the 3rd-party reviews provider. This is then used to set up Webhook subscriptions per store. Whenever a new review is submitted in any of the subscribed stores, the associated intake url is triggered.

### 4. Webhook Intake
Once subscriptions exist, events flow to the intake url. The intake handler does as little as possible: validate the payload, decide whether the event matters, record it, and acknowledge fast. 

Irrelevant review events (e.g a reported review) are dropped as they don't change the aggregate data being catered to by this flow. For qualifying events, the variant-sku (a subset of the product data) is extracted and saved to **a Redis set**. The Set acts as the first line of deduplication: ten reviews belonging to the same variant are represented as a single set entry. That's all about the Intake logic, no DB or 3rd party calls.


### 5. The Processing Cycle: Cron, a second dedup, and fan-out
The intake component keeps track of *which* variants got new reviews. However, the real work is driven by the **cron** lambda, which is triggered by an EventBridge task on a set schedule (e.g daily or weekly).
When invoked, this lambda atomically rotates the intake set into a processing set and runs a **second level of deduplication**: collapsing variants up to their *product-level*, the parent that groups a family of variant SKUs.

This second-level dedup is a key component of the design as the reviews provider aggregate review data at the product level, using a concept of the 'master' variant.

Also, since reviews are represented at the product-level, our internal system of record in this design captures review data at the same level. A product with fifty reviewed variants becomes a single unit of work carrying one variant that represents it. This essentially collapses dozens of raw webhook calls into one call per affected product.

The resulting per-product payloads are then fanned out as an **SQS batch**. 

A **Worker Lambda** drains the queue. It uses the representative variant data to fetch the product's review stats from the reviews provider, and then writes it to DynamoDB. Failures here are captured via a DLQ, which get inspected and re-driven when needed.


> **Why fan out over SQS instead of one long cron invocation.**
> 
> _A single Lambda looping over every product (there are over 50k of those per store) would press against the 15-minute ceiling. Fanning the deduplicated units onto SQS makes each independently retryable, with a DLQ and natural back-pressure. SQS was the best fit as we didn't need Kinesis for ordered, replayable streams nor EventBridge's multi-consumer capabilities._

### 6. Delivery Semantics & Idempotency

The reviews provider delivers webhooks **at-least-once**; a timeout or non-200 triggers redelivery. This matches with the at-least-once product in SQS. Should the same review update be seen more than once, the two layers of deduplication clean that up.

At intake, duplicate deliveries for a variant collapse into the **Redis Set**. At cron time, variants collapse again into **one entry per product**. This shrinks every redelivered / overlapping webhooks into one data point per reviewed product.


### 7. Persistence & The Data Model
Two things live in **DynamoDB**: the variant→product mapping the cron uses to roll variants up to their parent, and the product-level reviews summary data.

The processing state lives entirely in Redis: the intake set, the rotated processing set (at cron trigger time, the intake set data is emptied into the processing set, ensuring that new review data are captured for the next sync cycle), and a per-store metadata record marking each store's processing state. Maintaining this transient bookkeeping data out of DynamoDB ensures that our system of record holds only durable facts.

> **Why DynamoDB, not a relational store.**
> 
> _The access pattern is known and narrow: look up a product by store and key. DynamoDB gives single-digit-ms read times for the future read path and scales with Lambda concurrency without needing a connection pool to exhaust. Also, billing is per request, much better than having a long-running instance._


#### The Spine: Trace one review update

{{< img src="images/review_update_pipeline_schematic.png" alt="review data flow">}} 

1. The reviews provider POSTs to our internal webhook intake endpoint.
2. Intake gets processed to Redis only if the payload is from an eligible event.
3. Based on a set schedule, the cron rotates the Set and dedups variants up to their parent product, with one representative variant per product.
4. Each product unit is fanned out over SQS; a worker fetches that reviews summary from the reviews provider using the representative variant.
5. The reviews summary data is written to DynamoDB at product + store level; Redis metadata marks the cycle complete once processing for each store is completed.


### 8. Resilience, Blast Radius & Graceful Degradation

To address these, I implemented the following:
1. 3rd-Party auth tokens are cached in Redis to minimize re-authentication calls.
2. Requests to external services are implemented using retry with backoff.
3. Failures are wrapped in a custom error, ensuring a uniform response payload carrying a correlation id.
4. Worker messages that keep failing are moved to a DLQ, where they are inspected and re-driven when needed.

Taken together, these safeguards mean that should the reviews provider be unavailable for whatever reason, the worst outcome is a slightly delayed refresh, not a wrong or missing rating on the page. This predictability is key to keeping the trust intact when customers need those stars to make up their mind about a product.


### 9. Observability Across Async Hops

Given the design of this system, a single logical operation crosses multiple process boundaries: HTTP Lambda, SQS, cron and then a worker lambda. For optimal observability, a **correlation id** is generated at intake time. Leveraging on Node's Async Local Storage (ALS), this gets propagated across the service components. This way one webhook can be traced all the way to its final review summary write into DynamoDB.
Without this, the tracing would have hit dead-end, and observability data would be unreliable.


### 10. Closing Thoughts

Keeping review ratings accurate and current came down to a few deliberate choices: two layers of deduplication that turn a noisy webhook stream into one unit of work per product, SQS fan-out that makes each unit independently retryable, and a correlation id that lets us trace any update end to end. 

Although our customers never see this plumbing, getting it right is how those stars on the product pages stay worth believing.
