---
title: "Sending Emails with AWS SES Cross-Account Setup (The Hidden Sandbox Trap)"
date: 2026-08-10T01:54:28+02:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - AWS Cloud, SES
tags:
  -
---

Setting up Amazon Simple Email Service (SES) across multiple AWS accounts can be a maze of IAM policies and resource authorizations. I recently went through this process and ran into a scenario that completely stumped me. I assumed that my IAM and cross-account setup were all done correctly (policies, identites verification and authorizations, etc), but AWS *still* blocked my emails with an obscure error. 

Here is a look at what went wrong, the hidden AWS quirk I discovered, and how I finally got it resolved.

---

## My Architecture: A Standard Cross-Account Setup

My goal was to separate our email domain management from our application logic:

* An AWS Organization that Contains two accounts belonging to the same OU.
* **Account A (The Owner):** This account owned the verified domain identity (I'll call it `mydomain.com`). I set up an SES Sending Authorization Policy here, granting Account B permission to send emails on its behalf.
* **Account B (The Sender):** This is where my application ran and housed the IAM user executing the API calls using the AWS Go SDK.
* **The Environment:** Both of my accounts were currently in the **SES Sandbox**. 

Everything looked correct on paper. Account A had the right Resource Policy. Account B had the right IAM permissions. My Go application was explicitly targeting the correct region (`us-east-1`) and passing the correct `SourceArn` to assume the domain's identity. 

But when I tried to trigger an OTP email to a recipient I had just verified in Account A, AWS threw this frustrating error:

> `MessageRejected: Email address is not verified.`

---

## The Problem: A Misleading Error Message

The error suggested that an identity wasn't verified. My first instinct was to check my sender domain, but `mydomain.com` was fully verified in Account A. Then, I checked the recipient email, which was *also* verified in Account A. 

But something was definitely wrong.

After a lot of digging, I realized the issue wasn't my IAM permissions, my cross-region routing, or a typo in the domain names. The root cause was a subtle rule in how AWS SES handles **Sandbox restrictions during delegated sending**.

---

## The Root Cause: Sandbox Rules Apply to the Caller

When using cross-account sending, the AWS account making the API call (Account B in my case) is known as the "delegate sender." 

Here is the hidden trap I fell into: **Sandbox restrictions evaluate the account *executing the API call*, not the account that *owns the identity*.**

Because Account B was still in the SES Sandbox, it was strictly bound by its own Sandbox rules. Those rules dictate that it can only send emails to destination addresses verified directly within *its* account. Verifying the recipient email in Account A only satisfied Account A's sandbox limits. Account B had no idea who that recipient was, so it blocked the request entirely.

---

## How I Solved It

To fix this, I realized I had to satisfy the Sandbox restrictions of the delegate sender (Account B). I learned there are two paths depending on the phase of development:

### For Local Testing (My immediate fix)
Since I just wanted to test my cross-account authorization and verify my code worked, I needed to verify the recipient's email address in Account B.

1. I logged into Account B and navigated to the SES console (making sure I was in the correct region).
2. I went to **Verified Identities**.
3. I added and verified my destination email address (the test recipient inbox).
4. I invoked my API again, and the email sent perfectly!

### Option 2: For Production (The next step)
For actual production use, Account B needs to be taken out of the SES Sandbox. Once AWS grants Account B Production Access, it can send to any unverified recipient email address on behalf of Account A without me having to manually whitelist them.

---

## Did I need to verify the domain in Account B too?

**No.** Thankfully, I learned I didn't need to verify the sender domain in Account B. 

Cross-account authorization (via the `SourceArn`) handled the sender side perfectly. The Sandbox restriction *only* cares that the destination is verified when the calling account is sandboxed. 

---

## AWS Doc Reference
Although I found the documentation of this behavior a bit difficult to pinpoint, [this AWS doc](https://docs.aws.amazon.com/ses/latest/dg/sending-authorization-delegate-sender-tasks-email.html) provided some pointers

---

## Final Thoughts

This experience taught me something new about cross-account AWS infrastructure: always remember to check the account-level quotas and sandbox statuses of the *calling* account. IAM policies dictate what an account *can* do, but Sandbox rules dictate what an account *is allowed* to do. 

By verifying the recipient in the calling account, I was able to quickly bypass this hurdle and get back to building.
