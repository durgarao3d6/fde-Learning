# **Complete Study Notes: Building DemoStore's Chat Service and Widget with Claude Code**

FDE Course · AI SDLC · Claude Code · FastAPI · React · TypeScript · Pydantic · Agent Sessions · Plan Mode · Testing · Git

## **1. What Is This Lesson About?**

In earlier lessons, we built individual parts of DemoStore's AI assistant:

- `llm.py` — the adapter that calls Claude.
- `intent.py` — the structured intent definition.
- `agent.py` — the agent loop.
- Order tools — functions that retrieve order and delivery information.

These small programs already work.

But DemoStore needs more than a collection of scripts.

Customers need a **chat window on the website**, and that window needs a backend service that communicates with the AI assistant.

This lesson shows how to use Claude Code to turn existing scripts into an application, without losing engineering control.

The central principle is:

> **Give the coding agent a short brief and your project rules. Review its plan and questions before allowing it to write code.**

The lesson demonstrates the planning process, seven implementation phases, and independent verification of the first completed phase.

# **Part 1: What We Have and What We Want**

## **2. Existing Components**

| Component   | Purpose                                 | Current status                                    |
| ----------- | --------------------------------------- | ------------------------------------------------- |
| `llm.py`    | Calls Claude through an adapter         | Working                                           |
| `intent.py` | Defines the structure of model output   | Working, but schema is handwritten                |
| `agent.py`  | Runs the tool-calling agent loop        | Working, but does not accept conversation history |
| Order tools | Retrieve order and delivery information | Working with sample data                          |

These components prove that the basic AI functionality works.

However, they are not yet a customer-facing application.

## **3. Target Architecture**

The intended system connects a website widget to the existing agent through a backend service.

**DemoStore customer**

Sends a message through the website

**React + TypeScript chat widget**

Messages, typing indicator, human-support button

**FastAPI chat service**

Session management and API responses

**Existing agent loop**

Intent recognition and tool calling

`llm.py`** → Claude**

Centralized model API access

The service also manages session history, and the agent can use its existing order and delivery tools when appropriate.

# **Part 2: The Five-Part Implementation Brief**

The team uses the brief prepared in the previous lesson's interview.

It has been reduced to five implementation requirements.

## **4. Requirement One — FastAPI Chat Service**

The backend uses **FastAPI**, a Python framework for building web APIs.

The service must accept a customer message and return three things:

1. The assistant's reply.
2. The detected intent.
3. A session ID.

### **Why return a session ID?**

HTTP requests are normally independent.

Without a way to associate requests, the backend cannot reliably determine which messages belong to the same conversation.

A session ID allows the service to find the correct conversation history.

### **Illustrative API request**

```

{
  "session_id": "session-123",
  "message": "Where is my order?"
}

```

For a new conversation, the session ID might be omitted.

### **Illustrative API response**

```

{
  "session_id": "session-123",
  "reply": "I can help explain the next steps, but order lookup requires verification.",
  "intent": {
    "intent": "order_status",
    "order_number": null,
    "needs_human": false,
    "language": "en",
    "summary": "Customer asks about order status."
  }
}

```

These JSON examples illustrate the contract; the transcript does not provide exact endpoint names or JSON payloads.

### **Key point**

**The session ID identifies the conversation, while the intent describes what the customer wants.**

They serve different purposes.

# **Part 3: AI Disclosure**

## **5. Requirement Two — Tell Customers They Are Talking to AI**

The first assistant reply must clearly state:

- The customer is interacting with an AI.
- The customer can ask for a person at any time.

The lesson connects AI disclosure to the EU AI Act.

The human-support option is DemoStore's own product requirement.

### **Example first reply**

> "Hello! I'm DemoStore's AI assistant. You can ask to speak to a person at any time."

### **Why is this important?**

The customer should understand the nature of the interaction.

The application should not pretend to be a human support agent.

### **Implementation implication**

The backend or widget needs a reliable way to display the AI notice at the beginning of the conversation.

It should not depend on the model remembering to mention the disclosure spontaneously.

That last point is an engineering recommendation consistent with the requirement.

# **Part 4: One Shared Pydantic Intent Class**

## **6. Requirement Three — Define the Intent Once**

The transcript introduces **Pydantic**.

Pydantic is a Python library used to define, validate, and serialize structured data.

Instead of writing the intent schema manually in multiple places, the team defines one Python class.

### **Why?**

Three components depend on the intent structure:

- Claude's structured output.
- The chat service's response to the widget.
- The API documentation generated by the service.

If the team maintains multiple copies, they can become inconsistent.

## **7. The Single Source of Truth Principle**

**One Pydantic intent class**

Fields, types, and allowed values

Claude schema

API response

API docs

### **Example problem with duplicate schemas**

Suppose the engineer adds:

```
language: str
```

to the model's schema but forgets to add it to the backend response schema.

The model may return the language, but the backend may reject the data or fail to expose the field correctly.

The transcript describes this as a risk that could break every chat.

### **Better approach**

Define the field once and derive the necessary schemas from the same class.

This reduces synchronization mistakes.

## **8. The New Language Field**

The lesson specifically adds:

**The language of the customer's message.**

For example:

```
{
  "language": "en"
}
```

or:

```
{
  "language": "de"
}
```

The transcript does not prescribe the exact language-code format.

### **Why include language?**

The information may help a human support agent understand which language the customer is using when a real handoff is implemented.

# **Part 5: The Chat Widget**

## **9. Requirement Four — React and TypeScript**

The frontend uses:

- **React:** A library for building user interfaces.
- **TypeScript:** JavaScript with static type checking.

The widget must show:

- Conversation messages.
- A typing indicator.
- A persistent "Talk to a person" button.

### **Why TypeScript?**

TypeScript helps catch certain data-shape and type errors during development.

For example, the frontend can define the expected structure of the API response.

This does not replace runtime validation, but it improves development-time consistency.

## **10. The Typing Indicator**

The transcript requires a visible indication that the assistant is preparing a reply.

Without this indicator, the user may believe the interface is unresponsive.

Where is my order?

Assistant is preparing a reply…

 Talk to a person

Illustrative widget layout. The lesson's first version does not yet implement a real human transfer.

### **Important distinction**

A typing indicator communicates that a response is pending.

It does not guarantee that the response will arrive successfully.

A real implementation should also handle failures and timeouts, although those details are not specified in this transcript.

## **11. Why Must the Human-Support Button Always Be Visible?**

The transcript explains the user concern:

> Customers fear that a bot will prevent them from reaching a person.

Therefore, the "Talk to a person" button must never disappear.

This is a user-experience requirement, not merely a decorative design choice.

### **Why does visibility matter?**

If the option is hidden behind several menus or only appears after repeated failures, customers may feel trapped.

A permanently visible control provides reassurance and a clear escape path.

### **Important limitation**

From the previous lesson, the first build's button does not yet create a real ServiceNow handoff.

The interface must represent that limitation accurately.

# **Part 6: Live Tests That Call Claude**

## **12. Requirement Five — Real Model Integration Tests**

The team needs tests that actually call Claude.

These verify that the integration works with the real model rather than only with fake responses.

However, they should not run automatically with every fast test.

### **Why?**

Live model tests can be:

- Slower.
- More expensive.
- Dependent on API availability.
- Less deterministic than ordinary unit tests.

The transcript says they are marked `live`.

Fast tests skip them.

## **13. Illustrative Pytest Marking**

```
import pytest

@pytest.mark.live
def test_real_claude_response():
    # Make a real Claude API request.
    # Check the response against the intent schema.
    ...
```

An illustrative fast-test command:

```
python -m pytest -m "not live"
```

An illustrative live-test command:

```
python -m pytest -m live
```

The exact commands depend on the repository's configuration.

### **Key distinction**

Fast tests should be suitable for frequent local execution.

Live tests should be run deliberately.

# **Part 7: The Two Additional Technical Limits**

## **14. Limit One — Every Claude Call Goes Through **`llm.py`

The brief requires all model API calls to use the existing adapter.

### **Why?**

Centralizing model access makes it easier to manage:

- Model configuration.
- Authentication.
- Request formatting.
- Error handling.
- Logging.
- Future provider changes.

The transcript specifically requires the centralization; the listed benefits are engineering implications.

### **Bad architecture**

```
chat_service.py ──→ Claude API
agent.py ─────────→ Claude API
intent.py ────────→ Claude API
```

Different files implement their own API calls.

### **Preferred architecture**

```
chat_service.py ──┐
agent.py ─────────┼──→ llm.py ──→ Claude API
intent.py ────────┘
```

The adapter becomes the single model-access layer.

## **15. Limit Two — Sessions Stay in Memory**

The service stores sessions only while the process is running.

### **Consequence**

If the service restarts, the conversation history disappears.

### **Why accept this?**

The team is building an early internal version.

Persistent session storage is outside the current scope.

### **Important distinction**

A session ID is not the same as persistent storage.

A session ID identifies a conversation.

The storage implementation determines whether the conversation survives a restart.

# **Part 8: Project Rules and Permissions**

## **16. **`CLAUDE.md`** Rules**

Alongside the brief, the agent receives project rules.

The transcript specifies that `CLAUDE.md` documents:

- How to run fast tests.
- That every model call must go through `llm.py`.

These instructions keep the coding agent aligned with the repository's architecture.

## **17. Permission Settings**

The settings file divides actions by permission level.

| Action                    | Permission            |
| ------------------------- | --------------------- |
| Run approved tests        | Allowed automatically |
| Run approved Git commands | Allowed automatically |
| Install a package         | Ask first             |
| Read the secrets file     | Blocked               |

### **Why does this matter?**

A coding agent can execute commands.

That makes it powerful, but it also introduces risk.

The engineer must control what the agent can do without approval.

### **Important nuance**

Allowing certain Git commands does not mean every possible Git operation is safe.

Destructive operations still require careful restrictions.

The transcript's principle is to permit approved routine actions while blocking or requiring approval for risky ones.

# **Part 9: Starting Claude Code in Plan Mode**

## **18. What Happens in Plan Mode?**

Claude Code reads:

- The brief.
- Project rules.
- Requirements.
- Existing code.

It can ask questions and propose an implementation plan.

It does not change code during this stage.

### **Why is this important?**

Because the agent should understand the current architecture before implementing new features.

Otherwise, it might duplicate existing functionality or make incompatible changes.

## **19. The Planning Prompt**

The transcript describes the prompt's essential instructions:

- Read the brief, rules, and code.
- Plan the work in small phases.
- Give each phase a check that can be run.
- Ask about unclear requirements.
- After approval, save the plan to a file.
- Implement phase one only.
- Stop.

### **Reusable prompt**

Read the approved feature brief, the project requirements, `CLAUDE.md`, and the existing codebase.

Before making any changes:

1. Identify gaps between the current code and the requested feature.
2. Ask me about anything important that is unclear. Do not guess.
3. Propose a small, phased implementation plan.
4. Give every phase a concrete completion check, such as a test command or build check.
5. Keep existing callers and security constraints working unless I explicitly approve a change.

Show me the plan for review.

After I approve it:

- Save the plan to a repository file.
- Implement **phase one only**.
- Run the allowed checks.
- Report the changed files, actual results, and any deviations from the plan.
- Stop and wait for my review before starting phase two.

This is an adapted prompt based on the transcript, not a verbatim copy.

# **Part 10: The Agent Finds an Important Gap**

## **20. There Are No Tests Yet**

During repository inspection, Claude Code discovers that the project has no tests.

This is important because the plan requires every phase to end with a check.

### **Why does this matter?**

Without tests, the team cannot confidently verify that a change works or that it has not broken existing behavior.

The coding agent must account for the missing test infrastructure in its plan.

This is a good example of why repository inspection should happen before implementation.

# **Part 11: The Three Clarifying Questions**

## **21. Question One — How Should the Session Remember the Order Number?**

Claude Code asks how conversation state should retain an order number.

### **Agent's recommendation**

Store the order number as a separate session field.

For example:

```
session = {
    "history": [],
    "order_number": "TEST-1001"
}
```

### **Human decision**

The engineer chooses a simpler design:

**Store only the conversation history.**

The order number is already present in the conversation.

### **Why choose this?**

It avoids maintaining duplicate state.

If the order number exists both in the message history and in a separate field, the two values could become inconsistent.

### **Trade-off**

The agent may need to recover the relevant order number from the conversation history.

This is simpler structurally but may require care when conversations contain multiple order numbers.

That trade-off is an engineering inference; the transcript only states the choice and its immediate rationale.

## **22. Question Two — Can **`agent.answer`** Accept History?**

The existing agent function does not accept conversation history.

Claude Code asks whether it may add an optional history argument.

### **Human decision**

Yes.

The important requirement is to preserve existing callers.

### **Example**

Before:

```
def answer(message):
    ...
```

After:

```
def answer(message, history=None):
    if history is None:
        history = []
    # Process the current message
    # using the optional conversation history.
    ...
```

### **Why make the argument optional?**

Because existing code can continue calling:

```
answer("Where is my order?")
```

New code can call:

```
answer(
    "Has it shipped?",
    history=previous_messages
)
```

This is an example of **backward compatibility**.

The new feature is introduced without requiring all existing callers to change immediately.

## **23. Question Three — How Much Widget Testing Is Required?**

Claude Code asks how thoroughly the widget should be checked.

### **Decision**

Use a build check only.

Do not add extra frontend testing tools in this phase.

### **Why?**

The brief is intentionally small.

Adding a complete frontend testing framework would increase scope.

### **What does a build check provide?**

It can detect problems such as:

- TypeScript compilation errors.
- Missing imports.
- Certain build configuration problems.

### **What does it not prove?**

It does not prove that every user interaction works correctly.

For example, a successful build does not guarantee that the "Talk to a person" button behaves correctly in a browser.

That distinction matters when deciding whether the application is ready for a later pilot.

# **Part 12: The Seven-Phase Plan**

## **24. Claude Code Proposes Seven Phases**

After clarifying the requirements, the agent proposes a seven-phase plan.

| Phase | Work                             | Purpose                                      |
| ----- | -------------------------------- | -------------------------------------------- |
| 1     | Create one Pydantic intent class | Eliminate the handwritten schema             |
| 2     | Add conversation history support | Enable session-aware agent calls             |
| 3     | Build the FastAPI chat service   | Connect HTTP requests to the agent           |
| 4     | Add live Claude tests            | Verify real model integration                |
| 5     | Build the first widget component | Begin the frontend                           |
| 6     | Complete the widget              | Finish the chat UI and required behavior     |
| 7     | Write a short run guide          | Explain how to start and use the application |

The transcript specifies that phases five and six build the widget, without providing an exact split between their tasks.

Every phase must end in a check.

## **25. Why Seven Phases Instead of One?**

Smaller phases are easier to:

- Understand.
- Review.
- Test.
- Debug.
- Commit.
- Roll back.

If the agent makes a mistake in phase three, the engineer does not need to discard the work completed and verified in phases one and two.

This follows the earlier AI SDLC lesson.

# **Part 13: Three Code Gaps Found Before Implementation**

## **26. Gap One — No History Parameter**

The existing agent loop cannot receive earlier conversation messages.

This must change to support multi-turn chat sessions.

## **27. Gap Two — Handwritten Intent Schema**

The intent schema is currently maintained as a Python dictionary.

That creates duplication risk.

The plan replaces it with a Pydantic class.

## **28. Gap Three — The Adapter Accepts Only the Dictionary**

The existing `llm.py` adapter is designed to receive the handwritten schema dictionary.

The plan initially expects that it may need modification.

However, the agent later discovers that the schema generated from Pydantic already fits the adapter.

This becomes an important example of avoiding unnecessary changes.

# **Part 14: Approving the Plan and Entering Auto Mode**

## **29. Human Approval Comes First**

The engineer reviews the plan and approves it.

Only after approval does Claude Code begin editing files.

### **Why is this important?**

The human retains control over:

- Architecture.
- Scope.
- Phase boundaries.
- Security constraints.
- Testing requirements.

The agent does not independently decide to implement the entire application.

## **30. What Is Auto Mode in This Lesson?**

After plan approval, Claude Code operates with permission to:

- Edit files.
- Run allowed commands.
- Execute approved tests.

It does not need to request permission for every routine action.

However, the settings still restrict dangerous operations.

### **Important distinction**

**Auto mode is not unrestricted mode.**

The agent can act automatically only within the configured permissions.

# **Part 15: Phase One — Replace the Handwritten Schema**

## **31. Phase One Objective**

The first phase is to make intent data use one Pydantic class.

The class must include the new language field.

The schema supplied to Claude must be derived from this class.

### **Expected result**

- One authoritative intent definition.
- No duplicated handwritten schema.
- Compatibility with the existing Claude adapter.
- Passing fast tests.

## **32. What Claude Code Reports**

The coding agent says:

- Phase one is complete.
- Fast tests pass.
- The intent is now one Pydantic class.
- The language field is included.
- Claude's schema is generated from the class.

It also reports a deviation from the plan.

The plan anticipated modifying `llm.py`.

But the generated schema already works with the existing adapter.

Therefore, `llm.py` remains unchanged.

### **Why is that good?**

The agent avoided an unnecessary code modification.

This reduces the risk of introducing regressions.

### **Important principle**

**A plan should guide implementation, but unnecessary changes should not be made merely because they appeared in the plan.**

Deviations should be explained and reviewed.

# **Part 16: Verify the Agent's Work Independently**

## **33. Do Not Trust the Completion Message Alone**

The transcript emphasizes:

> "A report is only text."

The agent saying "all tests pass" is not proof.

The engineer must run the tests and inspect the code.

## **34. First Verification — Run the Fast Tests**

The engineer runs the fast test suite.

The result is:

# **4 passed**

Fast tests after phase one

The transcript does not provide the exact test command or individual test names.

### **What does this prove?**

The four tests executed successfully.

It does not, by itself, prove that the entire future chat application works.

Only phase one has been implemented.

## **35. Second Verification — Inspect **`intent.py`

The engineer opens the actual file and reads the generated class.

The transcript identifies these fields:

- `summary`
- `intent`
- `order_number`
- `needs_human`
- `language`

The `intent` field has **six allowed values**.

The transcript does not enumerate those six values, so we should not invent them.

### **Why inspect the file?**

Because tests may miss architectural problems.

The engineer needs to verify that:

- The Pydantic class exists.
- It contains the expected fields.
- The intent values are constrained correctly.
- The language field is present.
- The schema comes from the class.

## **36. Illustrative Pydantic Implementation**

```
from pydantic import BaseModel, Field

class IntentResult(BaseModel):
    summary: str
    intent: str
    order_number: str | None = None
    needs_human: bool
    language: str = Field(
        description="Language of the customer message"
    )

claude_schema = IntentResult.model_json_schema()
```

This example demonstrates schema generation from a Pydantic class.

In the actual DemoStore implementation, `intent` should be constrained to the six allowed values established by the project.

The transcript does not list those values.

### **Why is **`model_json_schema()`** important?**

It generates a JSON Schema representation from the Pydantic class.

That allows the class to serve as the source of truth for Claude's structured output.

# **Part 17: Why **`llm.py`** Did Not Need to Change**

## **37. Understand the Interface**

The adapter already accepts a dictionary containing a schema.

Pydantic can generate a schema dictionary.

Therefore, the existing adapter can receive the generated schema without modification.

### **Before**

```
schema = {
    "type": "object",
    "properties": {
        # Handwritten fields
    }
}
```

### **After**

```
schema = IntentResult.model_json_schema()
```

The adapter still receives a dictionary.

### **Engineering lesson**

**If a new implementation satisfies an existing interface, the dependent component may not need to change.**

This is an example of interface compatibility.

It also reinforces the earlier prototyping lesson about stable tool contracts.

# **Part 18: Commit Phase One**

## **38. Create a Git Checkpoint**

After the tests pass and the code is reviewed, the engineer commits phase one.

### **Why?**

The commit records a known-good state.

If phase two introduces problems, the team can recover without losing the verified work from phase one.

### **Illustrative commands**

```

git status
git diff
python -m pytest -m "not live"

git add intent.py tests/
git commit -m "Define shared Pydantic intent schema"

```

These commands are illustrative.

The transcript only states that the engineer ran fast tests, inspected the class, and committed the phase.

# **Part 19: How the Remaining Phases Proceed**

## **39. One Fresh Session Per Phase**

The transcript says the remaining phases follow the same pattern:

1. Start a fresh coding-agent session.
2. Read the persistent plan and rules.
3. Implement the next phase.
4. Run tests.
5. Review the changes.
6. Commit.

### **Why fresh sessions?**

Long coding-agent conversations accumulate irrelevant context.

A fresh session helps keep the agent focused.

Because the plan and project rules live in repository files, the important information remains available.

# **Part 20: What This Lesson Teaches About Engineering Judgment**

## **40. Lesson A — A Short Brief Can Be Enough**

The brief does not need to contain every implementation detail.

It needs to state:

- What to build.
- What constraints apply.
- What must be tested.
- What should not be built yet.

The coding agent can inspect the code and ask clarifying questions.

## **41. Lesson B — Ask Before Guessing**

Claude Code asked three questions.

Each prevented a potentially incorrect assumption:

- How to store order numbers.
- Whether to change the agent function signature.
- How much frontend testing to add.

This is better than silently choosing an architecture.

## **42. Lesson C — Preserve Backward Compatibility**

Making the history argument optional allows existing callers of `agent.answer` to keep working.

This is especially useful when integrating new product code with previously developed components.

## **43. Lesson D — Avoid Duplicate State**

The engineer chooses to keep the order number in conversation history rather than storing a second copy in the session.

This reduces the amount of state that must be synchronized.

However, later phases should test how the assistant handles conversations containing multiple order numbers.

## **44. Lesson E — Avoid Unnecessary Code Changes**

The agent originally planned to modify `llm.py`.

It later discovered that no change was required.

It explained the difference.

This is preferable to changing working code without a clear reason.

## **45. Lesson F — Verification Must Be Independent**

The engineer did not simply accept Claude Code's report.

They:

- Ran the fast tests.
- Confirmed four passed.
- Opened `intent.py`.
- Reviewed the Pydantic class.
- Committed the accepted work.

This is the AI SDLC in practice.

# **Part 21: Transcript Facts Versus Engineering Extrapolations**

| Statement                                              | Classification                        |
| ------------------------------------------------------ | ------------------------------------- |
| The brief contains five parts                          | Explicit transcript fact              |
| The backend uses FastAPI                               | Explicit transcript fact              |
| The widget uses React and TypeScript                   | Explicit transcript fact              |
| The intent contains six allowed values                 | Explicit transcript fact              |
| The transcript lists those six values                  | Not stated                            |
| The plan contains seven phases                         | Explicit transcript fact              |
| Claude Code asks three clarifying questions            | Explicit transcript fact              |
| The agent finds no existing tests                      | Explicit transcript fact              |
| Four fast tests pass after phase one                   | Explicit transcript fact              |
| `llm.py` remains unchanged                             | Explicit transcript fact              |
| The exact FastAPI endpoint is `/chat`                  | Not stated                            |
| The exact session JSON structure is known              | Not stated                            |
| The exact Git commit message is known                  | Not stated                            |
| The frontend needs error handling and timeout behavior | Reasonable engineering recommendation |

This distinction matters in FDE interviews.

A strong engineer can explain what was observed, what was decided, and what remains an assumption.

# **Part 22: FDE Interview Questions**

## **46. Main Interview Question**

**Q: How would you use a coding agent to turn a working AI script into a customer-facing application?**

A strong answer:

> I would begin with a short approved brief describing the backend service, frontend widget, shared data contracts, safety constraints, and testing requirements.
>
> I would give the coding agent the brief, repository rules, and existing code, then use Plan Mode to have it inspect the project and propose small implementation phases.
>
> I would require the agent to ask clarifying questions rather than guess. For example, I would resolve how sessions store conversation history and how existing function signatures can be extended without breaking callers.
>
> Each phase would have a concrete verification step. After implementation, I would independently run tests, inspect the changed files, and review any deviations from the plan.
>
> I would commit each verified phase before starting the next one, using a fresh coding-agent session and the persistent repository plan.

## **47. Additional Questions and Answers**

**Q1. Why use FastAPI?**

To expose the existing assistant through a Python web service that can receive chat messages and return structured responses.

**Q2. What does the chat API return?**

The assistant reply, intent, and session ID.

**Q3. Why does the service need a session ID?**

To associate multiple messages with the same conversation.

**Q4. Why use one Pydantic class?**

To avoid maintaining inconsistent copies of the intent schema.

**Q5. Which three places use the shared intent class?**

Claude's structured output, the service response to the widget, and generated API documentation.

**Q6. What new field is added to intent?**

The language of the customer's message.

**Q7. Why must the human-support button always be visible?**

Because customers should not feel trapped in an automated support interaction.

**Q8. Why are live Claude tests marked separately?**

So routine fast tests can run without making real API calls.

**Q9. Why must all model calls use **`llm.py`**?**

To centralize model access and avoid duplicated integration code.

**Q10. What happens to sessions when the service restarts?**

They are lost because they are stored only in memory.

**Q11. How many questions did Claude Code ask before implementation?**

Three.

**Q12. What were the three questions about?**

Order-number storage, optional history support in `agent.answer`, and the scope of widget testing.

**Q13. How many implementation phases were planned?**

Seven.

**Q14. What did phase one implement?**

One Pydantic intent class with a language field and a generated schema for Claude.

**Q15. Why was **`llm.py`** not changed?**

Because the schema generated by Pydantic already matched the adapter's expected input format.

**Q16. How many fast tests passed?**

Four.

**Q17. What happened after verification?**

The engineer committed phase one to Git.

# **Part 23: Quick Knowledge Check**

**Test your understanding**

0/6 answered

1\. What are the three values returned by the chat service?

Reply, intent, session ID

Reply, password, API key

Intent, refund amount, invoice

2\. Why define intent with one Pydantic class?

To eliminate all testing

To keep Claude output, API responses, and documentation consistent

To replace the agent loop

3\. How does the engineer choose to remember the order number?

A separate session field

Conversation history only

A permanent database

4\. Why make history optional in agent.answer?

To preserve existing callers

To disable sessions

To avoid testing

5\. Why was llm.py unchanged after phase one?

The agent forgot it

The Pydantic-generated schema already fit its existing interface

The model API was removed

6\. What did the engineer verify before committing phase one?

Only the agent's written report

Four passing fast tests and the contents of intent.py

The completed React widget

# **Part 24: Complete Revision Checklist**

**Revision progress**

0 of 57 concepts reviewed

Reset

**Starting point**

Existing llm.py adapter works

Existing intent check works

Existing agent loop and order tools work

Goal is a website chat widget plus backend service

**Five-part brief**

FastAPI chat service

API returns reply, intent, and session ID

First reply discloses AI and human-support option

One shared Pydantic intent class

Intent includes language

React and TypeScript widget

Widget shows messages and typing indicator

Talk to a person button remains visible

Live Claude tests are opt-in

**Technical limits**

All Claude calls use llm.py

Sessions are stored only in memory

Restart loses sessions

CLAUDE.md documents tests and adapter rule

Tests and approved Git actions run automatically

Package installation asks permission

Secrets file is blocked

**Planning**

Start Claude Code in Plan Mode

Read brief, rules, requirements, and code

Ask about uncertainty before guessing

Write small phases with checks

Save approved plan to a file

Implement phase one only and stop

Agent discovers no tests exist

**Three questions**

Order number: history or separate field?

Engineer chooses history only

agent.answer may receive optional history

Optional argument preserves existing callers

Widget checks use build only

No extra frontend test tooling

**Seven phases**

Phase 1: Pydantic intent class

Phase 2: history support

Phase 3: FastAPI service

Phase 4: live tests

Phases 5–6: widget

Phase 7: run guide

**Code gaps**

Agent loop has no history argument

Intent schema is handwritten

Adapter accepts schema dictionary

Generated Pydantic schema already fits adapter

**Phase-one verification**

Claude Code reports completion

One Pydantic class contains five fields

Intent has six allowed values

Language field is new

llm.py remains unchanged

Engineer runs fast tests

Four tests pass

Engineer opens and reviews intent.py

Engineer commits phase one

**Process and takeaway**

Auto mode remains subject to permissions

Each later phase uses a fresh session

Tests and code review precede commit

Avoid unnecessary changes

Read the plan and questions before allowing code changes

# **Final Summary**

| Topic                 | Key takeaway                                    |
| --------------------- | ----------------------------------------------- |
| Goal                  | Convert working scripts into a chat application |
| Backend               | FastAPI                                         |
| Frontend              | React and TypeScript                            |
| API response          | Reply, intent, session ID                       |
| Intent schema         | One Pydantic class                              |
| New intent field      | Language                                        |
| Model access          | Always through `llm.py`                         |
| Sessions              | In-memory only                                  |
| Live tests            | Run only when explicitly requested              |
| Clarifying questions  | Three                                           |
| Implementation phases | Seven                                           |
| First phase           | Shared Pydantic intent schema                   |
| Fast tests            | Four passed                                     |
| Adapter change        | None required                                   |
| Final step            | Review and commit phase one                     |

## **The Most Important Takeaway**

The coding agent should not be given a vague instruction to build the entire product.

Give it a short approved brief and project rules. Let it inspect the existing code, identify gaps, and ask questions before implementation.

Then approve a phased plan, verify each phase independently, and save a Git checkpoint.

**The practical rule: Give the agent a short brief and your rules. Then read its plan and its questions before you let it write code.**

**Practice task:** Write a five-line brief for one feature of your own. Ask Claude Code to plan it in Plan Mode and count how many clarifying questions it asks before writing any code.

---

**Additional deep-dive material:** [Extended notes and extra interview practice questions](../../extras/11-chat-service-widget-additional-study-notes.md).
