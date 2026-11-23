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
