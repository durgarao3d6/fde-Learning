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
