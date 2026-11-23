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
