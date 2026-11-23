# Planning the FastAPI Service and React Chat Widget

## Learning objective
Turn working AI scripts into product code through an approved phased plan and independently verified implementation.

## Existing components
- `llm.py`: adapter that calls Claude.
- `intent.py`: structured intent check, with a hand-written dictionary schema.
- `agent.py`: agent loop with order tools, but **no history parameter**.

## Five-part brief
### 1. FastAPI chat service
Accept a message and return a **reply, intent and session ID**. The session ID associates requests with one conversation.

### 2. AI disclosure
First reply says the person is talking to an AI and can request a human. The EU AI Act transparency rationale and DemoStore's human-access rule are distinct.

### 3. One Pydantic intent class
The same class shapes Claude's output, the service's widget response and the automatically generated API documentation. Add **language** once to this definition, rather than maintaining duplicate schemas.

### 4. React + TypeScript widget
Show messages, a typing indicator and a permanently visible **“Talk to a person”** control. Customers should not feel trapped in automation; this first version still lacks a real ServiceNow transfer.

### 5. Opt-in live Claude tests
Tests marked **live** actually call Claude; routine fast tests skip them unless explicitly requested.

## Technical limits
- All Claude API calls pass through `llm.py`.
- Session data stays in the running process's memory; restart loses it.
- `CLAUDE.md` specifies test commands and adapter usage.
- Settings allow approved tests and Git commands, ask before package installation and deny reading secrets.

## Prompt for planning
Use Plan Mode: “Read the brief, requirements, rules and code. Propose small phases with a runnable check at the end of each. Ask what is unclear. Once approved, save the plan, implement **phase one only**, and stop.”

## Three clarifying questions
1. **How remember order numbers?** Agent recommends an explicit field. Engineer chooses **conversation history only**, avoiding duplicate state.
2. **Can `agent.answer` receive history?** Yes, as an **optional argument**, keeping existing callers working.
3. **How much widget testing?** A **build check only**, no additional frontend test tooling in this phase.

The agent also discovers **there are no tests yet**.

## Seven-phase implementation plan
| Phase | Scope | Example check |
|---|---|---|
| 1 | One Pydantic intent model | Fast tests |
| 2 | Agent conversation history | Existing and new caller tests |
| 3 | FastAPI service and sessions | API tests |
| 4 | Opt-in live tests | Explicit live execution |
| 5 | Widget foundation | Frontend build |
| 6 | Finish widget behavior | Frontend build |
| 7 | Run instructions | Follow guide and inspect startup |

The exact phase-five/six split is not specified in the video; the checks in this table beyond phase one are illustrative.

## Phase one: actual outcome
Claude Code reports a single Pydantic class with:
- `summary`
- `intent` (**six allowed values**, which are not individually named in the transcript)
- `order_number`
- `needs_human`
- `language`

Claude's schema is generated **from the class**. The original plan anticipated changing `llm.py`, but the generated dictionary already fits the existing adapter. **No adapter edit was necessary.**

## Verify instead of trusting
The human independently runs fast tests: **four pass**. They open and inspect `intent.py`, confirm the fields and generated schema, then **commit phase one**.

Subsequent phases use a fresh conversation, manual checks and a commit each time.

## Review questions
1. Why return a session ID?
2. Why is one Pydantic class preferable to two dictionaries?
3. Why keep history optional?
4. Why did `llm.py` not need an edit?
5. What precisely did the engineer verify before committing?

## Remember
**Give the agent a short brief and your rules. Read its questions and plan before code changes.**

---

# Deep dive: from approved brief to first verified code phase

## 1. Starting position
The team has functional but separate small modules:
- `llm.py`, which adapts requests to Claude.
- `intent.py`, which defines and checks structured intent.
- `agent.py`, which runs the order-tool agent loop.

Customers ultimately need a **chat interface on the DemoStore website** and a **service** behind the widget. The project now transitions from exercises to **product code**.

The coding agent will implement it, but the human steers, reviews and accepts changes. The engineer provides both a **short brief** and **repository rules**.

## 2. The full five-part brief, line by line

### Part one: FastAPI chat backend
**FastAPI** is a Python framework for building HTTP APIs. The service accepts an incoming customer message and returns:
1. **Reply** — what the assistant says.
2. **Intent** — structured representation of the user's request.
3. **Session ID** — reference identifying this chat's history.

Why the session ID? Separate HTTP requests are otherwise independent. A session key lets the service find messages belonging to one conversation.

**Illustrative API contract** (the transcript does not name an exact endpoint):
```json
// POST /chat (illustrative)
{
  "session_id": "demo-session-1",
  "message": "Has my order shipped?"
}
```
```json
{
  "session_id": "demo-session-1",
  "reply": "Order lookup is available after identity verification.",
  "intent": {
    "summary": "Asks about shipment status",
    "intent": "order_status",
    "order_number": null,
    "needs_human": false,
    "language": "en"
  }
}
```

**Do not confuse:** Session ID identifies a conversation; a verified customer field establishes identity. A session ID is not itself authorization to view protected order data.

### Part two: AI notice
The **first reply** tells the customer they are speaking to AI and may request a person at any time. The lesson cites transparency under the EU AI Act; the route to a person is a separate DemoStore policy.

The UI should represent the current handoff honestly: the earlier interview explicitly deferred real ServiceNow case creation.

### Part three: one Pydantic intent class
**Pydantic** defines typed, validated Python data. Instead of a handwritten schema in more than one place, maintain **one class**.

Consumers:
- Claude's structured output format.
- Backend response sent to React widget.
- FastAPI's self-generated API documentation.

**The new field is `language`.** Adding the field once should update all three schema consumers if the wiring is correct.

Without a single source of truth, the model might return a field the response validator does not recognize, causing every affected chat request to fail.

### Part four: React + TypeScript widget
**React** provides reusable interface components. **TypeScript** supplies static types for JavaScript.

Minimum behavior:
- Show customer and assistant messages.
- Show a **typing indicator** while the answer is pending.
- Keep **“Talk to a person” always visible**.

Reason for persistent access: customers fear bots will trap them and make a person hard to reach. The button must not disappear when a session gets complicated.

This is a UX requirement. It does **not** imply the later actual human-handoff integration already exists.

### Part five: tests that really call Claude
Add model integration tests marked **live**. Routine fast tests **skip live tests**; run them only deliberately.

Why:
- Real API calls cost money.
- Network availability may vary.
- Model responses can be nondeterministic.
- The rapid unit suite should remain quick.

Illustrative:
```python
import pytest

@pytest.mark.live
def test_claude_response_schema():
    # Call real Claude only when the live test group is selected.
    ...
```

### Two additional limits
1. **All Claude API calls go through `llm.py`.**
2. **Sessions live in memory only.** Restarting the service loses conversation history.

These are intentional scope decisions for an internal early version.

## 3. Rules alongside the brief
The root `CLAUDE.md` states:
- How to run fast tests.
- Every model request must use `llm.py`.

The permission settings enforce:
- **Allow:** approved tests and Git commands.
- **Ask first:** installing a package.
- **Deny:** reading the secrets file.

Do not give the coding agent unrestricted permission just to make it work faster.

## 4. Plan Mode prompt
The engineer starts Claude Code in **Plan Mode**, where it can inspect and ask without editing.

A condensed request:
```text
Read the brief, repository requirements, CLAUDE.md, and existing code.
Propose small implementation phases with a runnable check after each.
Ask about anything unclear; do not guess.
After I approve, save the plan to a file.
Implement phase one only, then stop.
```

The agent also finds **there are no tests yet**. That discovery matters because every planned phase needs an independent check.

## 5. Three clarifying questions: detailed decisions

### Question 1 — How to remember an order number?
The agent recommends storing it in its own session field.

Engineer chooses **history only** because the order number already appears in the conversation.

**Benefit:** Less duplicated session state.
**Potential trade-off (engineering inference):** With several order numbers in one conversation, history extraction must handle ambiguity.

### Question 2 — May `agent.answer` accept history?
The current function accepts **no history**. The agent requests permission to add an **optional history argument**.

Engineer answers **yes**.

Why optional? Existing code that calls the function without history should continue working.

```python
# Conceptual API shape
def answer(message, history=None):
    if history is None:
        history = []
    ...
```

This demonstrates a backward-compatible extension rather than a breaking redesign.

### Question 3 — How deep should widget checks go?
The agent asks whether to set up more testing tools.

Decision: **Build check only, no extra frontend test framework.** That matches the brief's narrow scope.

A build check detects many compilation and type errors, but does **not** replace browser interaction tests before production.

## 6. The agent discovers three integration gaps
Before writing code, its repository inspection finds:
1. `agent.answer` does not take history.
2. The intent output schema is a **handwritten dictionary**.
3. The model adapter currently accepts **that dictionary**.

This is why inspecting existing code is essential: the brief defines desired behavior, while the code defines integration constraints.

## 7. Seven approved phases
| Phase | Work | What successful completion should show |
|---|---|---|
| **1** | One Pydantic intent class | Schema generated from class; fast tests |
| **2** | Support conversation history | Existing callers still work; new history behavior tested |
| **3** | FastAPI chat service | Request/response/session behavior validated |
| **4** | Live Claude integration tests | Explicit opt-in tests work |
| **5** | Begin React/TypeScript widget | Frontend builds |
| **6** | Complete widget behavior | Frontend builds and required controls exist |
| **7** | Short run guide | Team can follow setup instructions |

Only **phase one** is implemented during this lesson. Later-phase test suggestions in this table are explanatory examples, not exact commands from the transcript.

## 8. Plan approved; Auto Mode begins
After the engineer reads and approves the plan, Claude Code enters **Auto Mode** and edits files and runs **allowed** commands without asking each time.

Auto Mode is not the same as giving blanket permissions: the package-install and secret-file restrictions still apply.

The explicit execution instruction is **phase one only**. The agent must stop, even if it could continue writing phases two through seven.

## 9. Phase-one result: one schema definition
The coding agent reports:
- Pydantic class created.
- New `language` field included.
- Schema sent to Claude is **generated from the class**.
- Fast tests pass.

### Exact fields inspected by the engineer
| Field | Role |
|---|---|
| `summary` | Short textual description of the customer message |
| `intent` | Categorized request, with **six allowed values** |
| `order_number` | Order identifier when present |
| `needs_human` | Whether human support is required |
| `language` | Language of the message |

The transcript **does not list all six intent values**. We should not invent them and claim they were in the lesson.

### Illustrative Pydantic sketch
```python
from pydantic import BaseModel, Field

class IntentResult(BaseModel):
    summary: str
    intent: str  # Actual code constrains this to six allowed values.
    order_number: str | None = None
    needs_human: bool
    language: str = Field(description="Language of the customer message")

schema_for_claude = IntentResult.model_json_schema()
```

This is an **illustration**, not the exact file the agent generated.

## 10. An especially valuable deviation
The plan expected the adapter `llm.py` to change. But the generated schema is a dictionary and **already fits the adapter's existing contract**.

The coding agent **does not change `llm.py`**, and explains why.

This is an example of good restraint:
- Plans are hypotheses about needed implementation changes.
- Inspection can reveal a simpler path.
- Do not edit working code just to match a checklist.
- Report meaningful deviations so the reviewer can approve them.

## 11. Independent verification: “a report is only text”
The agent says phase one is complete and tests pass. The engineer does **not** stop there.

They:
1. Run the fast tests themselves.
2. See **four tests pass**.
3. Open **`intent.py`**.
4. Inspect the five fields and the six-value intent constraint.
5. Confirm schema generation comes from the class.
6. Confirm there was no unnecessary adapter change.
7. **Commit phase one**.

A green test suite is evidence, but code review is still necessary. A test can miss architectural or security problems.

## 12. Why commit immediately?
A commit captures a reviewed checkpoint. If phase two changes history handling incorrectly, the team can revert without losing validated phase-one work.

The following phases repeat:
```text
Fresh coding-agent session
  → read persistent plan and rules
  → implement one phase
  → independent tests
  → diff/code review
  → Git commit
  → next fresh session
```

This follows the earlier AI SDLC rule to clear stale context and preserve durable repository state.

## 13. Compare claimed progress with actual progress
| Topic | At this lesson's end |
|---|---|
| Pydantic intent schema | **Implemented and independently checked** |
| Fast tests | **Four passed** |
| `llm.py` modification | **Not needed** |
| Agent history support | Planned for phase two |
| FastAPI backend | Planned for phase three |
| Real-Claude tests | Planned for phase four |
| React widget | Planned for phases five and six |
| Setup guide | Planned for phase seven |

**Do not call the complete product finished.** Only the first phase has been shown.

## 14. Common mistakes and safeguards
**Mistake: one giant coding task.** Use seven small phases.

**Mistake: ignoring existing interfaces.** Inspect `agent.answer`, intent schema and `llm.py` before editing.

**Mistake: forcing redundant session fields.** Engineer explicitly chose history only.

**Mistake: breaking old callers.** Add history as an optional parameter.

**Mistake: bolting on unnecessary test tooling.** Frontend scope was deliberately a build check.

**Mistake: treating agent report as proof.** Run tests and inspect `intent.py`.

**Mistake: changing `llm.py` just because it was in the plan.** Reuse its current dictionary interface when compatible.

**Mistake: letting Auto Mode run seven phases without checks.** Approve one phase, verify and commit it, then continue.

## 15. Interview-ready answer
**Question:** “How would you instruct a coding agent to build a FastAPI backend and React chat widget around existing LLM scripts?”

**Answer:** “I'd give it a five-point brief and repository rules, use Plan Mode to inspect the code and clarify session state, function compatibility and frontend test scope, then approve a phased plan with runnable checks. I'd permit only the first phase, independently execute its tests, inspect the shared Pydantic intent model and generated schema, and commit it before moving on. In DemoStore the first phase passed four fast tests, and the adapter did not need modification because it already accepted the generated schema dictionary.”

### Revision checklist
- [ ] Five brief parts: **FastAPI / AI notice / Pydantic / React+TS / live tests**.
- [ ] API returns **reply, intent, session ID**.
- [ ] **Typing indicator** and **always-visible person button**.
- [ ] All model calls use **`llm.py`**.
- [ ] **In-memory sessions**, restart loses them.
- [ ] Settings: **tests/Git allowed; packages ask; secrets denied**.
- [ ] Plan Mode: **read and ask, no code edits**.
- [ ] **No tests initially**.
- [ ] Three questions: **history-only order number**, **optional history**, **widget build check**.
- [ ] Seven phases in correct order.
- [ ] Phase one fields: **summary, intent (six values), order_number, needs_human, language**.
- [ ] Schema from **one Pydantic class**.
- [ ] `llm.py` **unchanged** due to compatibility.
- [ ] Independent **four passing tests**, file inspection, Git commit.
- [ ] Fresh agent session for each next phase.
