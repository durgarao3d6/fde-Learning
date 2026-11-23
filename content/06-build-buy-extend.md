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

---

# Detailed notes: architecture and vendor due diligence

## 1. The FDE's real question
When DemoStore asks for an AI assistant, the engineer should not automatically start building a new support platform. The client already owns systems and contracts. The best solution may be to **extend existing ServiceNow** rather than replace it or buy a wholly new stack.

The lesson's governing line:
> Buy what fits, extend what the client already has, and build only the thin layer that nothing else covers.

Every word matters:
- **Buy** repeatable commodity capability that a vendor supplies well.
- **Extend** current workflow investments and approvals.
- **Build** unique integrations, policy logic and glue code not adequately supplied by vendors.

## 2. Inventory before designing
**Ravi Menon**, DemoStore's systems owner (identified elsewhere as Director of E-commerce Engineering), should establish:
- Which ServiceNow modules and licenses are already owned.
- Whether **Now Assist** is enabled.
- Which workflows exist for case management, note taking and escalation.
- Who controls changes to ERP and production integrations.
- Where APIs exist, and where they do not.

Now Assist may already support case summaries and case notes. Building another summarization layer may duplicate paid functionality.

## 3. Vendor landscape discussed
The course references **Intercom Fin**, **Zendesk AI**, **Salesforce Agentforce**, **Sierra** and **Decagon**, plus the existing ServiceNow stack. These names are examples to evaluate, not a recommendation to buy all of them.

A course-cited **2025 a16z survey** of **100 heads of IT** found **more than 90%** were trying off-the-shelf support products. This indicates a common market direction in that survey; it does not prove suitability for DemoStore.

### Questions to ask each vendor
- Can it integrate with the legacy order system?
- Can it read fresh carrier status safely?
- What exactly counts as a *resolved* interaction for billing?
- Does it honor market-specific return rules deterministically?
- How does it handle human escalation to ServiceNow?
- Can the team audit what data was sent, retained and used?
- Which privacy and security assurances are contractual?
- What breaks if the vendor changes price or model behavior?

## 4. Cost arithmetic in full
Illustrative course figures:
- Vendor **$0.99 per counted resolution**.
- Human support **$7 per contact**.
- **1,600 weekly order-status chats**.

Compute vendor baseline:
```text
1,600 × $0.99 = $1,584 per week
$1,584 × 52 = $82,368 per year
```

Compute human baseline:
```text
1,600 × $7 = $11,200 per week
$11,200 × 52 = $582,400 per year
```

Rough verbal approximation:
- Vendor ~$1,600/week.
- Human ~$11,000/week.

**Do not say the difference is net savings.** Reasons:
1. Not all chats will be resolved by the vendor.
2. Some customers will contact support again.
3. Escalations still consume human time.
4. Integration and engineering cost money.
5. Vendor billing metrics may be more generous than business-success metrics.
6. There may be minimum commitments or platform fees.

### More realistic unit-economic frame
```text
Effective cost per genuine resolution
  = total vendor fees
  + implementation / operational costs
  + remaining human support costs
  + costs of repeat contacts
  ------------------------------------------------
          genuinely resolved issues
```
The expression is conceptual. Do not use it without consistent definitions and attribution.

## 5. Defining genuine resolution
DemoStore chooses a **business-based** definition:
> A chat is genuinely resolved only when there is **no human handoff** and **no repeat contact for the same issue within seven days**.

This is stronger than a vendor counter that marks a chat resolved when the customer simply exits.

**Important distinction:** “The user went quiet” ≠ “the user got the answer.” Abandonment may masquerade as successful automation.

A pilot should reconcile vendor invoices with DemoStore's own outcome data.

## 6. Why custom integration remains
### Ledgerline
The legacy **Oracle ERP** lacks an easily usable API or supported vendor connector. This makes **custom middleware** necessary regardless of whether the chat frontend is purchased.

### Returns
Return eligibility depends on deterministic policy details that a generic conversational model should not invent. Implement an approved rules engine.

### Privacy
**Sofia Lindqvist**, DemoStore's Data Protection Officer, needs to review where customer data flows, how long it remains, which vendors process it and the applicable geography.

### Support workflows
Cases, approvals and human handoffs should remain inside ServiceNow rather than creating an unreviewed parallel system.

## 7. A plausible hybrid design
```text
Customer website
      ↓
Supported chat / ServiceNow-facing assistant
      ↓
Custom orchestration and authorization
      ├── Ledgerline middleware (unique integration)
      ├── Carrier status adapter (nightly data)
      ├── Deterministic return-policy engine
      └── ServiceNow (cases and approvals)
```

This is a **conceptual architecture** drawn from the lesson, not an assertion that DemoStore deployed it.

## 8. Common decision errors
- Calling vendor price a direct saving without considering real outcomes.
- Assuming an existing enterprise platform has no relevant AI feature.
- Equating vendor-marked resolution with actual customer success.
- Ignoring privacy data location and retention.
- Buying a system that still cannot access the ERP.
- Building everything custom because coding is fun.

## 9. Strong interview answer
**Question:** “Why not simply buy Intercom Fin or another support agent?”

**Answer:** “I would compare vendor capability against DemoStore's actual workflows, existing ServiceNow/Now Assist licenses, Ledgerline integration constraints, security and privacy needs, and cost per **genuine** resolution rather than vendor-billed resolution. My likely design would extend ServiceNow, purchase commodity functionality only where it fits, and build thin middleware and deterministic policy controls for the gaps.”

### Revision checklist
- [ ] Principle: **buy / extend / build thin missing layer**.
- [ ] Owners: **Ravi** for systems, **Sofia** for privacy.
- [ ] Existing: **ServiceNow, Now Assist**.
- [ ] Vendors: **Fin, Zendesk, Agentforce, Sierra, Decagon**.
- [ ] Course survey: **100 IT heads, >90% trying off-the-shelf**.
- [ ] Weekly and annual cost arithmetic.
- [ ] **Seven-day same-issue repeat** resolution definition.
- [ ] **Legacy Ledgerline** integration gap.

---

# Additional Vendor Selection and Financial Review Detail

## 1. Stakeholder review is not optional
A vendor can look excellent in a demo and still fail when connected to a legacy system. DemoStore requires coordinated decisions:
- **Lena Brandt**: sponsor and business outcome.
- **Marco Rossi**: support process and genuine resolution.
- **Ravi Menon**: engineering systems, existing licenses and integration permission.
- **Sofia Lindqvist**: privacy, retention and data location.

The FDE's job is to connect those concerns into a solution rather than optimize only the chatbot's conversational performance.

## 2. Vendor comparison scorecard (illustrative)
| Criterion | Check | Evidence to request |
|---|---|---|
| Existing-system fit | Can it extend ServiceNow / Now Assist? | Integration demonstration |
| Ledgerline support | Is custom middleware needed? | Connector documentation |
| Return decisions | Can deterministic policies be enforced? | Test cases per market |
| Handoff | Is ServiceNow case creation reliable? | End-to-end trace |
| Privacy | Where is data processed, stored and retained? | DPA/security review |
| Billing | What is a billable resolution? | Contract definition |
| Outcome | No handoff AND no repeat within 7 days? | Client-side measurement |
| Operational control | Logs, ownership, rollback, escalation | Engineering runbooks |

The scorecard is a proposed way to evaluate vendors, not a claim that DemoStore used these exact criteria in a formal procurement table.

## 3. Understand the baseline numbers properly
At 1,600 order-status chats weekly:
- **Human-only illustrative cost:** $11,200/week.
- **Vendor fee if every chat is billed at $0.99:** $1,584/week.
- **Annualized:** $582,400 versus $82,368.

A tempting but flawed headline is “we save roughly half a million dollars every year.” That presupposes **all chats are truly resolved**, nobody contacts support again and integration has no costs. The actual FDE model must account for unresolved and repeated cases and the vendor's charging definition.

### Example sensitivity exercise
Suppose only 60% of chats meet DemoStore's true-resolution definition (purely illustrative). The remainder still need handling, and the vendor may charge for outcomes the business does not count. Cost per *genuine* resolution can rise sharply, even if the per-billed-interaction price is low.

Do not present the 60% as observed course data; it is an example showing why definitions matter.

## 4. Why extend rather than replace ServiceNow?
Existing organizations have approved processes, case ownership, audit requirements and trained agents. Reusing them reduces migration risk. Building a new support backend solely for AI may create duplicate systems and scattered accountability.

A custom component should be **thin and specific**: Ledgerline middleware, policy evaluator, secure orchestration and evaluation harness.

## 5. Separation between customer success and vendor success
A vendor's “resolved” event might mean it did not transfer the chat. DemoStore's definition additionally requires the customer **not to come back within seven days for the same issue**.

That second condition forces the measurement to be tied to real customer outcomes rather than surface-level bot activity.

## 6. Where risk can override economic advantage
Even low-cost automation should not handle data in a region or retention arrangement that the DPO has not approved. Nor should it bypass human approval for refunds.

Model quality, privacy, policy governance and integration access are **constraints**, not just price-adjustment factors.


---

# FDE Interview Questions & Answers — Comprehensive Practice Bank

Use these prompts for oral practice. Answer aloud before revealing the explanation beneath each question.

<details>
<summary><strong>1. What is the core build/buy/extend rule?</strong></summary>

Buy what fits, extend what the client already has and build only the thin missing layer.

</details>

<details>
<summary><strong>2. What support platform does DemoStore already use?</strong></summary>

ServiceNow, with Now Assist for capabilities such as case summaries and notes.

</details>

<details>
<summary><strong>3. Who should inventory licenses and integrations?</strong></summary>

Ravi Menon, the systems owner and Director of E-commerce Engineering.

</details>

<details>
<summary><strong>4. Which vendors are discussed?</strong></summary>

Intercom Fin, Zendesk AI, Salesforce Agentforce, Sierra and Decagon, alongside ServiceNow.

</details>

<details>
<summary><strong>5. What survey detail does the course cite?</strong></summary>

A 2025 a16z survey of 100 IT heads in which more than 90% were trying off-the-shelf customer-support products.

</details>

<details>
<summary><strong>6. What example vendor cost is used?</strong></summary>

$0.99 per vendor-counted resolution, an illustrative historical figure.

</details>

<details>
<summary><strong>7. What example human cost is used?</strong></summary>

$7 per contact.

</details>

<details>
<summary><strong>8. At 1,600 chats a week, what are the weekly costs?</strong></summary>

Vendor 1,600 × $0.99 = $1,584; humans 1,600 × $7 = $11,200.

</details>

<details>
<summary><strong>9. What are the simple annualized totals?</strong></summary>

$82,368 vendor versus $582,400 human, before integration, repeat contacts and residual human work.

</details>

<details>
<summary><strong>10. Why is the arithmetic not net savings?</strong></summary>

It assumes equal true resolution, ignores remaining human handling and excludes integration, platform and operating costs.

</details>

<details>
<summary><strong>11. What is DemoStore's true-resolution definition?</strong></summary>

No human handoff and no repeat contact on the same issue within seven days.

</details>

<details>
<summary><strong>12. How could vendor resolution counts be misleading?</strong></summary>

The provider might count a chat ending or a customer abandoning it as resolved, even when the real issue was not solved.

</details>

<details>
<summary><strong>13. Why does DemoStore still need custom middleware?</strong></summary>

Ledgerline's legacy Oracle ERP lacks a suitable API or supported vendor connector.

</details>

<details>
<summary><strong>14. Who reviews privacy storage, retention and geography?</strong></summary>

Sofia Lindqvist, the Data Protection Officer.

</details>

<details>
<summary><strong>15. What should be implemented as deterministic code?</strong></summary>

Market- and contract-specific returns policy rather than trusting model-generated policy decisions.

</details>

<details>
<summary><strong>16. Why extend ServiceNow rather than replace it?</strong></summary>

Existing cases, approvals, staff processes and licenses already have organizational value and governance.

</details>

<details>
<summary><strong>17. What is the strongest interview answer to 'why not buy everything'?</strong></summary>

A vendor may cover commodity chat while failing legacy Ledgerline integration, specific policy rules, data-processing requirements or DemoStore's genuine-resolution metric.

</details>

## How to practise with this chapter

1. Explain the lesson's actual business problem in 45 seconds.
2. Recreate the architecture or sequence from memory.
3. Recall exact numbers and named decision-makers without checking the notes.
4. Explain one failure case and the deterministic safeguard.
5. Use the Q&A above to prepare a two-minute FDE interview answer.

