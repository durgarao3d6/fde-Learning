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
