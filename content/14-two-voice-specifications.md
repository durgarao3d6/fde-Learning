# Complete Study Notes: Writing a Two-Voice Specification for DemoStore's Coding Agent

**FDE Course · Requirements Engineering · Agentic SDLC · Business Stories · Technical Contracts · Traceability · Acceptance Criteria · Security Boundaries · Git · Pull Requests · Claude Code**

> **Practical rule:** **If a specification line has no reason or no test, fix the spec before the coding agent builds from it.**

This lesson continues from [Chapter 13 — TypeScript Fundamentals for Browser Chat Widgets](13-typescript-browser-fundamentals.md). The previous lessons built and reviewed the chat service and widget. Now the engineering focus moves to **what a coding agent should be handed before it starts planning or changing code**.

The lesson's idea is deceptively simple: a coding agent can only build from the information it reads. A verbal agreement from a meeting is not enough. The team needs a **reviewed specification in the repository**, structured so that the people who understand the customer's problem and the people who own the systems can both validate it.

**Source-fidelity note:** Facts, roles, numbers, scope decisions, and security rules in these notes come from the supplied lesson transcript. Named example requirement IDs, sample code, tables, and additional suggested tests are clearly **illustrative**; the transcript does not give literal spec file names, exact acceptance-ID strings, or complete runnable code.

---

# Part 1: What This Lesson Is About

## 1. The Gap Between Requirements and Implementation

A requirements document may say **what DemoStore needs**, but it does not necessarily say how to implement it.

For example:

> “A customer wants to ask where her order is.”

This describes a customer goal but leaves engineering questions open:
- Which order system is used?
- How is the customer verified?
- Which tools may the AI request?
- Where does the customer ID come from?
- How many tool rounds are allowed?
- What happens when the agent fails?
- Which tests establish completion?

If the engineer hands only a vague requirement to a coding agent, the agent has to **guess** answers. Even a capable coding model can confidently invent an unapproved architecture.

The course therefore introduces a **spec**, short for **specification**.

## 2. What Is a Specification?

A **specification** is the written, reviewed source document the coding agent uses to plan and build the next change.

It bridges:
1. **What the business needs.**
2. **How the technical system will satisfy it.**
3. **How the team proves the result works.**
4. **What is intentionally excluded or forbidden.**

A useful one-sentence definition for an interview:

> A spec is an approved, traceable agreement linking business outcomes to technical design and verifiable acceptance checks, including explicit safety and scope boundaries.

The important aspect is not document length or elaborate formatting. It is whether the next agent session can **reconstruct the actual decision from the file alone**, without relying on someone remembering an earlier meeting.

## 3. The Three Mechanisms

The lesson teaches **three things**:

1. **Two voices:** business and IT.
2. **Links:** requirements connect to implementation rules and tests.
3. **Boundaries:** out-of-scope items and forbidden actions are written explicitly.

```text
Business voice
   Who needs what, and why?
           │
           ▼
Traceability IDs
   Why does each technical part exist?
   Which test proves each requirement?
           │
           ▼
IT voice
   Tools, data contracts, identity,
   limits, adapters and tests
           │
           ▼
Boundaries
   Not in this build / never allowed
           │
           ▼
Coding agent's reviewed plan
           │
           ▼
Small verified implementation phases
```

---

# Part 2: Voice One — Business Requirements

## 4. The Business Voice Comes First

The **business voice** is the **first section** of the spec.

It says:
- **Who** needs a capability.
- **What** the person needs.
- **Why** that capability matters.
- **What observable conditions** show the requirement is satisfied.

It deliberately uses client language rather than code-level implementation details.

For this first DemoStore build, the central story is:

> A customer asks where her order is.

The business requirement must remain understandable to **Marco Rossi, Head of Customer Support**, who is responsible for reviewing and confirming this section.

## 5. The User-Story Formula

The course states the standard form:

> **As a [person or role], I want [something], so that [benefit].**

Example (reconstructed from the transcript's intent):

> **As a DemoStore customer, I want to know where my order is, so that I can plan for its delivery or know when to ask for help.**

The benefit is not optional decoration. It explains why the feature should exist.

### Decompose the story

| Part | Example | Purpose |
|---|---|---|
| As a | DemoStore customer | Identifies whose need matters |
| I want | Ask about order status | Describes the capability |
| So that | Know delivery progress and plan next steps | States the business outcome |

### Why not write only “Build an AI agent”?

Because an agent is an **implementation choice**, not the customer's problem. The business story should not presuppose tools or model architectures that the client hasn't approved.

## 6. Acceptance Criteria — How Do We Know the Story Is Done?

A story alone is not an acceptance test. The spec also needs **acceptance criteria**: observable conditions that determine whether a requirement has been satisfied.

The transcript's central criterion is:

> **A customer who is not verified sees no order data.**

“Verified” has a specific meaning here:

**The customer entered the one-time code that DemoStore sent to her email, and the application checked it.**

This is stronger than:
- The customer knows an order number.
- The customer supplied an email address.
- The customer typed “I am verified.”
- The model thinks she sounds legitimate.

The criterion is stated at the business level. Marco can validate it without reading an implementation of sessions or functions.

### Illustrative acceptance scenarios

```text
Scenario A — Unverified customer
Given a customer has not completed email-code verification
When she asks for an existing order
Then no private order information is shown

Scenario B — Verified customer
Given a customer has completed valid email-code verification
When she asks about an order she owns
Then the system may show the approved order information

Scenario C — Other customer's order
Given a verified customer asks for someone else's order
When the order tool looks it up
Then no protected order information is shown
```

Only the **first** criterion is stated explicitly in this lesson; the additional cases are educational extensions consistent with earlier DemoStore guardrails.

## 7. The Prototype's Measurable Goal Is Also in the Business Voice

The business section also includes the target inherited from DemoStore's prototype plan:

> **At least 36 of the 40 labeled support conversations must receive the correct action.**

Arithmetic:

```text
36 / 40 = 90%
```

This is a **business-facing outcome**. It says how well the assistant must perform on the agreed evaluation set.

A previous prototype lesson also discussed **zero missed mandatory handoffs** as an illustrative safety criterion. This particular spec transcript **explicitly repeats the 36/40 target**; it does not separately repeat the zero-missed-handoff threshold here. Keep that distinction clear rather than silently adding it to this lesson's quoted specification.

### Why include an evaluation threshold?

Without a measurable target, an agent might make the code compile and declare the work complete even though the AI chooses incorrect actions.

**Build success** is not the same as **business success**.

## 8. No File Names in the Business Section

The narrator explicitly says:

> **There are no file names here.**

This is intentional. The support lead is reviewing a customer's goal and acceptance conditions, not choosing whether the function should live in `agent.py`, `tools.py`, or another module.

### Business voice example

```markdown
## Business requirements

BR-01: As a DemoStore customer, I want to ask
where my order is, so that I know its delivery status.

BR-02: A customer who has not completed
email one-time-code verification must see no order data.

BR-03: At least 36 of 40 labeled conversations
must receive the correct action.
```

**The IDs and wording are illustrative.** The transcript says business lines have **short IDs**, not which exact strings were used.

## 9. Who Signs Off?

**Marco Rossi, Head of Customer Support**, reviews and confirms the business voice.

He can answer:
- Is the story phrased as a real customer need?
- Is the behavior acceptable to support agents?
- Does the verification criterion match DemoStore policy?
- Is the action-correctness target appropriate to the agreed test?

The technical implementation may be adjusted later, but the business decision should not be replaced silently by a coding agent.

---

# Part 3: Voice Two — IT and Engineering Requirements

## 10. The IT Voice Is the Second Section

The **IT voice** says **how** the agreed business outcomes will be implemented.

It names the system components, their data shapes, the trust boundaries, operational limits, and tests.

This section belongs to people who understand DemoStore's systems, especially **Ravi Menon's engineering team**, who own the underlying systems and review the technical specification.

### The division of responsibility

| Business voice | IT voice |
|---|---|
| Who needs what and why | How the system will achieve it |
| User story and acceptance criteria | Functions, tools, data flow and limits |
| Customer-visible outcome | Implementation and authorization behavior |
| Evaluated by Marco | Reviewed by Ravi's engineering team |
| No file names required | Technical contracts may name tools and code |

Both are necessary. The goal is not to make Marco review a Python schema or to leave Ravi guessing what business behavior the client actually approved.

## 11. The Two Approved Order Tools

The IT section explicitly names:
1. **`find_order`**
2. **`delivery_status`**

These tools form part of the bounded agent's available capabilities.

The spec also defines **the input fields** each tool expects.

The transcript states exactly one of those field constraints:

> **`find_order` takes only the order number.**

Why is this constraint important? Because customer identity must not come from the model.

## 12. The Trusted Customer-ID Boundary

**The customer ID comes from the verified application session, never from the model.**

This is a deliberate architectural decision from earlier agent-security lessons.

### Unsafe design — illustrative

```ts
// Conceptual insecure model-requested tool input:
find_order({
  order_number: "ORDER-123",
  customer_id: "someone-else"
})
```

The model controls both the order number and the customer identity. A customer might manipulate the model into supplying **another person's ID**.

### Safer design — illustrative

```python
def find_order(order_number: str, verified_session):
    customer_id = verified_session.customer_id

    if customer_id is None:
        return {"error": "verification_required"}

    return ledgerline_adapter.find_order_for_customer(
        order_number=order_number,
        customer_id=customer_id,
    )
```

The **tool's model-visible input** only includes the order number. The application retrieves trusted identity from its own verified session.

### What this separation achieves

- Model input cannot grant an identity.
- Prompt manipulation cannot directly change the trusted session customer.
- Each tool can authorize access using the real identity.
- Verification remains controlled by deterministic application code.

This connects directly to the business rule: **an unverified customer sees no order data**.

## 13. Why the Customer Could Manipulate the Model

The course gives the reason for excluding customer ID from model arguments:

**A customer could push the model to write someone else's ID.**

This is a trust-boundary problem. The customer controls their message, and the model's tool arguments are influenced by that message.

If tool permissions are based on model-generated IDs, the system could reveal another customer's purchases.

A concise FDE principle:

> **Model-generated arguments may specify which resource to request, but trusted application state decides whether the caller may access it.**

## 14. Centralize Every Model Call in One Adapter

The IT voice also states:

> **Every model call goes through the adapter, with the model name in one constant.**

Earlier lessons called this adapter `llm.py`.

### Why centralize?

An engineering team can change the model name, logging or request handling in one place rather than searching through many files.

### Conceptual architecture

```text
Chat service
      │
Agent loop
      │
Intent extraction
      │
      ▼
Central model adapter (llm.py)
      │
      └── Model name declared in one constant
              ↓
            Claude
```

The transcript directly requires the adapter and single constant. Benefits such as reduced configuration drift are engineering explanations.

### Illustrative code

```python
# llm.py — example, not verbatim lesson source
MODEL_NAME = "configured-model-id"

def ask_model(messages, tools=None):
    return provider_client.messages.create(
        model=MODEL_NAME,
        messages=messages,
        tools=tools or [],
    )
```

The provider call above omits required operational details and is **not a drop-in SDK implementation**; it illustrates the “one model constant / one adapter” requirement.

## 15. The Ten-Round Agent Limit

The IT section specifies:

> **After ten rounds, the agent loop stops and opens a case in ServiceNow.**

This places a firm boundary on model autonomy.

```text
Customer request
      ↓
Agent chooses tool or answer
      ↓
Tool round counter
      ↓
Is work finished?
  ├── Yes → return result
  └── No → Has round limit been reached?
              ├── No → next approved step
              └── Yes → stop and create ServiceNow case
```

A limit enforced in code prevents runaway loops and uncontrolled model/tool traffic.

### Scope nuance worth remembering

In earlier lessons, the minimal internal chat widget did **not yet implement a real person handoff**. This **new spec** establishes a desired **ServiceNow case handoff after ten rounds**.

**A requirement in the spec is not proof that the application already implements it.** The next coding-agent build must be reviewed and tested against that requirement.

## 16. Who Reviews the Technical Voice?

**Ravi Menon's team** reviews the IT portion because they own DemoStore's systems.

They can confirm:
- Which Ledgerline operations are permitted.
- Whether the session identity flow is trusted.
- Whether the tool schemas match the actual adapters.
- Whether the ServiceNow case workflow is available.
- Whether the model-call adapter follows project standards.
- Whether the intended tests demonstrate correct control boundaries.

An FDE must be comfortable moving between Marco's customer-outcome vocabulary and Ravi's system-contract vocabulary.

---

# Part 4: Why Both Voices Are Needed

## 17. What Happens Without the IT Voice?

If the agent only receives:

> “Help customers see where their orders are.”

It can invent an implementation:
- Pass a customer ID through an LLM tool.
- Access Ledgerline directly without middleware.
- Omit verification.
- Keep calling tools indefinitely.
- Invent a human handoff path.

The business outcome alone leaves too many technical decisions ambiguous.

The IT section prevents the agent from **guessing architecture and safety boundaries**.

## 18. What Happens Without the Business Voice?

If the agent only receives a technical requirement such as:

> “Implement `find_order` and `delivery_status` with a ten-round cap.”

It may produce working code but fail to understand:
- Why verification must precede order data.
- Which customer problem matters.
- Why accuracy is measured on real support conversations.
- Why human fallback exists.
- Which extra technical components are unnecessary.

The business section explains **the reason behind the work**.

### Summary comparison

| Missing component | Likely consequence |
|---|---|
| No IT voice | Agent guesses implementation |
| No business voice | Agent lacks business rationale |
| No traceability | Team cannot explain or test choices systematically |
| No boundaries | Agent may overbuild or expose forbidden capabilities |

---

# Part 5: Traceability — Links Between Lines

## 19. Short IDs on Business Requirements

The lesson introduces **short IDs** for business lines. Each line gets a stable reference identifier.

Examples used here:

```text
BR-01: Customer asks where her order is.
BR-02: Unverified customer sees no order data.
BR-03: At least 36/40 labeled cases get the correct action.
```

**These ID strings are illustrative.** The lesson does not give the exact naming convention.

## 20. IT Lines Name the Business Requirement They Serve

An IT rule should refer to the business line that justifies it.

Example:

```text
IT-02 → BR-02
Both order tools remain locked until a valid email
one-time code has been checked in the session.
```

The link answers **“Why does this technical guardrail exist?”**

Without that connection, a reviewer might see a costly or complicated feature without knowing which customer problem requires it.

## 21. Tests Name the Same Business Requirement

The acceptance test also references the business line.

Example:

```text
TEST-02 → BR-02
Given an unverified chat session,
when a customer asks about an existing order,
then the service returns no protected order data.
```

This connection answers **“How do we know the requirement is satisfied?”**

## 22. Follow the Verification Requirement All the Way Through

The transcript uses a complete three-step example.

### Business line

> **An unverified customer sees no order data.**

### IT line

> **Order tools remain locked until the email one-time code is checked.**

### Test line

> **A chat without a checked code asks about a real order and gets nothing back.**

Visualize:

```text
BR-02: No order data before verification
            │
            ▼
IT-02: Block tools until one-time code verified
            │
            ▼
TEST-02: Ask about real order without valid code
            │
            ▼
PASS only if no protected order data is returned
```

This is more meaningful than simply checking that the UI shows a “Please verify” message. The test must establish that **the underlying tool does not leak protected order data**.

## 23. The Two Review Questions Traceability Enables

The lesson gives two short checks that can be applied to every code-review cycle:

1. **Does each business line have a test?**
2. **Does each IT line have a reason?**

These are different directions through the same traceability graph.

### Direction A — Coverage

```text
Business requirement → test
```

A business requirement without a test may be implemented inconsistently or remain unverified.

### Direction B — Justification

```text
IT implementation detail → business requirement
```

A technical item without a reason may be **extra work** or evidence that the business spec is missing a valid need.

**Important nuance:** Some infrastructure or security work can be required by law, policy, operations or architecture even when it is not tied to an end-user story. In this framework, that need should still be made **explicit as a requirement** rather than remaining an unexplained implementation detail.

## 24. A Traceability Matrix

The following matrix is an **illustrative artifact**, based on the transcript's requirements.

| Business ID | Business goal / criterion | IT rule | Test |
|---|---|---|---|
| BR-01 | Customer asks about order status | Approved `find_order`, `delivery_status` tools | Authorized order-status scenario |
| BR-02 | No order data unless verified | Customer identity from checked session; lock tools | Unverified existing-order attempt shows no data |
| BR-03 | At least 36/40 correct actions | Shared adapter and evaluation pipeline | Replay 40 labeled cases, count correct actions |
| BR-04 | Safe fallback when agent cannot finish | Max 10 rounds → ServiceNow case | Exhaust 10 rounds and assert case workflow |

**BR-04 is added here for explanation:** The transcript states the ten-round IT rule but does not say the business section has this exact line. A reviewer should explicitly document its business rationale.

### Why stable IDs help in pull requests

When code changes affect verification, the reviewer can inspect every requirement and test linked to the same ID instead of searching the entire repository for loosely related prose.

---

# Part 6: Guard Against Unrequested Features

## 25. What if the Agent Adds an Admin Page?

The lesson presents a concrete example:

> **The agent's plan contains an admin page, but no user story asks for one.**

The team should **not build the page simply because the agent suggested it**.

Instead:
1. Stop and ask which business problem the page solves.
2. Ask **Marco Rossi** whether support actually needs it.
3. If the feature is needed, add a business requirement **first**.
4. Link the technical work and its test to the new requirement.
5. Only then approve the plan change.

### Why the admin page is a good example

It may appear “helpful,” but:
- It increases scope.
- It creates new UI and permissions to maintain.
- It can add an administrative security surface.
- It may consume time needed for the approved customer-support goals.

An agent's helpfulness is not authorization for feature creep.

## 26. Why an Unlinked IT Line Is a Signal

The lesson says:

> An IT line with no reason is extra work—or a sign that a business line is missing.

The goal is **not** to delete every technical item without a visible customer story instantly. It is to **stop and clarify** its reason before building.

### Interview answer

**Q: What do you do if a coding agent proposes an admin dashboard nobody requested?**

> “I would ask what requirement justifies it. I would not approve it as ‘helpful extra work.’ If Marco confirms a real need, I update the business section of the spec, link the dashboard to that new requirement, add an acceptance test, and only then let the implementation plan include it.”

---

# Part 7: Out-of-Scope Boundaries

## 27. Why Must a Spec Say What Not to Build?

The transcript observes that **helpful coding agents fill gaps with extra features**.

If the specification lists only desired behavior, an agent may assume that adjacent capabilities are also part of the task.

So the spec includes an **out-of-scope list**.

“Out of scope” means **intentionally excluded from this particular build**, not necessarily forbidden forever.

## 28. The Four Explicit Out-of-Scope Items

The lesson gives these examples:

1. **Real Ledgerline connection**.
2. **Returns**.
3. **Contract prices**.
4. **Policy answers**.

### What each means

**Real Ledgerline integration:** The current build should not attempt to connect to the production ERP. Earlier versions used controlled mock order data.

**Returns:** Return eligibility and return processing are not part of this scoped implementation.

**Contract prices:** The assistant must not add contract-price lookup or answers in this build.

**Policy answers:** It should not invent or implement broader customer policy explanation as a new feature.

A later approved build may add some of these, but this agent session cannot silently expand the scope.

### A conceptual boundary

```text
BUILD NOW:
  Order-status story and approved safeguards
  Specified tools, session verification and tests

NOT IN THIS BUILD:
  Real Ledgerline integration
  Returns
  Contract pricing
  Policy answers
```

**The transcript does not specify a complete delivery schedule or exact backend scope beyond these points.** Do not infer that all other adjacent features are approved.

---

# Part 8: Never-Do Rules — Hard Constraints

## 29. What Is the “Never List”?

The spec includes a **separate never list**: activities the assistant must **not perform**.

This differs from out-of-scope work.

- **Out of scope:** Not planned in this iteration.
- **Never list:** Prohibited behavior under the stated safety constraints; it must not occur even if the model thinks it would help.

The course provides four concrete rules.

## 30. Never Rule One — No Tool Moves Money

> **No tool moves money.**

The assistant may not execute refunds, charges or other money-moving actions.

This is consistent with earlier lessons, where the model may only **draft a refund proposal for human review**.

Why hard enforcement matters: A prompt telling the model not to refund anyone does not prevent it from requesting an exposed refund tool. Safer design **does not expose such a capability** to the model.

## 31. Never Rule Two — Restricted Ledgerline Writes

> **Nothing writes to Ledgerline outside the tables approved by the ERP owner.**

This is a restriction on permitted database changes. A coding agent must not invent a new write path or table because it seems useful.

**Important distinction:** This does **not** itself authorize writes to every approved table. A specific operation must still have an approved business need, proper privileges, validation and review.

This is a strong enterprise-integration design pattern: **data-system owners control the allowed write surface**.

## 32. Never Rule Three — No Order Data Before Verification

> **No order data appears before the customer is verified.**

The condition must be enforced in tool/application code. The model cannot waive it through an apology, explanation or confidence estimate.

“Verified” specifically means the application checked the **email one-time code**, as defined in the business voice.

## 33. Never Rule Four — Dates Come From Tools

> **Dates come only from a tool, never from the model.**

This applies to factual customer-support dates such as delivery dates. If the carrier data does not provide a confirmed ETA, the assistant must not invent one.

It connects directly to the earlier **Leeds depot** example: the carrier data said the shipment was delayed, so the model could state the observed status but could not promise an unsupported delivery date.

### Why this belongs in the spec

A developer who begins only with “answer order-status questions” might accept a fluent but invented shipping date. The never rule tells both the coding agent and the reviewer that such behavior is unacceptable.

## 34. Hard Constraints Must Be Enforced, Not Merely Written

Documenting a never rule is necessary for alignment, but **the file itself is not an enforcement mechanism**.

Example enforcement mapping:

| Never rule | Implementation-level safeguard |
|---|---|
| No tool moves money | Do not provide a money-moving tool to the model |
| Ledgerline write restrictions | Database grants and explicit write allowlists |
| No order data before verification | Session verification + ownership checks in each tool |
| Dates only from tools | Ground answers in returned data; test missing-date cases |

The right tests should intentionally attempt violations.

### Illustrative negative tests

```python
def test_unverified_customer_gets_no_order_data():
    # Try a known order without verified session state.
    ...

def test_agent_has_no_money_moving_tool():
    # Assert the tool allowlist excludes refund execution.
    ...

def test_missing_delivery_date_is_not_invented():
    # Carrier record has no confirmed ETA.
    ...

def test_ledgerline_writes_are_restricted():
    # Assert only owner-approved write operations are reachable.
    ...
```

These are examples of acceptance conditions, **not tests shown running in the supplied transcript**.

---

# Part 9: Keep the Spec Beside the Plan

## 35. Where Does the Spec Live?

The transcript states:

> **The spec is a file in the repository, next to the plan.**

The exact file path is **not provided** in this lesson. A project might choose `docs/spec-chat.md`, but that is only an example.

The important point is that **the next fresh coding-agent session can read the same reviewed document**.

## 36. Why Repository Files Matter More Than Meeting Memory

A human might remember that Marco approved a different acceptance threshold or Ravi changed an integration limit. A coding agent starting a fresh session will not necessarily know that.

It will read the repository files that are available.

If the spec is stale, it can reconstruct and reintroduce an **outdated design**.

This creates a subtle regression mechanism:
1. The team changes an implementation decision verbally.
2. Code is updated manually.
3. The spec remains unchanged.
4. A fresh coding agent later reads the old spec.
5. The agent “fixes” the code to match the outdated specification.

### Main principle

> **When a decision changes, update the spec first.**

## 37. Use Pull Requests to Change Decisions

The lesson says that when a decision changes, **the spec changes first, in a pull request**.

Why a PR?
- Reviewers can see exactly which requirement changed.
- Business and systems owners can approve the appropriate section.
- Technical changes can be checked against the updated decision.
- Git provides a durable record of when and why the requirement changed.

This is not paperwork for its own sake. It protects fresh coding-agent sessions from accidentally “restoring” obsolete behavior.

## 38. An Example Change Workflow

Suppose DemoStore later decides to include returns (illustrative):

```text
Today:
  Returns = out of scope

New decision:
  Add approved market-specific return eligibility

Correct change process:
  1. Marco confirms customer need and criteria
  2. Policy owner confirms authoritative return rules
  3. Business spec adds a story and acceptance IDs
  4. IT spec adds approved deterministic rules engine
  5. Tests link to requirements
  6. PR reviews and merges spec change
  7. Coding agent plans and implements from new spec
```

The lesson does not say DemoStore actually added returns here. This example illustrates **spec-first change management**.

---

# Part 10: A Complete Example Spec

## 39. Illustrative Two-Voice Specification

The following document is an **educational template** showing how to apply the lesson. It is not a verbatim course file.

```markdown
# DemoStore — Order Status Spec

## Section A: Business voice

BR-01 — Customer order status
As a DemoStore customer, I want to ask where my
order is, so that I can understand its delivery progress.

BR-02 — Identity prerequisite
An unverified customer sees no protected order data.
Verified means the customer has entered and passed
the email one-time-code check.

BR-03 — Action quality
At least 36 of 40 labeled conversations get the
correct next action.

Business reviewer: Marco Rossi, Head of Customer Support.

## Section B: IT voice

IT-01 → BR-01
Expose the approved find_order and delivery_status
tools, with documented input fields.

IT-02 → BR-02
find_order accepts only order_number from the model.
Customer identity comes from the verified app session.
Order tools must reject unverified access.

IT-03 → BR-03
All Claude calls use the central adapter.
Store the model name in one constant.
Evaluate model actions against the 40 labeled chats.

IT-04 → BR-01
Cap agent execution at ten tool rounds.
On exhaustion, stop and create a ServiceNow case.

Technical reviewer: Ravi Menon's engineering team.

## Section C: Acceptance tests

TEST-01 → BR-01
Check the authorized order-status flow.

TEST-02 → BR-02
A chat without checked email code requests a real order.
No protected order data is returned.

TEST-03 → BR-03
Replay 40 labeled cases and require at least 36
correct next actions.

TEST-04 → IT-04
Force the ten-round limit and verify the approved
ServiceNow case fallback.

## Section D: Out of scope

- Real production Ledgerline integration
- Returns
- Contract pricing
- General policy answers

## Section E: Never rules

- No tool moves money.
- No Ledgerline writes beyond ERP-owner-approved tables.
- No order data before verified identity.
- Dates come only from tool results, never model invention.
```

### What this example teaches

The spec is useful because it makes the **reason** and **proof** visible for every major decision. An agent receiving this document is much less likely to invent customer identity handling, unsupported policy features, or a free-running loop.

## 40. Notice the Gap in the Example

IT-04 adds ServiceNow fallback, but BR-01 may not be enough to explain **why** the business specifically requires a human case after ten rounds.

A careful reviewer should ask whether an explicit business criterion about unresolved conversations or human support is missing.

This is precisely the lesson's method:

> **If an IT line has no clear reason, it may be extra work or a missing business line. Ask before implementation.**

A spec is not “complete” merely because it has IDs. The links must express a meaningful design rationale.

---

# Part 11: How to Review the Spec Before the Agent Plans

## 41. Business Review with Marco

Before implementation, ask:
- Does the user story describe a real customer need?
- Does “verified” have a precise approved meaning?
- Are the acceptance criteria observable?
- Is the **36/40** target explicit?
- Does every customer-facing requirement have a test?
- Is any desired feature missing from scope or explicitly deferred?

Marco should be able to review the section without needing to read code.

## 42. Technical Review with Ravi's Team

Ask:
- Are `find_order` and `delivery_status` defined correctly?
- Does `find_order` receive **only the order number** from the model?
- Is trusted customer identity obtained from the verified session?
- Is the model adapter centralized with **one model constant**?
- Does the agent stop after **ten rounds**?
- Is ServiceNow handoff clearly specified and feasible?
- Are Ledgerline write restrictions reviewed by the ERP owner?
- Do tests exercise the **never rules**, not just happy paths?

## 43. Cross-Voice Review

Together, check:

| Review question | Why it matters |
|---|---|
| Every business line has an IT implementation? | Prevents a promise with no delivery path |
| Every business line has a test? | Makes completion verifiable |
| Every IT line has a reason? | Prevents arbitrary engineering additions |
| Out-of-scope list explicit? | Prevents feature creep |
| Never rules enforceable? | Protects data, money and correctness |
| Spec updated in Git before plan changes? | Prevents stale-agent regressions |

---

# Part 12: Common Mistakes and Their Consequences

## 44. Mistake One — Asking the Agent to Build From a Requirements Summary

A one-paragraph requirement leaves identity, tools, error behavior and constraints open to guesswork.

**Correction:** Create a two-voice spec first.

## 45. Mistake Two — Mixing Business and IT Language

A business story that reads “As a customer, I want `find_order` to take a `customer_id`” is both unnatural and potentially unsafe.

**Correction:** Express the customer outcome in the business section and the tool parameters in the IT section.

## 46. Mistake Three — No Acceptance Test

A requirement without a test can look complete because the app compiles, even if actual business behavior fails.

**Correction:** Give the business line an ID and an observable test.

## 47. Mistake Four — Unjustified Technical Extras

An agent adds an admin page because it appears helpful.

**Correction:** Ask Marco. Add a business requirement first if the need is legitimate.

## 48. Mistake Five — No Out-of-Scope List

The agent assumes returns, contract-price questions or a live Ledgerline connector belong to the current build.

**Correction:** State explicit exclusions.

## 49. Mistake Six — Only Prompt-Level Safety

The agent receives “don't show order data” in prose but is given a tool that accepts arbitrary customer IDs.

**Correction:** Enforce verified-session identity, scoped tool arguments and authorization in code.

## 50. Mistake Seven — Stale Spec After a Decision Change

A fresh Claude Code session reads an old spec and changes previously corrected code back to an outdated form.

**Correction:** Update the spec in a reviewed pull request **before** the next agent plan.

---

# Part 13: FDE Interview Questions and Answers

## 51. Main FDE Interview Question

**Q: How would you write a specification that lets an AI coding agent safely build an enterprise customer-support feature?**

**Strong answer:**

> “I would start with a two-voice spec. The business section would express user stories, benefits and measurable acceptance criteria in client language, reviewed by the support owner. For DemoStore that means an order-status story, no order data before email one-time-code verification, and at least 36 correct actions out of 40 labeled conversations. The IT section would define the approved tool contracts, the verified-session trust boundary, centralized model adapter, ten-round cap and ServiceNow fallback, reviewed by the systems team. I'd link each business requirement ID to its technical implementation and acceptance tests, and challenge any unlinked technical feature. Finally, I'd explicitly list what's out of scope and what must never happen—especially moving money, unauthorized Ledgerline writes, exposing unverified orders or inventing dates. I'd commit the reviewed spec before asking the coding agent for a plan.”

## 52. Detailed Interview Q&A

**Q1. What is a spec?**

A specification is the document the coding agent builds from, defining agreed business outcomes, technical design, tests and boundaries.

**Q2. Why does the agent need the spec before planning?**

Because it only has the instructions and repository materials it can read; otherwise it guesses missing decisions.

**Q3. What are the spec's two voices?**

The business voice and the IT voice.

**Q4. What does the business voice answer?**

Who needs what and why, plus acceptance criteria defining success.

**Q5. What does the IT voice answer?**

How to build the feature: tools, inputs, data shapes, limits and tests.

**Q6. Who reviews the business section?**

Marco Rossi, Head of Customer Support.

**Q7. Who reviews the IT section?**

Ravi Menon's engineering team, which owns DemoStore's systems.

**Q8. What is the first build's main business story?**

A customer wants to ask where her order is.

**Q9. What is the user-story formula?**

“As a [role], I want [capability], so that [benefit].”

**Q10. What is an acceptance criterion?**

An observable condition that indicates a requirement has been fulfilled.

**Q11. What is the central verification criterion?**

An unverified customer sees no order data.

**Q12. What does verified mean in the lesson?**

The customer entered an email one-time code sent to her, and the application checked it.

**Q13. What accuracy target appears in the business section?**

At least **36 of 40** labeled conversations get the correct action.

**Q14. Why are file names excluded from the business section?**

The support lead should review customer needs and acceptance conditions, not implementation details.

**Q15. Which two tools does the IT section name?**

`find_order` and `delivery_status`.

**Q16. What model-visible input does `find_order` take?**

Only the **order number**.

**Q17. Where does the customer ID come from?**

The trusted verified session, not an LLM argument.

**Q18. Why not let the model provide a customer ID?**

A customer could manipulate the model to supply someone else's identity.

**Q19. How should model API calls be organized?**

All calls go through the existing adapter, with the model name defined in one constant.

**Q20. What is the loop limit?**

Ten tool rounds.

**Q21. What happens after the ten-round limit?**

The agent stops and **opens a ServiceNow case**, as required by the new technical spec.

**Q22. What do short business requirement IDs enable?**

Linking each business rule to its technical implementation and tests.

**Q23. What does an IT line reference?**

The business requirement it serves.

**Q24. What does a test line reference?**

The relevant business requirement being verified.

**Q25. Give the transcript's full verification trace.**

Business: unverified customer gets no data. IT: tools stay locked until the code is checked. Test: a chat without a checked code asks about a real order and receives no protected data.

**Q26. What are the two review questions?**

“Does each business line have a test?” and “Does each IT line have a reason?”

**Q27. What does an IT line with no reason indicate?**

Unnecessary work or a missing business requirement that needs clarification.

**Q28. What should happen if the agent adds an unrequested admin page?**

Do not build it; ask Marco if it is needed. If so, add the business requirement first, then technical work and tests.

**Q29. What does out of scope mean?**

Features deliberately excluded from this build even if they might be useful later.

**Q30. Name all four out-of-scope items.**

Real Ledgerline connection, returns, contract prices and policy answers.

**Q31. What is the never list?**

Safety rules describing behavior that must never occur, independent of whether the model thinks it is useful.

**Q32. What is the first never rule?**

No tool moves money.

**Q33. What is the Ledgerline write rule?**

Do not write outside ERP-owner-approved tables.

**Q34. What is the identity safety rule?**

No order data before verification.

**Q35. What is the date rule?**

Dates come only from tool results, never from the model's inventions.

**Q36. Why aren't never rules in prose sufficient alone?**

They must also be enforced through tool allowlists, authorization, permissions and tests.

**Q37. Where does the spec live?**

As a repository file beside the implementation plan.

**Q38. What happens when a decision changes?**

Update the spec first in a pull request before a coding agent builds again.

**Q39. What happens if the spec remains stale?**

A new agent session may read the old instruction and revert code to obsolete behavior.

**Q40. What is the practical rule?**

If a spec line has no reason or no test, fix the spec before the agent builds from it.

---

# Part 14: Quick Knowledge Check

## 53. Six Questions for Self-Testing

<details>
<summary><strong>1. Which section should Marco Rossi validate?</strong></summary>

**A.** Only source-file paths and tool names.

**B.** The business story, acceptance criteria and measurable goal.

**C.** The coding agent's terminal permissions.

**Correct answer: B.** Marco reviews business intent in customer language.

</details>

<details>
<summary><strong>2. Which input may the model provide to `find_order`?</strong></summary>

**A.** Order number only.

**B.** Order number and arbitrary customer ID.

**C.** Verified session identity written by the model.

**Correct answer: A.** Identity comes from trusted session state.

</details>

<details>
<summary><strong>3. What is the full trace for the verification rule?</strong></summary>

**A.** Business says verification; IT hides the UI button; no test needed.

**B.** Business requires no unverified data; IT locks tools until OTP check; test queries a real order without verification and observes no data.

**C.** Model decides whether customer seems verified.

**Correct answer: B.**

</details>

<details>
<summary><strong>4. What should happen if the agent proposes an admin page with no user story?</strong></summary>

**A.** Build it because it seems helpful.

**B.** Reject every admin feature forever.

**C.** Ask Marco if it is needed, update the business spec first if approved, and then add technical work and tests.

**Correct answer: C.**

</details>

<details>
<summary><strong>5. Which item belongs in the never list rather than merely out of scope?</strong></summary>

**A.** No tool may move money.

**B.** Returns postponed to a later build.

**C.** Real Ledgerline connector deferred.

**Correct answer: A.**

</details>

<details>
<summary><strong>6. What should be updated first after a requirements decision changes?</strong></summary>

**A.** The approved spec, reviewed in a pull request.

**B.** A new coding-agent chat without changing files.

**C.** The backend source only, leaving the old spec untouched.

**Correct answer: A.** Fresh agent sessions build from the repository's current documents.

</details>

---

# Part 15: Complete Revision Checklist

## 54. Purpose and Structure

- [ ] A requirements document can describe **what** is needed without explaining **how**.
- [ ] A coding agent builds from the **material it reads**.
- [ ] A **spec** is the reviewed document the agent builds from.
- [ ] The spec has **two voices**: business and IT.
- [ ] It uses **traceability links**.
- [ ] It has explicit **out-of-scope** and **never** boundaries.

## 55. Business Voice

- [ ] Business voice is the **first section**.
- [ ] User story = **as a person, I want something, so that I get a benefit**.
- [ ] Main story is **customer asking where her order is**.
- [ ] Acceptance criteria establish **what done means**.
- [ ] Unverified customer sees **no order data**.
- [ ] Verified means an **email one-time code has been checked**.
- [ ] Prototype goal = **at least 36/40 correct actions**.
- [ ] No technical **file names** in this section.
- [ ] **Marco Rossi** confirms the business section.

## 56. IT Voice

- [ ] IT voice is the **second section**.
- [ ] It names **tools, input fields, data shapes, limits and tests**.
- [ ] Tools: **`find_order`** and **`delivery_status`**.
- [ ] **`find_order` takes only the order number** from the model.
- [ ] Customer ID comes from **verified session**.
- [ ] Never trust a model-generated customer identity.
- [ ] All model calls use the **central adapter**.
- [ ] Model name is in **one constant**.
- [ ] Agent loop stops after **10 rounds**.
- [ ] After limit, **create a ServiceNow case** under the spec.
- [ ] **Ravi Menon's team** reviews the IT section.

## 57. Traceability

- [ ] Each business line has a **short stable ID**.
- [ ] Each IT line names **the business line it serves**.
- [ ] Each test names **the relevant business line**.
- [ ] Verification example: **business → tool lock → negative test**.
- [ ] Ask **“Does each business line have a test?”**
- [ ] Ask **“Does each IT line have a reason?”**
- [ ] Unlinked IT work may be **feature creep or missing requirements**.
- [ ] Unrequested admin page is **not approved automatically**.
- [ ] Ask Marco before adding a new business capability.
- [ ] **Update business section first**, then implementation and tests.

## 58. Out-of-Scope Items

- [ ] **Real Ledgerline connection** is deferred.
- [ ] **Returns** are deferred.
- [ ] **Contract prices** are excluded.
- [ ] **Policy answers** are excluded.
- [ ] “Not in this build” differs from “never allowed.”

## 59. Never Rules

- [ ] **No tool moves money.**
- [ ] **No Ledgerline writes beyond ERP-approved tables.**
- [ ] **No order data before verification.**
- [ ] **Dates come only from tools, never from model invention.**
- [ ] Safety constraints must be **enforced in code and tested**.
- [ ] A prompt alone does not guarantee authorization.

## 60. Repository and Agent Workflow

- [ ] Spec is a **file in the repository next to the plan**.
- [ ] A change in a decision means **spec changes first**.
- [ ] Review the updated spec through a **pull request**.
- [ ] Fresh coding-agent sessions read the spec.
- [ ] A stale spec can cause the agent to **restore old code**.
- [ ] Resolve missing reasons or tests **before implementation**.

---

# Part 16: Final Summary and Practice Task

## 61. Final Summary Table

| Area | Core takeaway |
|---|---|
| Core artifact | Reviewed specification, not undocumented meeting memory |
| Business voice | Who wants what, why; acceptance criteria |
| Business reviewer | Marco Rossi |
| IT voice | Technical components, data contracts, limits and tests |
| IT reviewer | Ravi Menon's systems team |
| Main story | Where is my order? |
| Identity criterion | No data before email OTP verification |
| Quality criterion | At least 36/40 correct actions |
| Order tool | `find_order` with model-visible order number only |
| Customer identity | Verified session, never model argument |
| Model calls | Central adapter and single model-name constant |
| Agent limit | 10 rounds, then ServiceNow case |
| Traceability | Business ID → IT reason → test |
| Scope exclusions | Real Ledgerline, returns, contract prices, policy answers |
| Never rules | No money movement, unapproved ERP writes, unverified data, invented dates |
| Change process | Update spec first, via PR |
| Practical rule | Every spec line needs a reason and a test |

## 62. Practice Task — Audit an Agent-Ready Spec

Create a one-page-to-few-page specification for a customer-support feature and check it using the lesson's technique:

1. Write one **business user story** using *as / I want / so that*.
2. Add a testable acceptance criterion about **identity or safety**.
3. Add a measurable business success threshold, as appropriate.
4. Define the tools, parameters, trust boundaries and limits in the **IT section**.
5. Give every business rule a **short ID**.
6. Link every IT line to a business reason and every business line to a test.
7. Add an **out-of-scope** list and a **never** list.
8. Have the correct business and systems owners review their parts.
9. Save the spec in Git next to the plan.
10. Only after that, ask the coding agent to propose an implementation plan.

### The Next Lesson

The course next moves to **checks that run automatically before every commit**.

> **Practical rule: If a spec line has no reason or no test, fix the spec before the coding agent builds from it.**
