# Implementing the Agent Loop

## Learning objective
Implement the control loop that dispatches model-requested tools and returns their results.

## Files and data
- `tools.py`: mock ERP JSON, carrier CSV and the corresponding tool functions.
- `agent.py`: tool definitions, dispatching and repeated model calls.
- `find_order(order_number, customer)`: checks order ownership.
- `delivery_status(tracking_number)`: looks up a carrier file record.

## Implementation outline
```python
messages = [customer_message]

for round_number in range(10):
    response = call_claude(messages, tools=tool_definitions)
    messages.append(response.content)

    if response.stop_reason == "end_turn":
        return final_text(response)

    if response.stop_reason != "tool_use":
        return safe_fallback()

    for request in extract_tool_requests(response):
        result = run_tool_with_checks(request)
        messages.append(tool_result(request.id, result))

return hand_off_to_human()
```

The pseudocode expresses the control flow, not a drop-in SDK implementation.

## Debugger example
A shipment's carrier record reports a delay and a scan at a **Leeds depot**. The agent should identify the status and acknowledge that the file does **not** establish a reliable new ETA. It should not fabricate one.

## Hardening tasks
The lesson's simple demo still needs:
- Ownership verification on `delivery_status`, not just `find_order`.
- Exception handling when files, tools or external systems fail.
- The ten-round cap and handoff behavior.
- Tests for missing or nonexistent tracking records.

A missing tracking number is **not** permission to invent a status.

## Review questions
1. What belongs in the dispatcher?
2. What does `end_turn` mean?
3. How do you prevent an infinite tool loop?
4. What is the correct answer when no tracking result exists?

## Remember
**An agent is a controlled loop, not a model with unlimited tool permissions.**

---

# Detailed lesson notes

## 1. Transition from tools to agent
A single tool call is not yet an agent. An agent handles a sequence in which the **next step depends on tool results**. In DemoStore's customer-support case, a message may require order lookup, tracking lookup and a final answer. The application must orchestrate these exchanges without handing over arbitrary code execution to the model.

## 2. File responsibilities
### `tools.py`
Includes **mock order data** in a JSON file and **carrier status data** in CSV. It implements `find_order(order_number, customer)` and `delivery_status(tracking_number)`.

The order lookup enforces ownership: if an order exists but belongs to somebody else, the tool behaves as though it is not found.

### `agent.py`
Defines the tool schemas the model sees, a `run_tool` dispatcher, and the repeated request/execute/result loop.

### `llm.py`
Adapter that makes calls to Claude. The agent should not duplicate model API access in every module.

## 3. The complete tool loop
```text
customer message
      ↓
messages = [customer turn]
      ↓
call Claude with messages + tool definitions
      ↓
inspect structured stop_reason
      ├── end_turn ──→ return final answer
      └── tool_use
             ↓
        read requested tool(s)
             ↓
        validate and dispatch via run_tool
             ↓
        append tool_result referencing tool_use_id
             ↓
        call Claude again with updated messages
```

Do not stop simply because the assistant wrote an explanatory paragraph along with a tool request. Use `stop_reason`.

## 4. Dispatcher example (illustrative)
```python
def run_tool(name, args, verified_customer):
    if name == "find_order":
        if verified_customer is None:
            return {"error": "Verification required"}
        return find_order(
            order_number=args["order_number"],
            customer=verified_customer,
        )
    if name == "delivery_status":
        # A production-safe implementation must authorize the shipment.
        return delivery_status(args["tracking_number"])
    return {"error": "Unsupported tool"}
```

**Caution:** This illustrative dispatcher intentionally highlights the lesson's remaining gap: **a carrier lookup also needs authorization**. Do not ship it as secure production code unchanged.

## 5. The Leeds depot debugging case
The customer wants to know where an order is. The order tool finds a shipment record. The carrier file reports a delayed scan at a **Leeds depot**.

The correct answer should:
- State the status as supported by the carrier record.
- Explain the shipment appears delayed.
- Avoid promising an ETA the carrier did not supply.
- Be clear about the data's freshness if it is from a nightly feed.

An invented ETA would make the support experience sound more helpful but would be less accurate.

## 6. Tool results are not executable instructions
Treat external tool data as **data**. A carrier CSV field might contain unexpected strings. It must not acquire authority to change business rules or run unapproved tools.

The application should validate result shapes and handle missing/invalid records deliberately.

## 7. Negative-path exercise
The transcript asks about nonexistent tracking information. The assistant must not hallucinate a package location. Good alternatives include:
- “I couldn't find a carrier record for that tracking number.”
- “I don't have a confirmed update.”
- A handoff or request for corrected details, if the workflow allows it.

What matters is **honest uncertainty**.

## 8. Production hardening mentioned in the lesson
**Ownership check on delivery status.** The carrier tool must be tied to the verified customer's shipment.

**Exceptions.** Missing files, corrupt CSV data, failed APIs and timeouts should not crash the entire chat service.

**Ten-round cap.** The model cannot loop indefinitely.

**Human fallback.** The assistant needs a safe path when the required information is unavailable.

### Tests to add
```text
✓ Known order belongs to verified user
✓ Other customer's order is hidden
✓ Missing order returns safe not-found
✓ Known tracking returns exact stored status
✓ Unknown tracking does not produce fabricated ETA
✓ Tool failure becomes a controlled error
✓ Overlong tool loop stops safely
```

## 9. Typical conceptual error
An LLM calling `delivery_status` is **not** the same as the LLM directly browsing carrier data. The model cannot assume that a requested function succeeded until it receives the **actual tool result**.

## 10. Interview-ready answer
**Question:** “Walk me through a production-oriented agent loop.”

**Answer:** “I send the user's conversation and approved tool schemas to the model, inspect the stop reason, execute requested calls through a validated allowlist with per-tool authorization, return each result under its tool-use ID, and repeat under a bounded round limit. On a final answer, I return the text; on missing data, failures or exhausted rounds, I use a controlled fallback or human escalation.”

### Revision checklist
- [ ] `tools.py` uses **JSON order fixture** and **carrier CSV**.
- [ ] `agent.py` owns the **dispatcher and loop**.
- [ ] Tool result links via **`tool_use_id`**.
- [ ] Leeds delay: **no invented ETA**.
- [ ] Missing tracking: **no fabrication**.
- [ ] Add **exception handling, ownership verification, 10-round cap**.
