# Build vs Buy vs Extend

## Learning objective
Choose an enterprise architecture using existing investments, vendor capability, cost and integration constraints.

## The guiding principle
**Buy what fits, extend what the client already has, and build only the thin layer that nothing else covers.**

## DemoStore starting point
DemoStore already uses **ServiceNow**, and its **Now Assist** add-on can summarize cases and assist with notes. Systems owner **Ravi Menon** inventories licenses and available functionality.

Vendors discussed include Intercom Fin, Zendesk AI, Salesforce Agentforce, Sierra and Decagon.

A 2025 a16z survey cited by the course spoke to 100 IT leaders and reported more than 90% exploring off-the-shelf support AI. This is course-cited background, not a current survey result.

## Unit economics
Illustrative vendor price: **$0.99 per resolution**. Illustrative human cost: **$7 per contact**.

At **1,600 order-status contacts per week**:
- Vendor billed resolutions: `1,600 × $0.99 = $1,584/week`
- Human contacts: `1,600 × $7 = $11,200/week`
- Annualized gross numbers: **$82,368** versus **$582,400**

These are *not* automatically net savings. Include licensing terms, integration costs, handoffs, repeat contacts and the real meaning of “resolved.”

## True resolution
DemoStore defines a genuine resolution as **no human handoff AND no repeat contact for the same issue within seven days**. Vendor billing definitions may differ: a customer abandoning the conversation should not be confused with success.

## Custom gaps
- Legacy **Ledgerline** Oracle ERP lacks a suitable modern API/vendor connector: build middleware.
- Return eligibility needs deterministic policy rules.
- DPO **Sofia Lindqvist** reviews where customer data is stored, retained and processed.
- Existing ServiceNow workflows should be reused where appropriate.

## Review questions
1. Why not build the whole support platform?
2. What is the difference between vendor-billed resolution and business resolution?
3. What custom integrations remain even if an AI vendor is purchased?
4. Who owns privacy and systems approvals?

## Remember
**Extend ServiceNow, build the missing integration and rules, and evaluate genuine outcomes.**
