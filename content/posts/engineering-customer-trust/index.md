---
title: "Engineering for Customer Trust: Keeping Product Reviews Accurate and Current"
date: 2026-09-15T18:04:32+02:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - Distributed Systems
  - 
  - System Architecure
tags:
  - AWS
  - Queues
  - Lambdas
  - DynamoDB
---


In the highly-subjective e-commerce world, social proof plays a major role in shaping one's buy decision.
As a customer, when I land on a Product Listing Page (PLP) with twenty different products matching my search criteria, I really don't want to open ten tabs to find the best one, I could use some help from the opinions of other customers instead.
Research data has shown that surfacing social proof early in the funnel potentatially signals high transparency, reduces cognitive load and accelerates decision-making which improve core business metrics like Click-Through Rate (CTR).

This article covers the engineering approach taken to make this possible.
I'll walk through how a single user review moves through the pipeline, from the moment it arrives to the moment its aggregate score is recomputed and the system of record is updated.

---

### 1. What we're solving for

In this domain context, review ratings (stars) behave like product attributes. They sit next to price and availability on the product card, they appear on listing pages where dozens of products render at once. They can be sorted, filtered and searched like any other facet. 
However, unlike price and availability, they live in a rate-limited 3rd Party API out of our direct control.

The goal is to own this review data in order to enable these capabilities, while keeping the data as fresh as needed, across every store/country we operate in (19 at the time of this writing).


{{< img src="images/reviews-svc-architecture.png" alt="architecture diagram" caption="architecture diagram">}} 


### 2. The backbone at a glance

As seen in the above diagram, three Lambda runtimes share one codebase. 
- An **HTTP** lambda receives webhooks and admin calls
- A **worker** lambda drains an SQS queue
- A **cron** lambda runs the scheduled aggregation. 

They're composed per-entry-point with dependency injection, maintaining one deploy and one source of truth for the domain logic, rather than three services that drift out of sync. 
Redis caches Yotpo auth tokens across all of them, and DynamoDB is the durable store.

### 3. Bootstrapping the integration: webhook registration
First we expose our webhook intake URL to the 3rd-party reviews management tool, which then triggers it whenever a relevant review event occurs.

### 4. Webhook intake
Once subscriptions exist, events flow to to the intake url. The intake handler does as little as possible: validate the envelope, decide whether the event matters, record it, and acknowledge fast. 

Irrelevant evnets are dropped as they don't change the aggregate data being catered to by this flow. For qualifying events, the variant-sku (a subset of the product data) is extracted and saved to **saved to a Redis set**. The Set acts as the first line of deduplication: ten reviews belonging to the same variant are represented as a single set entry. That's all about the Intake logic, no DB or 3rd party calls.


### 5. The processing cycle: cron, a second dedup, and fan-out
The intake component keeps track of *which* variants got new reviews, However, the real work is driven by the **cron** lambda, which is triggered by an EventBridge task on a set schedule (e.g daily or weekly).
When invoked, this lambda atomically rotates the intake set into a processing set and runs a **second level of deduplication**: collapsing variants up to their *product-level*, the parent that groups a family of variant SKUs.

This second-level dedup is a key component of the design as our review management tool aggregates review data at the product level, using a concept of the 'master' variant.
Also, since reviews are represented at the product-level, our internal system of record in this design  captures review data at the same level. A config with fifty changed variants becomes a single unit of work carrying one variant that speaks for it. This essentially collapses dozens of raw webhooks into one call per affected product.
The resulting per-product payload are then fanned out as an **SQS batch**. 

A **Worker Lambda** drains the queue. It uses the representative variant data to fetch the product's review stats from the 3rd party tool, and then writes it to DynamoDB. Failures here a captured via a DLQ, which get inspected and re-driven.


> **Why fan out over SQS instead of one long cron invocation.** 
A single Lambda looping over every config (there are over 50k of those per store) would press against the 15-minute ceiling. Fanning the deduplicated units onto SQS makes each independently retryable, with a DLQ and natural back-pressure. SQS was the best fitm as we didn't need Kinesis forr ordered, replayable streams nor EventBridge's multi-consumer capabilities.

## 6. Delivery semantics & idempotency

Review Tool X delivers webhooks **at-least-once**; a timeout or non-200 triggers redelivery. This matches with the at-least-once config in SQS. Should the same review update will be seen more than once, the two layers of deduplication cleans that up.

At intake, duplicate deliveries for a variant collapse into the **Redis Set**. At cron time, variants collapse again into **one entry per product**. This shrinks every redelivered / overlapping webhooks into one data poit per reviewed product.


### 7. Persistence & the data model
Two things live in **DynamoDB**: the variant→config mapping the cron uses to roll variants up to their parent, and the product-level reviews summary data.

The processing state lives entirely in Redis: the intake set, the rotated processing set (at cron trigger time, the intake set data is emptied into the processing set, ensuring that new review data are captured for the next sync cycle), and a per-store metadata record marking each store's processing state. Maintaining this transient bookkeeping data out of DynamoDB ensures that our system of record holds only durable facts.

> **Why DynamoDB, not a relational store.** 
The access pattern is known and narrow: look up a config by store and key. DynamoDB gives single-digit-ms read times for the future read path and scales with Lambda concurrency withiout needing a connection pool to exhaust. Also, billing is per request, much better than having a long-running instance.


### The spine — trace one review update

1. Review Tool POSTs to the our internal webhook intake endpoint.
2. Intake gets processed to Redis only if the  paylload is for an eligible event.o
3. Based on aset schedule, the cron rotates the set and dedups variants up to their parent product, with one representative variant per product.
4. Each config unit is fanned out over SQS; a worker fetches that config's bottomline from Yotpo using the representative variant.
5. The reviews summary data is written to DynamoDB at product + store level; Redis metadata marks the cycle complete once processing for each store is completed.

### 8. Resilience, blast radius & graceful degradation
Some things I implemented to cater for these includes:
1. 3rd-Party auth tokens are cached in Redis to minimize re-authentication calls.
2. Requests to external services are implemented using retry with backoff.
3. Failures are wrapped in a custom error, ensuring a uniform response envelope carrying a correlation id.


### 9. Observability across async hops
Given the design of this system, a single logical operation crosses mulitple process boundaries: HTTP Lambda, SQS, cron and then a worker lambda. For optimal observability, I  leveraged the Node runtime's Async Local Storage (ALS) to generate a **correlation id** at intake, which get propagated across every bounddary. This way one webhook can be traced all the way to its finalyreview summary write into DynamoDB.
Without this, the tracing would have hit dead-end, and observability data would not be reliable.

