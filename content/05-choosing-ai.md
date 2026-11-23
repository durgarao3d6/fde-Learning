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
