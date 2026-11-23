# Tool Calling: The Model Asks, Code Acts

## Learning objective
Understand the distinction between an LLM's requested tool action and the application's execution of that action.

## DemoStore tools
- `find_order` queries order data through custom middleware connected to the legacy **Ledgerline** ERP.
- `delivery_status` inspects shipping information from a carrier status file refreshed nightly.

The assistant might receive: “Where is the laptop I ordered?” It may need an order lookup followed by a delivery-status lookup. A customer's identity must be verified before showing private order data.

## The mechanism
A tool description contains its name, purpose and JSON parameter schema. The model does **not** directly execute database queries or read files; it emits a structured request.

```text
Customer message
    ↓
Claude chooses a tool (tool_use)
    ↓
Application validates and executes it
    ↓
Application sends tool_result back
    ↓
Claude produces an answer or requests another tool
```

The API's structured `stop_reason` distinguishes a tool request from a finished answer (e.g. `tool_use` versus `end_turn`). Do not parse ordinary prose to decide whether to execute tools.

## Conversation state
API calls are effectively stateless. The program passes the necessary message history, tool request and tool result into subsequent calls. Pair each tool result with its **tool_use_id**.

## Example
An order exists and the carrier file says it was scanned at a depot. A safe assistant reports the known scan and avoids fabricating a delivery date.

## Review questions
1. Who decides whether a tool actually runs?
2. Why is the tool schema not an authorization policy?
3. What is the role of `tool_use_id`?
4. Why must message and tool history be resent?

## Remember
**The model asks. The application validates and acts.**

---

# Detailed lesson notes

## 1. The precise boundary: model versus application
Tool calling is frequently misunderstood. Giving a model the description of `find_order` does **not** give the model direct database access. The model decides *which tool it would like to call* and supplies proposed structured arguments. **Your application** validates identity and arguments, invokes the corresponding function, and sends the result back.

The distinction is especially important for Forward Deployed Engineers, who integrate LLMs into enterprise systems. External actions must stay under deterministic program control.

## 2. Tool definition components
A typical tool definition has:
- **Name:** stable identifier such as `find_order`.
- **Description:** what it does and when it should be used.
- **Parameters:** JSON Schema-like description of required input fields.
- **Required fields and types:** keep calls machine-readable.

Illustrative definition:
```python
find_order_tool = {
    "name": "find_order",
    "description": "Find an order belonging to the verified session customer.",
    "input_schema": {
        "type": "object",
        "properties": {
            "order_number": {"type": "string"}
        },
        "required": ["order_number"]
    }
}
```

This is an educational representation, not the original source code.

## 3. The DemoStore integration problem
**Ledgerline** is a legacy ERP that contains order records. It lacks a convenient direct API for the proposed assistant, so a **custom middleware layer** bridges the application and ERP.

**Delivery-status data** comes from a **nightly carrier CSV file**, not a live, fully reliable ETA service. The freshness of the carrier data is part of the answer's limitations.

Customer asks: *“My laptop order hasn't arrived. Where is it?”*

Possible assistant steps:
1. Interpret order-status intent.
2. Request `find_order`.
3. Application checks session identity and order ownership.
4. Read the order's shipment/tracking reference.
5. Request `delivery_status`.
6. Application reads the recorded carrier status.
7. Assistant answers only from the returned evidence.

A single chat may therefore require multiple tool requests. The model should not fabricate absent tracking IDs.

## 4. `stop_reason` and the control loop
The API gives a **structured signal** indicating why a response stopped.

| `stop_reason` | Meaning | Application behavior |
|---|---|---|
| `tool_use` | Model wants one or more tools | Validate and execute approved calls |
| `end_turn` | Model completed an answer | Return answer to user |

Do **not** infer these states from a sentence like “I'll look that up.” A sentence has no authority to execute code.

Illustrative logic:
```python
if response.stop_reason == "tool_use":
    # Extract structured tool requests and dispatch through a safe allowlist.
    pass
elif response.stop_reason == "end_turn":
    # Present final text to the customer.
    pass
else:
    # Handle other stop conditions deliberately.
    pass
```

## 5. How the model learns the result
A tool result must be sent back as a message associated with the particular tool request, using its **`tool_use_id`**. That pairing matters when several tools are requested in a single model response.

The model does not automatically know that your Python function ran. The application has to **append the actual result** and make the next model request.

```text
messages = [customer]
Claude → tool_use(id="u1", name="find_order", input={...})
Python → find_order(...)
messages += [Claude's request, tool_result(tool_use_id="u1", result=...)]
Claude → tool_use(id="u2", name="delivery_status", input={...})
Python → delivery_status(...)
messages += [request, result]
Claude → final answer (end_turn)
```

The format is conceptual; actual SDK content blocks may differ.

## 6. Identity verification is a separate capability
DemoStore uses an **email one-time code** before returning personal order data. The model may request or explain verification, but it **must not decide** who is verified or accept a customer ID supplied by itself.

A legitimate session's verified customer is established by an application-controlled flow. A successful tool call then authorizes the requested order against that identity.

## 7. Stateless requests and conversation reconstruction
Every successive API call must include enough prior conversation context. This includes:
- Relevant customer messages.
- Earlier assistant messages and tool-use blocks.
- Matching tool results.
- Updated context needed to answer.

The model API does not magically preserve your application's session across separate requests.

## 8. Freshness and uncertainty
A carrier status such as “scanned at depot” does not establish a guaranteed arrival time. If the CSV has not refreshed since last night, explain that limitation. Reliability sometimes means saying **“I don't have a confirmed ETA.”**

## 9. Common mistakes
- Treating the model-generated tool arguments as trusted.
- Giving the model access to raw database credentials.
- Forgetting `tool_use_id` and mismatching results.
- Assuming tool calls execute themselves.
- Guessing shipment dates when a carrier tool has no ETA.
- Using a vague tool description that encourages unsafe calls.
- Losing history between successive model requests.

## 10. Interview-ready answer
**Question:** “What is tool calling?”

**Answer:** “The model requests a named tool with structured arguments. The host application validates and executes that tool, checks authorization, returns the result associated with its tool-use ID, and asks the model to continue. The tool schema describes capability; the application enforces permissions.”

### Revision checklist
- [ ] Explain the purpose of **name, description and schema**.
- [ ] Explain **`tool_use` versus `end_turn`**.
- [ ] Explain **`tool_use_id`**.
- [ ] Describe **Ledgerline middleware and nightly CSV**.
- [ ] Explain why API requests must resend history.
- [ ] Distinguish verified identity from model-generated arguments.
