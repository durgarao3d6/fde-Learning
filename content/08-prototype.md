# Time-Boxed AI Prototypes

## Learning objective
Test one business assumption cheaply without turning a demo into production by accident.

## Prototype as a question
**A prototype is a question, not a product.** It should test a written assumption within a fixed number of days.

DemoStore's assumption: with order data, the assistant can correctly answer order-status inquiries and escalate cases requiring a human.

An **illustrative** acceptance target is at least **36/40 correct actions (90%)** and **zero missed required handoffs**. Both must hold. These are proposed prototype criteria, not universal production thresholds.

## Time box
Agree on **five working days** in advance. At the end, decide **yes, no or not clear yet**. Sponsor **Lena Brandt** can redirect spending based on the result.

Do not automatically add a week for appearance. Extend only for a concrete unresolved question.

## Real versus fake
**Keep real:** the model, prompt, tool descriptions and Marco Rossi's **40 messy real labeled chats**.

**Fake surrounding dependencies:**
- Ledgerline → file of **20 test orders**.
- Carrier nightly feed → fixed statuses, such as “out for delivery, Thursday.”
- Email one-time-code infrastructure → controlled test-customer mechanism.

Preserve the contract: `find_order` has the same name and fields behind real or fake backends. The model should not care which implementation is behind the tool.

**Never fake the input being evaluated** by replacing messy real chats with unnaturally neat samples.

## Safety rules still apply
- Keep API keys out of source code.
- Use customer data only in DPO Sofia Lindqvist-approved places.
- Never expose a money-moving tool.
- Enforce identity and order ownership, even with fake systems.

## After five days
Track shortcuts in a repository **quick-fix list**. If the assumption fails, discard experimental implementation but keep the learnings. If it passes, keep clean tool definitions, prompt, evaluation harness and fake tool fixtures for regression tests. Rewrite shortcuts with tests and review before any pilot.

Do not let a hardcoded market-specific return window remain in production for months.

## Review questions
1. What should a prototype prove?
2. Why should the 40 real chats never be replaced by easy synthetic examples?
3. Which components can be faked?
4. Which safety controls cannot be faked?
5. What is kept after success or failure?

## Remember
**Fake the systems around the question, never the question itself; decide when you will stop.**

---

# Detailed notes: the five-day decision experiment

## 1. Prototype is not product
A prototype should answer a **specific uncertain question** for the client. It is not primarily a beautiful interface, polished architecture or production service.

The transcript's memorable line:
> A prototype is a question, not a product.

A prototype may be successful even when it concludes **“this approach does not work”**, because avoiding a large mistaken build has business value.

## 2. Agree on one falsifiable assumption
DemoStore's proposed test:
> With appropriate order data, can an AI assistant answer “where is my order?” correctly and escalate the right chats to a human?

The lesson illustrates an acceptance bar with **40 labeled real chats**:
- At least **36 of 40** correct actions = **90%**.
- **Zero missed required human handoffs**.

These are **illustrative prototype criteria**; a production pilot may require more scenarios, reliability checks, monitoring and operational review.

**Both criteria must be satisfied.** A model that achieves 38 correct actions but misses a mandatory handoff has not met the proposed safety condition.

## 3. Time-box in advance
The time limit is **five working days**. Agree on it **before** implementation.

At the deadline, choose:
- **Yes** — the assumption held under the stated test.
- **No** — the approach did not meet the conditions.
- **Not clear yet** — evidence is insufficient and a new narrowly scoped experiment is needed.

**Lena Brandt** can use this decision to change direction before committing additional resources.

Do not turn five days into another week merely to improve appearance. Additional time should answer a specific unresolved question.

## 4. Real versus fake: the most important design choice
Keep **real** everything directly bearing on the assumption:
- Actual candidate model.
- Prompt and escalation instructions.
- Tool descriptions and contract shape.
- **Forty real labeled support conversations** supplied by Marco Rossi's agents.

Fake infrastructure not central to the question:
- Ledgerline database → **20 test orders** in a small file.
- Carrier nightly feed → a few fixed statuses, e.g. **“out for delivery, Thursday.”**
- Email-code verification infrastructure → simplified controlled test mechanism.

The code's interface should remain stable:
```python
find_order(order_number=..., customer=...)
delivery_status(tracking_number=...)
```

Both the fake implementation and later production implementation should satisfy the same validated tool contract.

## 5. Why real messy chat inputs matter
The biggest experimental mistake is to replace the genuine 40 conversations with clean, polite, short messages invented to make the model look good.

Real customer text includes:
- Missing numbers and context.
- Typos, emotion and frustration.
- Multi-part requests.
- Implicit human-handoff needs.
- Unclear shipment descriptions.

Synthetic easy examples can hide the exact failure modes you are trying to discover.

## 6. A mock backend example (illustrative)
```python
TEST_ORDERS = {
    "DEMO-100": {"customer": "TEST-CUSTOMER-1", "tracking": "TRACK-001"},
    "DEMO-101": {"customer": "TEST-CUSTOMER-2", "tracking": "TRACK-002"},
}

def find_order(order_number, verified_customer):
    if verified_customer is None:
        return {"error": "verification_required"}
    item = TEST_ORDERS.get(order_number)
    if not item or item["customer"] != verified_customer:
        return {"error": "not_found"}
    return item
```

The prototype can use fake records **without** bypassing ownership checks.

## 7. Safety is not faked
Even for a five-day experiment:
- **Keep the model API key out of source code.**
- Use customer data only where DPO **Sofia Lindqvist** approves.
- **Do not expose a tool that moves money.**
- Preserve verified identity and order ownership requirements.
- Avoid unapproved modifications to Ledgerline.
- Keep demo fixtures separate from real customers.

A disposable prototype may use shortcuts for presentation and integrations, but it should not teach unsafe action patterns.

## 8. The shortcut ledger
Maintain a list in the repository of:
- Fake integrations.
- Hardcoded test configuration.
- Missing observability.
- Temporary data storage.
- Simplified error handling.
- Known tests not yet covered.

This list prevents a pilot team from treating experimental shortcuts as intentional production design.

## 9. What to keep after the decision
**If the assumption fails:** discard experimental architecture that only served the question, but retain the tested evidence and lessons.

**If the assumption holds:** keep clean, reusable assets such as:
- Tool schemas.
- The prompt.
- The evaluation runner.
- Fake tool implementations used by regression tests.
- Clearly documented acceptance criteria.

Rewrite shortcuts with tests and code review before a broader pilot.

A particularly risky shortcut: **hardcoding a return window** and leaving it unchanged when the approved policy evolves.

## 10. The stage-specific quality bar
The meaning of “good enough” changes:
| Stage | Main evidence needed |
|---|---|
| Prototype | Answers the narrow assumption on specified labeled examples |
| Pilot | Real approved integrations, logs, access controls, handoffs, operational review |
| Production | Reliability, monitoring, incident handling, governance and continuous reevaluation |

**Prototype pass ≠ authorization to ship.**

## 11. Mistakes from the lesson
1. Extending the time box because the demo looks rough.
2. Faking the *test input* rather than the surrounding systems.
3. Deploying demo shortcuts into production.

## 12. Interview-ready explanation
**Question:** “How do you time-box an LLM prototype?”

**Answer:** “I write a specific assumption and acceptance test, agree on five days, retain the real model and real labeled customer cases, fake nonessential ERP or carrier infrastructure behind stable interfaces, enforce authorization and privacy throughout, and stop on the deadline to decide yes, no or insufficient evidence. I then keep reusable evaluation and test fixtures but do not promote shortcuts directly into production.”

### Revision checklist
- [ ] **Five working days**.
- [ ] **36/40** correct and **zero missed handoffs** as an illustrative bar.
- [ ] **20 test orders**.
- [ ] **Real 40 chats** cannot be replaced with easy fakes.
- [ ] Reuse `find_order` and `delivery_status` contracts.
- [ ] Decision: **yes / no / not clear yet**.
- [ ] Track shortcuts and keep regression fixtures.
- [ ] Never fake **privacy, authorization or money controls**.

---

# Complete Experiment Plan and Decision Record

## One-page example of the prototype agreement
**Question:** With sample order data available, can the model correctly decide what to do for real order-status and escalation cases?

**Data:** Marco Rossi's **40 real labeled customer chats**.

**Assumed pass condition (illustrative):**
- **At least 36/40** correct required actions.
- **Zero missed mandatory human handoffs**.

**Duration:** **Five working days**, agreed before implementation.

**In scope for testing:** model, prompt, tool descriptions, evaluation data and scoring harness.

**Simulated supporting systems:** **20-order sample Ledgerline file**, a small carrier-status fixture, and controlled demonstration of identity-verification infrastructure.

**Always enforced:** approved data handling, secrets outside code, verified ownership checks, no money-moving tool.

**Decision at deadline:** yes, no or not clear yet.

## A worked interpretation of the result
| Outcome | Pass? | Reason |
|---|---|---|
| 36 correct, zero missed handoffs | Yes under illustrative prototype bar | Both criteria met |
| 39 correct, one missed handoff | **No** | Safety condition failed |
| 34 correct, zero missed handoffs | **No** | Accuracy condition failed |
| No trustworthy grading for five cases | Not clear yet | Evidence incomplete |

These rows are examples of how to apply the stated criteria, not results reported from the lesson's actual prototype run.

## Why one high-risk miss dominates the score
Suppose the assistant gives a confident answer to a conversation requiring a human. A high aggregate score would disguise a failure in the exact escalation mechanism the prototype needs to establish.

This links the experiment's design back to the model-comparison lesson, which found unacceptable missed handoffs despite comparing three candidate models.

## The keep-or-throw-away decision
At five days:
- **Keep** reusable verified knowledge and test fixtures.
- **Keep** clean tool descriptions, prompt versions and evaluation scripts if appropriate.
- **Discard** disposable mock presentation layers that have no reason to exist beyond the test.
- **Rewrite** any temporary integration shortcut before a pilot.
- **Document** what remains unknown and which assumption should be tested next.

### Danger: the accidental production prototype
A five-day system may contain temporary code that appears to work. The team can become attached to it and deploy it without production controls. The lesson explicitly warns against this failure mode. **Prototype evidence is not production certification.**

## How to explain a time-box extension
**Bad rationale:** “The demo doesn't look polished.”

**Better rationale:** “We discovered that five of the real messages cannot be graded consistently because the label definition is unclear; we need an owner-led labeling pass to answer the original assumption.”

Even then, define the narrow new question and deadline rather than silently extending scope.

## A shortcut log template
```markdown
# Prototype shortcuts and risks

- [ ] Ledgerline is a file fixture of 20 test orders.
- [ ] Carrier data is a fixed status fixture.
- [ ] Verification uses a controlled test mechanism.
- [ ] No production persistence or monitoring.
- [ ] API keys provided outside repository source.
- [ ] Customer data location approved by Sofia.
- [ ] Refund execution is not exposed.
- [ ] All shortcuts must be reviewed before a pilot.
```

This list is illustrative, but the **shortcut ledger and review before pilot** are explicit lesson ideas.


---

# FDE Interview Questions & Answers — Comprehensive Practice Bank

Use these prompts for oral practice. Answer aloud before revealing the explanation beneath each question.

<details>
<summary><strong>1. How does the course define a prototype?</strong></summary>

A question, not a product. It tests one written assumption under a fixed time box.

</details>

<details>
<summary><strong>2. What is DemoStore's proposed assumption?</strong></summary>

With appropriate order data, can the assistant answer order-status chats correctly and escalate the right cases?

</details>

<details>
<summary><strong>3. What illustrative accuracy threshold is proposed?</strong></summary>

At least 36 correct actions out of 40 labeled chats, or 90%.

</details>

<details>
<summary><strong>4. What is the second mandatory criterion?</strong></summary>

Zero missed required human handoffs. Both accuracy and safety criteria must pass.

</details>

<details>
<summary><strong>5. What is the agreed time box?</strong></summary>

Five working days, agreed before implementation.

</details>

<details>
<summary><strong>6. What are the end-of-time-box decisions?</strong></summary>

Yes, no or not clear yet.

</details>

<details>
<summary><strong>7. Who can redirect spending after the experiment?</strong></summary>

Sponsor Lena Brandt, using the experiment's evidence.

</details>

<details>
<summary><strong>8. Which parts must be real?</strong></summary>

The actual model, prompt, tool descriptions and Marco's 40 messy real labeled conversations.

</details>

<details>
<summary><strong>9. What may be faked around the test?</strong></summary>

Ledgerline with 20 test orders, a few fixed carrier statuses and controlled email-code test infrastructure.

</details>

<details>
<summary><strong>10. What is the sample carrier status example?</strong></summary>

'Out for delivery, Thursday' is an illustrative fixture.

</details>

<details>
<summary><strong>11. Why preserve find_order and delivery_status contracts?</strong></summary>

So fake fixtures can later be swapped for real systems without teaching the model a different interface.

</details>

<details>
<summary><strong>12. Why not replace real chats with clean synthetic ones?</strong></summary>

That would make the actual tested input easier and potentially hide real failure modes.

</details>

<details>
<summary><strong>13. What safeguards still apply in a five-day prototype?</strong></summary>

No API keys in source, DPO-approved data handling, ownership checks and no money-moving tools.

</details>

<details>
<summary><strong>14. What is a quick-fix list?</strong></summary>

A repository record of temporary shortcuts that must be examined and rewritten before pilot or production.

</details>

<details>
<summary><strong>15. What is kept if the prototype works?</strong></summary>

Clean tool definitions, prompt, evaluation program and fake tool versions for automated regression tests.

</details>

<details>
<summary><strong>16. What is kept if it fails?</strong></summary>

The evidence and learning; discard experimental implementation that has no remaining value.

</details>

<details>
<summary><strong>17. Why is a failed prototype sometimes a success?</strong></summary>

It rejects an expensive wrong assumption quickly and informs the sponsor before a large commitment.

</details>

<details>
<summary><strong>18. What are the three common mistakes?</strong></summary>

Extending the time box for polish, faking the actual test inputs, and treating prototype shortcuts as production design.

</details>

<details>
<summary><strong>19. What is the central quote?</strong></summary>

Fake the systems around the question, never the question itself, and decide before you start when to stop.

</details>

## How to practise with this chapter

1. Explain the lesson's actual business problem in 45 seconds.
2. Recreate the architecture or sequence from memory.
3. Recall exact numbers and named decision-makers without checking the notes.
4. Explain one failure case and the deterministic safeguard.
5. Use the Q&A above to prepare a two-minute FDE interview answer.

