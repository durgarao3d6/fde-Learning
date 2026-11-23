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

---

# Expanded Study Companion: All Planning and Phase-One Details

## Part A — Why we need product code

The team has functional small programs but no customer-facing experience.

- **`llm.py`** adapts API requests to Claude.
- **`intent.py`** checks the structured intent result.
- **`agent.py`** contains a multi-step agent loop using the order tools.

These demonstrate individual capabilities. DemoStore customers instead need a **chat window on the website** and a **service behind it** connected to those modules.

This is **product code**, so Claude Code may generate it; the engineer **steers and checks**. The inputs to the agent are the **five-point brief** from the preceding interview and **project rules**.

**Critical ordering:** Review Claude Code's **plan before any code is written**.

## Part B — The five requirements, fully unpacked

### Requirement 1 — FastAPI chat service
The application uses **FastAPI**, a Python library/framework for web services.

A message comes in. A response contains:
1. The assistant's **reply**.
2. The classified **intent**.
3. A **session ID**.

Why session ID? It associates multiple HTTP requests with the correct conversation, allowing the backend to keep context.

The lesson does not provide a literal route name, so `POST /chat` below is **illustrative**.

```text
React widget --message, optional session ID--> FastAPI service
React widget <--reply, intent, session ID------- FastAPI service
```

A session is a **record of one conversation**. It is not an authentication credential. Protected order data still requires a separately verified identity.

### Requirement 2 — The AI notice
The **first reply** states that:
- The customer is talking to **an AI**.
- They may ask to speak to a **person at any time**.

The lesson ties disclosure to the **EU AI Act** and the person option to **DemoStore's own rule**. These are distinct rationales.

**Potential implementation technique (illustrative):** Generate the notice deterministically when a session is created, so it does not rely solely on the model remembering disclosure instructions.

### Requirement 3 — One Pydantic intent schema
**Pydantic** is a Python library that defines and validates data shapes with Python classes.

The team needs **one class** used in **three different places**:
1. Claude's structured response.
2. The backend's return data to the React widget.
3. The FastAPI-generated documentation page.

The additional field is **language**.

Why not two copies? Suppose the model schema includes `language` but a separate response validator does not. The model may send an answer that the service rejects. The speaker warns this inconsistency might make **every chat fail**.

The correct solution is to derive all three consumers from one authoritative definition.

### Requirement 4 — Chat widget
Frontend technology:
- **React**, for web UI components.
- **TypeScript**, JavaScript with types.

Required display:
- Customer and assistant messages.
- A **typing indicator** while the reply is coming.
- An **always-visible “Talk to a person” button**.

Why always visible? Customers often worry that a chatbot is a barrier preventing them from reaching a human. Hiding the option undermines trust.

**Clarify stage:** The button does **not** implement a real ServiceNow case handoff in this first build; that comes later.

### Requirement 5 — Actual Claude integration tests
The team wants tests that make **real Claude API calls**, not solely mocks.

They are explicitly labeled **live**, and the fast suite **skips** them. They should run **only when explicitly requested**, because real model calls consume money and time and may be less deterministic.

Unit tests and real integration tests serve different needs:
| Test | What it verifies | Run cadence |
|---|---|---|
| Fast unit tests | Local code behavior and schema checks | Frequent |
| Live Claude integration | Real model API responds in expected shape | Explicit/on demand |
| Widget build check | Frontend compiles/type-checks | After frontend changes |
| Browser interaction tests | Actual click and message behavior | Not included in first widget plan |

The exact test framework and commands are not prescribed by this transcript.

## Part C — Two non-negotiable limits

**Limit 1: every Claude call passes through `llm.py`.**

This avoids multiple independently maintained API adapters and centralizes provider configuration.

**Limit 2: sessions live only within the running service.**

No database; a **restart loses sessions**. Do not represent that as production-ready persistence.

## Part D — The project rule files

Root `CLAUDE.md` states:
- How to run **fast tests**.
- The model adapter rule: every model call through `llm.py`.

The **settings file** is an enforced permissions layer:
- Tests may run without repeated approval.
- Approved Git commands may run.
- Installing a new package asks the engineer first.
- The file containing secrets is blocked.

This echoes the earlier coding-agent lesson: **instructions are guidance, settings are controls**.

## Part E — Plan Mode, before touching files

The engineer starts Claude Code in **Plan Mode**. It can read and ask but **changes nothing**.

The planning request says:
- Read the brief.
- Read rules and requirements.
- Inspect current code.
- Propose **small implementation phases**.
- End every phase with a **check the human can run**.
- Ask about unclear details.
- When approved, save the plan to a file.
- Implement **phase one only**, then **stop**.

A good plan is not simply a list of features; each step should create independently reviewable evidence.

The model discovers **there are no tests in the repository yet**. This matters because the first phase must establish or use tests, not claim unseen coverage.

## Part F — Question 1: should sessions duplicate the order number?

The agent wants to know how an order number stays accessible across turns.

**Agent recommendation:** Store the order number separately in the session.

**Human decision:** Keep **conversation history only**.

Reason: the order number is already in the customer's conversation. A duplicate field introduces state synchronization work.

Illustrative contrast:
```python
# Rejected extra state idea
session = {"history": [...], "order_number": "ORDER-123"}

# Approved simpler structural choice
session = {"history": [...]}
```

The *actual transcript* does not provide the session representation or imply that every multi-order situation is already solved. It simply records the design choice.

**Engineering trade-off:** Later testing must ensure the assistant does not confuse an old order number with a newer one in a long conversation.

## Part G — Question 2: backwards-compatible history parameter

The agent detects `agent.answer` **currently accepts no conversation history**.

It asks whether it can add an **optional** history argument.

The engineer answers **yes**, because existing calls should still work.

Illustrative:
```python
def answer(message, history=None):
    if history is None:
        history = []
    # existing agent flow continues here
```

Existing usage remains valid:
```python
answer("Where is my order?")
```

New usage can pass context:
```python
answer("Any update?", history=previous_messages)
```

**Why optional?** It introduces capability while preserving backward compatibility.

## Part H — Question 3: extent of widget checks

The model asks how much frontend test machinery it may add.

Human answer: **A build check only**, with **no extra testing tools**.

This is scope control, not an assertion that a build check proves every UI behavior.

### What a frontend build can catch
- Syntax errors.
- TypeScript type errors.
- Missing imports and some configuration problems.

### What it cannot reliably prove
- Human-support button always visible under every UI state.
- Typing indicator appears and disappears at the right time.
- Network failures produce understandable messages.
- Accessibility and real browser interaction.

Those deeper checks may be appropriate in later stages.

## Part I — Three integration gaps discovered from the code

Before writing anything, the agent reports:
1. The **agent loop accepts no history**.
2. Intent schema is **a dictionary written by hand**.
3. The **adapter accepts the schema dictionary**.

These are different sorts of gaps:
- **History** is a functional capability gap.
- **Handwritten dictionary** is a maintainability/schema consistency gap.
- **Adapter's interface** is a compatibility question: will it need changes?

This information helps shape the seven-phase plan.

## Part J — The seven phases in exact sequence

| Number | Phase | Transcript detail | Stage of work shown |
|---|---|---|---|
| 1 | Shared intent schema | Make intent one Pydantic class | **Completed** |
| 2 | Conversation history | Pass history to agent | Planned |
| 3 | Chat service | Build FastAPI service | Planned |
| 4 | Live tests | Add tests that really call Claude | Planned |
| 5 | Widget | Begin React widget implementation | Planned |
| 6 | Widget | Continue/complete widget | Planned |
| 7 | Running guide | Short instructions to run whole product | Planned |

The lesson explicitly says **each phase ends in a runnable check**. It **does not** provide exact acceptance commands for phases two through seven; invented commands should be labeled as examples.

## Part K — Plan approval and Auto Mode

The engineer reads and **approves** the proposed plan.

Only now does Claude Code leave the read-only planning step and enter **Auto Mode**. It can **edit files** and run **allowed commands** without asking for permission at every routine step.

The user has not granted unlimited access: package installs still ask, and secrets remain blocked.

**Instruction:** **Implement phase one only. Stop after it.** Auto Mode is constrained by phase scope as well as tool permissions.

## Part L — Reported phase-one result

Claude Code reports:
- Phase one is done.
- Fast tests pass.
- The intent is a **single Pydantic class**.
- The new **language** field exists.
- Schema supplied to Claude is **generated from the class**.

Then it explicitly explains what **it did not do**.

### The unexpected adapter decision
The original plan said the `llm.py` adapter might need changes. But `IntentResult.model_json_schema()` (illustrative Pydantic mechanism) yields a dictionary in the format the adapter already accepts.

Therefore **`llm.py` is left unchanged**.

This is a good example of implementing the *objective* instead of blindly executing unnecessary modifications from an initial plan.

## Part M — The report is text, not proof

The narrator's exact conceptual point is:

> A report is only text, so we check for ourselves.

The human does three substantive verification actions.

### 1. Run fast tests independently
**Result: four tests pass.**

This confirms four executed test cases, not that the complete chat platform is ready.

### 2. Read `intent.py`
The Pydantic class contains exactly these described fields:
- **`summary`**
- **`intent`**, constrained to **six allowed values**
- **`order_number`**
- **`needs_human`**
- **`language`**, newly added

**Do not fabricate the six values.** The provided transcript says there are six but does not list them.

### 3. Confirm schema generation
The JSON Schema Claude must follow comes **from the one Pydantic class**, avoiding a separately maintained duplicate schema.

### 4. Commit phase one
After the checks, the engineer makes a **Git commit**. The commit marks a known-good checkpoint.

The speaker says the remaining phases use **one fresh session each, followed by tests and a commit**.

## Part N — A careful illustrative Python schema

```python
from pydantic import BaseModel, Field

class IntentResult(BaseModel):
    summary: str
    intent: str  # In the actual lesson, exactly six allowed choices.
    order_number: str | None = None
    needs_human: bool
    language: str = Field(description="Language of the message")

schema_for_claude = IntentResult.model_json_schema()
```

**Why is this labeled illustrative?** The lesson does not provide the final source text, exact constrained-value type or validators. The principles are one class, five fields, six intent choices and generated schema.

### An example of schema drift
Suppose handwritten Claude JSON requires `language`, but the backend response model omits it. One part accepts fields that another rejects. This could break request processing on every chat.

By reusing one Pydantic definition, a schema update can be propagated deliberately through model output, service response and generated docs.

## Part O — A careful illustrative FastAPI service shape

The API itself is **not implemented in the lesson yet**; it is phase three. To understand the requested contract, imagine:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class ChatRequest(BaseModel):
    message: str
    session_id: str | None = None

class ChatResponse(BaseModel):
    reply: str
    intent: IntentResult
    session_id: str

# Actual implementation must wire sessions and authorized agent calls.
```

This example is educational, **not a claim that Claude Code created this code during phase one**.

## Part P — A careful illustrative session flow

```text
Customer opens chat
  ↓
Widget sends first message (no session ID yet)
  ↓
FastAPI creates a new in-memory session
  ↓
Assistant sends AI notice / processes the message
  ↓
FastAPI returns response + session ID
  ↓
Widget stores that ID locally for subsequent requests
  ↓
Next message carries the ID
  ↓
FastAPI loads prior history
  ↓
Agent processes the message with optional history
```

The route name, exact identity implementation and UI storage details are **design examples**. The transcript's requirements are the **session ID**, **in-memory sessions**, and **optional history parameter**.

## Part Q — Human access and nonfunctional product requirements

The "Talk to a person" button is **always visible** because of a **specific customer fear** that bots block access to humans. A permanently accessible path is part of trustworthy UX.

The **typing indicator** addresses a separate problem: LLM replies can take time, and users need to see that a reply is pending.

The **AI notice** addresses transparency.

These are three separate product requirements:
1. Know **who/what** you're talking to.
2. Know that the app **is responding**.
3. Know how to **request a person**.

None of those requirements should be treated as optional visual polish.

## Part R — What has been delivered versus what is merely planned?

| Capability | At the end of the supplied transcript |
|---|---|
| Existing Claude adapter | Already working |
| Existing order-tool agent loop | Already working |
| One shared Pydantic intent class | **Phase one completed** |
| New language field | **Completed** |
| Generated Claude schema | **Completed** |
| Fast tests | **Four passed** |
| History-aware agent function | Next phase |
| FastAPI chat backend | Phase three |
| Live Claude tests | Phase four |
| React/TypeScript widget | Phases five and six |
| Setup and run guide | Phase seven |
| Real customer-facing handoff | Still deferred from earlier brief |
| One-time-code sender | Still deferred from earlier brief |

The full chat site should not be described as built during the lesson. The exercise shows **planning and first-phase implementation**, not seven completed phases.

## Part S — AI SDLC connections

### Why inspect code before planning?
Avoid breaking existing adapters and recognize where new requirements fit.

### Why ask questions first?
The session strategy and compatibility policy are legitimate design decisions, not things the agent should silently invent.

### Why insist on a runnable check per phase?
Completion should be observable.

### Why check the code after the agent reports success?
A language-model assertion of success is not evidence. Tests and diffs are evidence.

### Why commit?
A small accepted checkpoint lowers the cost of recovery.

### Why fresh sessions?
Short contexts reduce irrelevant earlier attempts; plans and rules remain in the repository.

### Why didn't the adapter change?
Its existing interface already met the new requirement: **working code can remain untouched**.

## Part T — Interview question bank with detailed answers

**Q1. What are the existing programs?** The Claude adapter, intent extraction/validation and agent loop with order tools.

**Q2. What is the business deliverable?** A website chat window and backend service talking to the existing assistant.

**Q3. Who writes the product code?** The coding agent, subject to human planning, steering and verification.

**Q4. What does the FastAPI response include?** Reply, intent and session ID.

**Q5. Why use a session ID?** To associate turns with one conversation.

**Q6. Why one Pydantic class?** To keep model output, widget API response and API docs consistent.

**Q7. Which new field was added?** Language.

**Q8. What happens if duplicated schemas drift?** The backend may reject otherwise valid responses, potentially breaking chats.

**Q9. What technologies implement the widget?** React and TypeScript.

**Q10. What visual feedback is required?** A typing indicator.

**Q11. Why always show a human-support button?** Customers fear automation prevents access to humans.

**Q12. Are live model tests part of the fast suite?** No, marked live and run explicitly.

**Q13. Where must every Claude call go?** Through `llm.py`.

**Q14. Where is conversation state held?** In the running service's memory.

**Q15. What happens on restart?** Session history is lost.

**Q16. What belongs in `CLAUDE.md`?** Fast test instructions and adapter rule.

**Q17. What may run automatically?** Approved tests and Git commands.

**Q18. What requires permission?** Installing a package.

**Q19. What is forbidden?** Reading the secrets file.

**Q20. What is Plan Mode's guarantee in this lesson?** Read/ask with no code changes.

**Q21. What did the agent discover about tests?** None existed yet.

**Q22. What were the three clarifying questions?** Session order-number representation, optional history argument and widget testing depth.

**Q23. Which session representation was chosen?** History only.

**Q24. Why add optional history?** Preserve existing callers.

**Q25. How far did widget testing go?** Build check only, without extra test tools.

**Q26. What gaps were identified?** No agent history, handwritten intent schema, adapter accepts dictionary.

**Q27. How many phases?** Seven.

**Q28. What does phase one do?** One Pydantic intent class with generated schema and language.

**Q29. What does phase two do?** Add conversation history.

**Q30. What do phases three and four do?** Build FastAPI service, then live Claude tests.

**Q31. What do phases five, six and seven do?** Widget, widget and short run guide.

**Q32. When can Auto Mode write code?** After the plan is approved.

**Q33. Why didn't `llm.py` change?** Generated schema already fit its interface.

**Q34. What did the developer independently run?** Fast tests; four passed.

**Q35. Which file was inspected?** `intent.py`.

**Q36. What fields are in the class?** Summary, intent, order number, human-required flag, language.

**Q37. How many allowed intent values?** Six, not enumerated in the transcript.

**Q38. What happened after review?** A Git commit of phase one.

**Q39. What happens for later phases?** Fresh agent session, tests, review and commit.

**Q40. What is the practical rule?** Short brief plus rules; inspect questions and plan before writing code.

## Part U — Complete revision checklist

- [ ] I know the **three existing modules**.
- [ ] I can explain **product code** versus individual scripts.
- [ ] I know all **five brief parts** in order.
- [ ] I know the three **API response items**.
- [ ] I can explain **session ID ≠ identity verification**.
- [ ] I remember the first message **AI disclosure**.
- [ ] I can separate **EU transparency** from **DemoStore's human-access policy**.
- [ ] I know the **three consumers of the Pydantic model**.
- [ ] I know `language` is newly added.
- [ ] I can explain duplicate-schema drift.
- [ ] I know **React** and **TypeScript**.
- [ ] I know the **typing indicator**.
- [ ] I know the human button **never hides**.
- [ ] I know **live tests are opt-in**.
- [ ] I know all model calls go through **`llm.py`**.
- [ ] I know sessions are **in-memory only**.
- [ ] I know what **`CLAUDE.md`** contains.
- [ ] I can recall settings: **allow tests/Git, ask package installs, deny secrets**.
- [ ] I know **Plan Mode changes nothing**.
- [ ] I remember the repository **had no tests**.
- [ ] I can state the **three clarifying questions and choices**.
- [ ] I can list **all seven phases**.
- [ ] I remember **three code gaps**.
- [ ] I know only **phase one** was implemented.
- [ ] I can name all **five class fields**.
- [ ] I remember **six intent values, names not given**.
- [ ] I can explain **why `llm.py` stayed unchanged**.
- [ ] I remember **four passing fast tests**.
- [ ] I know the engineer **opened `intent.py`** themselves.
- [ ] I know the engineer **committed phase one**.
- [ ] I know later phases use **fresh sessions, tests and commits**.

**Course exercise:** Write a five-line brief for one feature of your own. Let Claude Code plan it in Plan Mode and count the questions it asks before it writes any code.
