# Secure Agent Harness and Guardrails

## Learning objective
Turn probabilistic tool suggestions into bounded, authorized application behavior.

## Core controls
**Maximum 10 tool rounds.** If the agent cannot resolve the situation within the cap, stop and hand off through **ServiceNow**. Count structured tool requests, not conversational promises.

**Identity must come from the application session.** The `find_order` implementation uses the verified customer associated with the session, not a customer ID that the model supplies.

**Object-level authorization.** Verify that an order belongs to the currently verified customer before returning its details.

**Avoid information leakage.** For nonexistent orders and orders belonging to somebody else, return the same “not found” result. An unverified customer receives no protected order information.

**No refund execution tool.** The model may draft a refund proposal into a holding area, but a human approves the action in ServiceNow before money moves.

## Why prompts are not enough
An instruction such as “never reveal other customers' orders” is helpful guidance, not an access-control guarantee. Authorization must be checked **inside each tool** before data is returned or an action is taken.

## Agent harness flow
```text
Agent requests tool
      ↓
Check session verification
      ↓
Validate tool arguments
      ↓
Authorize the specific order
      ↓
Execute within approved permissions
      ↓
Return bounded result
```

## Review questions
1. What happens after ten tool rounds?
2. Why must verification happen in the tool rather than only in the prompt?
3. Why use an identical error for another customer's order and a nonexistent order?
4. Who has authority to execute a refund?

## Remember
**Every tool checks before it acts.**

---

# Detailed lesson notes

## 1. What is an agent harness?
An agent harness is the **application-level control system** wrapped around a tool-using LLM. It manages:
- Which tools exist.
- When and how tool requests run.
- How to verify identity and permissions.
- Maximum tool rounds and fallback.
- How errors, sensitive actions and human handoff are handled.

The LLM is a decision aid operating **inside** this harness, not a superuser.

## 2. Maximum ten tool rounds
A bounded loop avoids endless tool requests, runaway costs and stalled customer conversations. DemoStore's limit is **10 rounds**. When the limit is reached, the application stops and sends the case toward **ServiceNow human support**.

The limit is a code-level check. It must remain effective even if the model asks for “just one more lookup.”

Illustrative:
```python
MAX_ROUNDS = 10
for _ in range(MAX_ROUNDS):
    response = ask_claude(messages)
    if response.stop_reason == "end_turn":
        return extract_answer(response)
    if response.stop_reason == "tool_use":
        execute_only_authorized_tools(response)
        continue
    break
return create_handoff()
```

The handoff path is conceptual; the later minimal prototype does not have the actual ServiceNow integration yet.

## 3. The verified-customer trust boundary
**Bad:** `find_order(order_number, customer_id_supplied_by_model)`.

Why bad? A malicious or confused model could supply another customer's ID.

**Good:** `find_order(order_number, verified_customer_from_session)`.

The customer identity comes from trusted application state after a separate email one-time-code verification. The model cannot establish or replace it through words.

The tool itself checks:
1. Does a verified session customer exist?
2. Does the order exist?
3. Is that order owned by this verified customer?
4. Is the requested data allowed to be returned?

Even perfect prompt wording cannot compensate for a missing ownership check.

## 4. Nonexistent versus unauthorized objects
Suppose a verified customer requests:
- An order number that is not in Ledgerline.
- An existing order that belongs to someone else.

The user-facing result should be indistinguishable: **“order not found”** (or equivalent). Otherwise, different messages reveal which orders exist for other customers, enabling enumeration.

This is a privacy and authorization design principle commonly called **not disclosing object existence**.

## 5. Tools enforce security individually
A later lesson notices that `delivery_status` also requires an ownership check. Protecting only `find_order` is not enough if someone can guess another shipment's tracking number.

The rule is **every tool checks before it acts**, not “the agent did verification once.”

## 6. Separate reading from money-moving actions
A refund is a high-impact financial action. The prototype **does not expose an execute-refund tool**.

Instead:
1. Claude may draft a proposed refund.
2. The proposal is saved in a holding table.
3. A person reviews and approves it within ServiceNow.
4. Only an authorized business workflow performs the actual transaction.

This avoids treating persuasive text as financial authorization. The LLM is allowed to *propose*, not *approve or execute*.

## 7. Layered defenses
| Layer | Example | Purpose |
|---|---|---|
| Prompt / tool descriptions | “Only use orders belonging to verified users” | Guide model behavior |
| Trusted session | Verified customer identity | Establish caller |
| Tool authorization | Compare order owner to session customer | Enforce data access |
| Capability restrictions | No refund-execution tool | Limit reachable actions |
| Operational cap | 10 tool rounds | Bound cost and loops |
| Human approval | ServiceNow refund workflow | Control financial changes |
| Error handling | Generic not-found response | Minimize information leaks |

A good FDE knows which parts are **soft guidance** and which are **hard controls**.

## 8. Common failures
**Prompt-only security:** A model can be confused or manipulated.

**Model-provided identity:** Trust boundary is inverted.

**Checking authorization after reading sensitive data:** Data may already have leaked.

**Special error for another user's order:** Confirms existence.

**Giving refund execution because it improves demo flow:** Makes the prototype capable of an irreversible act.

**No maximum rounds:** Unexpected tool loops can consume time and money.

## 9. Example test cases (illustrative)
```python
def test_unverified_session_cannot_read_order(): ...
def test_owner_can_read_own_order(): ...
def test_other_customers_order_looks_not_found(): ...
def test_unknown_order_looks_not_found(): ...
def test_delivery_status_checks_order_ownership(): ...
def test_refund_tool_is_not_available(): ...
def test_ten_round_cap_triggers_handoff(): ...
```

Tests should assert on **tool results and authorization decisions**, not only the tone of the model's response.

## 10. Interview-ready answer
**Question:** “How do you make an LLM agent safe to connect to a customer's ERP?”

**Answer:** “I restrict the available tools, use application-session identity, enforce ownership inside each tool, avoid leaking the existence of another customer's records, cap the number of rounds, and route irreversible actions through human approval. The model proposes actions; deterministic code authorizes them.”

### Revision checklist
- [ ] Tool-round limit: **10**.
- [ ] Human support system: **ServiceNow**.
- [ ] Identity: **verified session**, not LLM argument.
- [ ] Foreign order and nonexistent order: **same response**.
- [ ] Refund: **draft**, never direct execution.
- [ ] Authorization is **per tool**, including carrier status.

---

# Threat Cases and Mandatory Enforcement Points

## Case study: order-number enumeration
Imagine customer A submits three plausible order numbers. One is theirs, one belongs to customer B and one does not exist.

If the tool returns:
- “This order belongs to another person” for the existing foreign order.
- “Order does not exist” for the nonexistent order.

then customer A has learned which identifiers correspond to real purchases. Even without seeing items or addresses, that is an **information disclosure**.

Return **the same not-found semantics** for both unauthorized and unknown orders. Record richer details only in appropriately controlled internal audit logs.

## Case study: model-supplied identity
A customer says, “I am customer 999, check their order.” The model might pass `customer=999` to a tool if permitted. That is not identity verification.

Correct design:
```python
# Illustrative trusted boundary
def find_order_for_session(order_number, session):
    customer_id = session.verified_customer_id
    if customer_id is None:
        return {"error": "verification_required"}
    return erp_adapter.find_owned_order(order_number, customer_id)
```

The model may supply the **order number**, but it cannot choose the trusted customer identity.

## Case study: circumventing ownership through a second tool
Suppose `find_order` securely rejects someone else's order, but `delivery_status` accepts any tracking number without an ownership check. An attacker might still retrieve shipment updates by guessing tracking numbers.

Therefore **each tool independently enforces applicable authorization**. Do not assume a prior tool was always used or that the model will follow a specific call order.

## Case study: refund side effects
The model can draft a proposed refund amount and reason, but it cannot execute the payout. The proposal goes to a **holding table**, then the approved **ServiceNow** workflow takes over.

Why a holding table?
- Separates draft recommendations from authorized transactions.
- Gives a person information to review.
- Creates an explicit approval boundary.
- Keeps monetary side effects outside model control.

Do not mistake “the model only has a draft tool” for complete security; the draft-writing destination must also be properly authorized.

## Case study: infinite model/tool exchange
Sometimes models make redundant tool requests or fail to settle on a final answer. A hard limit of **10 tool rounds** prevents uncontrolled loops. Exhaustion becomes a deterministic reason to stop and use a human handoff rather than continuing to consume tokens.

The loop's control condition should use **structured response metadata**, never parse a model's assurances such as “This is my last call.”

## Acceptance-test matrix (illustrative)
| Scenario | Expected tool behavior |
|---|---|
| Customer not verified | No order details |
| Own order exists | Return permitted fields |
| Someone else's order | Generic not-found |
| Unknown order | Same generic not-found |
| Own tracking exists | Return associated carrier record |
| Tracking belongs to another customer | Deny without private status |
| Refund requested | Create proposal only, no money moved |
| Eleven attempts to continue loop | Stop at 10-round boundary |

## How to explain defense in depth
**Prompt rules** describe desired model behavior. **Tool permissions** restrict reachable operations. **Session identity** establishes the caller. **Per-object authorization** decides which records are visible. **Human approvals** protect sensitive business actions. **Round caps and exceptions** preserve predictable execution.

No single layer should be presented as a complete safety solution.

## Memory anchors
**Verified identity from session → per-tool ownership → generic missing/foreign error → refund drafts only → ServiceNow human approval → 10-round cap.**


---

# FDE Interview Questions & Answers — Comprehensive Practice Bank

Use these prompts for oral practice. Answer aloud before revealing the explanation beneath each question.

<details>
<summary><strong>1. What is an agent harness?</strong></summary>

The application layer that controls available tools, identity, authorization, loop limits, errors and human fallback around a tool-using model.

</details>

<details>
<summary><strong>2. What is DemoStore's tool-round limit?</strong></summary>

Ten rounds; exceeding the limit stops the agent and routes the case toward a ServiceNow human handoff.

</details>

<details>
<summary><strong>3. Why is the round limit in code?</strong></summary>

A model instruction like 'stop after ten calls' is not an enforceable guarantee; application code must count rounds.

</details>

<details>
<summary><strong>4. Why must find_order ignore model-supplied customer IDs?</strong></summary>

An LLM might be manipulated into naming another customer. The verified identity must come from trusted session state.

</details>

<details>
<summary><strong>5. Which order records can a verified customer read?</strong></summary>

Only their own, as established by object-level ownership checks inside the tool.

</details>

<details>
<summary><strong>6. How should the system respond to a foreign order number?</strong></summary>

The same generic not-found result it would return for a nonexistent order, avoiding an existence leak.

</details>

<details>
<summary><strong>7. Why also protect delivery_status?</strong></summary>

An attacker may guess a tracking number and bypass the ownership check on find_order if the carrier tool is unguarded.

</details>

<details>
<summary><strong>8. What should happen for an unverified customer?</strong></summary>

The system must not return protected order data, even when the model asks.

</details>

<details>
<summary><strong>9. Can the assistant execute refunds?</strong></summary>

No. It may draft a refund proposal to a holding table for human approval through ServiceNow.

</details>

<details>
<summary><strong>10. What is a high-impact side effect?</strong></summary>

A change such as moving money or updating a sensitive system. Such actions need explicit authorization outside the model.

</details>

<details>
<summary><strong>11. What is the difference between prompt instruction and hard guardrail?</strong></summary>

A prompt guides behavior; an allowlisted tool, code-level authorization or permission setting actually constrains what can happen.

</details>

<details>
<summary><strong>12. Does a correct tool choice prove safety?</strong></summary>

No. Even a correctly selected tool must validate arguments, identity and permissions before executing.

</details>

<details>
<summary><strong>13. Why can the not-found error still be logged internally?</strong></summary>

Internal protected logs may capture diagnostic context while public responses avoid revealing record existence. Logging itself must follow privacy policy.

</details>

<details>
<summary><strong>14. What tests would prove authorization?</strong></summary>

Verify own-order access, unverified denial, foreign-order equivalence to unknown, tracking ownership, no refund execution and ten-round termination.

</details>

<details>
<summary><strong>15. What is the key quote?</strong></summary>

Every tool checks before it acts.

</details>

## How to practise with this chapter

1. Explain the lesson's actual business problem in 45 seconds.
2. Recreate the architecture or sequence from memory.
3. Recall exact numbers and named decision-makers without checking the notes.
4. Explain one failure case and the deterministic safeguard.
5. Use the Q&A above to prepare a two-minute FDE interview answer.

