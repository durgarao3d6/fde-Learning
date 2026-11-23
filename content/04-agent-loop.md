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
