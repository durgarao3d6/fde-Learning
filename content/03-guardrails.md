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
