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
