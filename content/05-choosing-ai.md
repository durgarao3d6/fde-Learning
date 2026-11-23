# Choosing Whether to Use AI at All

## Learning objective
Select the simplest effective solution rather than automatically building a full agent.

## Four levels of intervention

### 1. No AI: self-service
Roughly **four in ten** DemoStore support chats ask about order status. A tracking page, proactive delay alert or direct carrier link may resolve many of these requests without any model call.

### 2. Deterministic software
Use code for precise return and refund eligibility. Inputs may include market, delivery date, contract and the version of the policy. Illustrative example: a UK rule has a 14-day window, so day 15 is ineligible under that stated rule. Real legal and policy requirements must be independently verified.

### 3. One structured LLM call
Use the model to extract intent, order number and handoff flag. Let application code choose the next step in a fixed workflow. A separate call may optionally generate user-friendly wording.

### 4. Full agent
Use a multi-step dynamic tool loop only for complex, ambiguous or multi-issue conversations where the next action depends on intermediate evidence.

## A useful question
**Could a web page or a simple rule solve the customer's problem better?** If yes, using an agent may add latency, cost and failure modes without improving the outcome.

## Governance example
An airline chatbot invented a refund-policy promise, and the airline faced responsibility for the answer. The lesson uses this to explain why official policy decisions should be enforced by deterministic systems.

## Review questions
1. Which support cases need no AI?
2. When is deterministic code preferable to a prompt?
3. What distinguishes structured extraction from a full agent?
4. How does a more complex architecture create new risks?

## Remember
**Use the least complicated approach that produces the business result.**

---

# Detailed notes: deciding the right automation level

## 1. Why this question matters for an FDE
A Forward Deployed Engineer is not hired simply to deploy the most advanced model. The job is to improve a real workflow, often under time, privacy and cost constraints. Before using a model, ask **what problem is being solved and whether AI is necessary**.

A useful decision ladder is **no AI → deterministic code → one structured LLM call → autonomous agent**. Complexity should increase only if measured business needs justify it.

## 2. Level one: no AI at all
DemoStore finds about **40% (four in ten)** support chats concern order status. If a customer wants tracking information, the highest-value product intervention might be:
- A clearly visible order-tracking page.
- A reliable, direct link to the carrier.
- A proactive notice when shipping is delayed.
- Better order emails with accessible delivery updates.

These are not inferior to an agent. They may have less latency, almost no per-chat inference cost, fewer failure cases and less support friction.

**Important nuance:** If the carrier's source file updates nightly, even a self-service system must communicate data freshness and avoid implying a live view.

### Example decision question
“Could the customer solve this in one click without needing to explain the issue to an AI?” If yes, test the self-service improvement first.

## 3. Level two: deterministic program logic
Use code when answers depend on explicit policy conditions rather than open-ended language interpretation.

The course's return-policy inputs include:
- Customer market or jurisdiction.
- Actual delivery date.
- Contract or special agreement.
- Applicable version of the policy.

An illustrative eligibility function:
```python
from datetime import date

def eligible_within_window(delivery_date: date, request_date: date, days: int) -> bool:
    age = (request_date - delivery_date).days
    return 0 <= age <= days
```

**This is only a simplified example.** A real returns engine must handle exceptions, weekends or time zones where relevant, statutory rights, policy versions, products, special agreements and approved legal rules.

The lesson's **UK 14-day, day-15 ineligible** example is an illustrative policy rule, not a universal statement of all UK consumer-law rights.

Why not prompt Claude with “Our return window is 14 days, decide if this is eligible”? Because the model might be inconsistent, overlook market-specific policy, or misapply dates.

### Airline warning
The lesson refers to an airline support chatbot that incorrectly described refund terms, and the airline bore responsibility for the representation. The practical point: **customer-facing AI statements can create real business obligations**, even if a team calls them merely an LLM response.

## 4. Level three: one LLM call for extraction
A single model call is valuable when customers express the same intent in many ways:
- “Where's my parcel?”
- “My laptop never turned up.”
- “Still waiting for order 123.”
- “Can someone tell me where the delivery is?”

Instead of making the LLM manage an entire workflow, ask it to return a **structured classification**:
```json
{
  "intent": "order_status",
  "order_number": "123",
  "needs_human": false
}
```

The application then decides the **fixed next action**. For example:
```text
customer message
    ↓
one LLM intent extraction
    ↓
validated structured result
    ↓
deterministic code chooses workflow
    ↓
authorized data lookup / handoff
    ↓
template response or optional second wording call
```

The extraction class names are illustrative; actual controlled values must match the shared project schema.

**Benefit:** The model handles flexible language while the code owns policy and side effects.

## 5. Level four: agent loop
Use a dynamic sequence when the next tool depends on intermediate results. A support chat might combine:
- An order that is marked shipped.
- A delivery complaint.
- A possible return.
- A request for a person.

The agent may need to inspect several pieces of evidence before knowing what to do next. This is where tool-calling with a bounded loop may be warranted.

**Agent ≠ default.** A multi-step agent increases token use, tool-call costs, debugging complexity, authorization surfaces and potential failure paths.

## 6. Practical comparison
| Level | Good fit | Main advantage | Main failure concern |
|---|---|---|---|
| Self-service/no AI | Simple status queries | Fast, cheap, direct | Bad or stale underlying information |
| Deterministic code | Return/refund eligibility | Consistent, testable | Incorrectly configured policy |
| One LLM extraction | Messy customer phrasing | Language flexibility | Misclassification |
| Full agent | Complex multi-issue cases | Dynamic evidence gathering | Tool errors, loops, prompt injection |

## 7. Decision workflow an FDE can use
1. Classify the business problem and frequency.
2. Quantify value of resolving it.
3. Try simple UX/process improvements first.
4. Identify decisions governed by precise rules.
5. Reserve AI for ambiguous language or complex reasoning.
6. Restrict model actions to supported application capabilities.
7. Measure outcomes on **real representative cases**.
8. Check whether benefit justifies additional operational complexity.

## 8. Common mistakes
**“We have an AI budget, therefore every step must use AI.”** Business value is the target, not token consumption.

**Using a model as a policy engine.** Prompts are not an approved returns-rule database.

**Using an agent for a single lookup.** Adds complexity without a dynamic decision need.

**Treating customer satisfaction with a pleasant answer as proof of correctness.** A fluent false refund promise is still a failure.

## 9. Interview scenario
**Question:** “A client wants an AI agent for 1,600 weekly tracking questions. How would you respond?”

**Answer:** “I would first investigate whether better tracking pages and proactive notifications could deflect routine queries. I would keep exact return and refund decisions in code, use structured extraction if natural language is needed, and reserve an agent for cases that genuinely require multiple dependent tool calls. I would compare actual resolution, errors, handoffs and costs rather than maximizing AI usage.”

### Revision checklist
- [ ] Four architecture levels in order.
- [ ] **Four out of ten** order-status chats.
- [ ] Policy inputs: market, delivery date, contract, policy version.
- [ ] **UK 14-day/day-15** example is illustrative.
- [ ] Extraction model proposes **intent**, code chooses action.
- [ ] Full agent only where sequence is **dynamic**.
- [ ] Airline refund-policy warning.

---

# Worked Architecture Decision Exercises

## Exercise A — Order status, one straightforward question
A customer wants a tracking link for an order already visible in their account.

**Best initial design:** self-service tracking. No model necessary.

**What to measure:** share of customers finding tracking, reduced support contacts, return visits with the same issue, freshness of carrier data.

**Why not an agent?** There is no unresolved decision requiring dynamic tool planning.

## Exercise B — Return eligibility under an explicit policy
A market-specific policy grants eligibility within a specified number of days, subject to contract and product exceptions.

**Best design:** deterministic rule evaluation with approved policy data.

**What can AI do?** Extract the customer's question or explain the *already-determined* result in plain language. It must not substitute its own imagined law or policy.

**Important nuance:** The lesson's UK 14-day/day-15 example is a policy illustration and does not replace legal review.

## Exercise C — Messy order-status message
User: “Still waiting on that laptop from last week, can anybody see what's going on?”

**Best design:** one structured LLM extraction may identify order status, missing order number and whether the case needs a human. A fixed workflow then uses authorized tools.

**Why not make the model run the complete system?** The steps may be predictable once the intent is known.

## Exercise D — Multi-problem conversation
User: “My late parcel is damaged, I ordered two items and want a refund for one but still need the other delivered.”

**Potentially appropriate:** bounded multi-tool agent that discovers the orders and issue details, then routes to deterministic return/refund checks and human approval.

**Still forbidden:** allowing the model to send an actual refund merely because it concludes one is appropriate.

## Architectural decision record (example)
```markdown
Decision: use deterministic return rules.
Context: policy differs by market, dates and contract.
Alternative: ask model to decide refund eligibility.
Reason: consistent, testable policy execution is necessary.
AI role: extract request intent and explain policy outcome.
Safeguard: human approval for financial actions.
```

The record is illustrative; the value is making the decision defensible.

## Cost and risk comparison
An FDE should explain that AI complexity adds **inference cost, latency, extra failure modes, security surface and operational monitoring**. Those costs may be justified for complex chats, but not for a simple “show tracking” link.

## Final question to remember
**“What is the simplest intervention that improves the actual customer outcome?”** The answer may be no model, one model call, deterministic logic or a full agent—and that choice should follow evidence.
