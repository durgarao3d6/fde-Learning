# Coding Agents and the AI Software Development Life Cycle

## Learning objective
Use a coding agent for speed while retaining engineering ownership of every accepted change.

## What is a coding agent?
A model working in a codebase through tools that read files, execute commands and modify code. The lesson uses **Claude Code**; similar concepts appear in Codex, Gemini CLI, GitHub Copilot and Cursor.

It resembles our earlier agent loop: the model requests a tool; the application executes it and returns the result.

## The three habits
1. **Plan in phases.** Do not paste an entire specification and say “build the assistant.” A large uncontrolled rewrite can change 30 files with little explanation.
2. **Give explicit rules.** Keep persistent project guidance in root-level `CLAUDE.md`; use permission settings for real limits.
3. **Demand proof and checkpoints.** Run tests yourself, inspect every changed line, and commit verified work after each phase.

## Plan mode
Claude Code's Plan Mode is used to **read and ask without editing**. Review the plan like a colleague's proposed implementation. Each phase must have tasks and an observable pass condition.

DemoStore phases include identity verification, case creation with summaries, and evaluating Marco's 40 labeled chats.

Save the plan **in the repository**, not only in a conversation.

## CLAUDE.md
Document how to run tests, that the model name lives in one constant, and these business rules:
- A refund is only a draft.
- No writing to Ledgerline tables beyond those approved by the owner.

Project instructions **guide** the agent; they cannot enforce permissions.

## Three permission groups
- **Allow:** ordinary file reads and approved tests.
- **Ask:** installing packages or pushing to GitHub.
- **Deny:** accessing secrets files.

Do not choose “allow everything” merely to avoid approval prompts.

## Clear context between phases
Use Claude Code's `/clear` after a completed phase. Plans and rules survive in files; stale conversation noise does not need to accumulate.

## Independent proof
A message claiming “all tests pass” is **not proof**. Run the command and read its output. Review modifications to tests especially carefully: an agent can weaken a test instead of fixing code.

Commit after successful review. When a later phase rewrites too much, recover from the last checkpoint and refine the plan.

## Human accountability
**Ravi Menon**, DemoStore's Director of E-commerce Engineering, owns the underlying systems. His team reviews pull requests. **“The agent wrote it” is not an acceptable technical explanation.**

## Review questions
1. Why keep Plan Mode separate from editing?
2. Which limits belong in settings rather than prompt instructions?
3. Why scrutinize changed tests?
4. How do small commits reduce failure cost?

## Remember
**Plan in phases, prove each one, and commit before the next.**

---

# Deep dive: responsible coding-agent workflow

## 1. Why the team switches from hand-written scripts
Earlier lessons involved short files showing one Claude call, structured intent extraction and an agent loop. A customer-facing product involves many more moving parts: API service, session history, UI, tests, documentation, integrations and release processes.

Rather than typing every line by hand, the FDE delegates implementation to a coding agent such as **Claude Code** while retaining full responsibility for correctness.

The lesson also names **Codex, Gemini CLI, GitHub Copilot and Cursor** as related products. They are not identical, but they share the basic idea of using models to inspect or modify software.

### Important mental model
```text
User requirement
    ↓
Coding agent proposes actions
    ↓
Agent's tools read files / write files / run commands
    ↓
Developer validates results
    ↓
Accepted change becomes repository history
```

A coding agent is itself a tool-using agent. The same concerns we discussed for enterprise customer-support tools also apply to coding: **permissions, limits, evidence and human review**.

> The agent writes the code, but you answer for it.

## 2. The three habits
### Habit A: plan in phases
Do not paste a large requirements document and ask “build the assistant” without boundaries. The lesson's cautionary case describes an agent changing **30 files** after a long autonomous run, leaving the team unable to explain the choices.

A better approach is to split the work into manageable, reviewable increments: one identity-verification capability, one handoff workflow, one evaluation harness, and so on.

### Habit B: give the agent rules
Project-level guidance is stored in a root **`CLAUDE.md`** file. This is not a magical security boundary. It is durable context that should be read at the start of subsequent sessions.

### Habit C: require proof and checkpoints
After every small phase:
1. Run actual tests.
2. Read results.
3. Inspect diffs line by line.
4. Verify tests themselves were not weakened.
5. Commit the accepted changes.

## 3. Plan Mode versus implementation mode
**Plan Mode** is for analysis, repository reading and clarification without writing code. The engineer should review the proposed phases as if reviewing a colleague's implementation proposal.

A useful planning request:
> “Read the requirements, inspect the repository and project rules, list what is unclear, propose small phases with checks, and do not edit files until the plan is approved.”

The plan should be **saved to a repository file**, not only kept in the agent conversation. It needs to survive a context reset.

### Example phase structure
| Phase | Deliverable | Evidence |
|---|---|---|
| A | Email one-time-code verification | Unverified sessions cannot read orders |
| B | Human handoff in ServiceNow with summary | Test that a case is created with expected context |
| C | Evaluation replay of 40 labeled chats | Metrics compared with acceptance targets |

The ordering is illustrative and depends on the project's actual brief.

## 4. Persistent rules in `CLAUDE.md`
The transcript's example project rules include:
- Exact command to run fast tests.
- Model name defined in **one constant** rather than duplicated.
- **Refunds are only drafts**, never executed by the assistant.
- **No writing to Ledgerline** beyond owner-approved tables.

An illustrative rules document:
```markdown
# Coding-agent rules

- Run fast tests using the repository's documented test command.
- Route model requests through the centralized adapter.
- Define model name in one constant.
- Never execute a refund: proposals are drafts for review.
- Do not modify Ledgerline tables outside the owner's allowlist.
- After each phase, show changed files and test output.
- Do not modify expected tests merely to make the suite green.
```

Rules focus the agent, but do not stop a tool from doing something unsafe. That's why a second layer is necessary.

## 5. Permission settings: allow, ask and deny
The lesson distinguishes:
- **Allow**: ordinary reading and tests.
- **Ask first**: installing packages, pushing to GitHub.
- **Deny**: reading a secrets file containing API keys.

**Why not allow everything?** A coding agent may misinterpret a request, run a destructive command or expose confidential data. Convenience is a weak reason to remove approval boundaries.

Permission controls are **enforced by tooling**, not merely requested in prose. They should be scoped: not all Git commands are equally safe.

### Rule versus enforcement
| Example | Agent guidance | Hard enforcement |
|---|---|---|
| Don't expose a token | Instruction in CLAUDE.md | Deny secret-file access and avoid token printing |
| Don't move money | Prompt/business rule | No refund-execution capability |
| Run tests | Instruction and plan | Developer independently runs tests |
| Respect Ledgerline limits | Repo rule | DB credentials limited to approved tables |

## 6. Why reset context with `/clear`
A long agent chat accumulates:
- Failed attempts.
- Outdated file details.
- Partial approaches.
- Unrelated decisions.
- Irrelevant tool output.

This can distract the model, reduce quality and raise cost. The course recommends **Claude Code's `/clear` after each phase**.

A fresh context is only safe if the important state is durable:
- Approved plan in the repo.
- Project rules in `CLAUDE.md`.
- Tests and acceptance checks.
- Git commit of prior verified work.

**Clear conversation noise, not institutional knowledge.**

## 7. Verify with evidence
A coding agent may say:
> “All tests pass.”

That sentence is only a **claim**. The developer must run the tests or inspect trusted CI output.

### Important review scope
Read **every changed line**, particularly:
- Authentication and authorization.
- Tool dispatch and data handling.
- Error branches.
- New dependencies.
- **Test modifications**.

Why scrutinize test changes? The agent could satisfy an assertion by weakening or deleting it rather than fixing the underlying code.

Example of a misleading “fix”:
```python
# Before: meaningful assertion
assert response.status_code == 403

# Bad agent change: removes the requirement rather than fixing authorization
assert response.status_code in (200, 403)
```

Such a test can pass while security becomes weaker.

## 8. Reuse the labeled evaluation set
After agent changes that alter behavior, replay **Marco Rossi's 40 labeled support conversations** and compare:
- Intent correctness.
- Correct action.
- Missed necessary human handoffs.
- Unnecessary handoffs.

Software unit tests and behavioral LLM evaluations prove **different properties**. Both may be necessary.

## 9. Git checkpoints
After accepting a phase, commit it. If phase three unexpectedly rewrites half the tools, the team can revert to the last verified commit and adjust the plan.

Without checkpoints, the scope of recovery can expand from one small phase to a day of tangled changes.

Illustrative acceptance sequence:
```bash
git diff --check
python -m pytest -m "not live"
git diff
git status
git add <reviewed files>
git commit -m "Complete reviewed phase"
```

The exact commands depend on the project's tooling and permission settings.

## 10. Human accountability at DemoStore
**Ravi Menon**, Director of E-commerce Engineering, has a team that reviews pull requests. They need to understand what the code does, what tests were run and why the approach fits their systems.

“The agent wrote it” is not an acceptable explanation if a security flaw, billing mistake or broken integration reaches the client's environment.

## 11. Failure modes and remedies
| Failure | Consequence | Remedy |
|---|---|---|
| Whole app in one prompt | Too many unreviewed changes | Small phased plan |
| Permissions allow everything | Secret exposure or destructive commands | Allow/ask/deny settings |
| Plan only in chat | Context reset loses decisions | Repository plan file |
| Accept agent's test claim | Unverified code enters project | Run tests independently |
| Don't review test edits | Tests can be weakened | Diff review |
| No Git checkpoints | Large recovery cost | Commit each phase |
| Keep stale session forever | Confused context and cost | `/clear` and fresh sessions |

## 12. Interview answer
**Question:** “How can an FDE safely delegate software development to an AI coding agent?”

**Answer:** “I start from an approved specification, ask for a phased plan without code changes, enforce project rules and tool permissions, review each small implementation against a runnable acceptance check, independently run tests and inspect the diff including test changes, and commit the verified phase. Then I begin the next phase with a fresh context and durable repository notes. The team remains accountable for every line it accepts.”

### Revision checklist
- [ ] Three habits: **plan, rules, proof/checkpoint**.
- [ ] Remember the **30-file uncontrolled change** failure.
- [ ] `CLAUDE.md` stores durable rules.
- [ ] **Allow / ask / deny** permissions.
- [ ] **No execute refund**, **restricted Ledgerline writes**.
- [ ] Use **`/clear`** between phases.
- [ ] Agent's “tests pass” message is **not proof**.
- [ ] Inspect **changed tests**.
- [ ] Replay **40 labeled cases**.
- [ ] **Ravi Menon** owns engineering review.
- [ ] Key line: **plan in phases, prove each, commit before next**.

---

# Phase-by-Phase Operating Playbook for a Coding Agent

## The difference between generated code and accepted code
A coding agent can finish producing files before the engineer has decided that those files are correct. The course treats **generation** and **acceptance** as separate events.

**Generation:** Claude Code edits files and reports its work.

**Verification:** Engineer runs tests and examines the diff.

**Acceptance:** Engineer approves a small phase and commits it to Git.

This distinction is the core of the lesson's phrase **“The agent writes the code, but you answer for it.”**

## A durable plan is more than a chat summary
Consider an illustrative repository file:
```markdown
# Plan — DemoStore Chat Service
Phase 1: Shared intent schema.
Verification: fast schema tests and code review.
Phase 2: Conversation history support.
Verification: old callers and new history callers work.
Phase 3: FastAPI API.
Verification: request/session contract tests.
```

This is not the exact plan file from the video, but it demonstrates why the plan must be accessible after `/clear` and between coding sessions.

## What to inspect in a proposed phase
1. **Scope:** Is it small enough to understand?
2. **Interfaces:** Which existing contracts may change?
3. **Safety:** Which permissions and identity checks are affected?
4. **Tests:** What evidence will establish completion?
5. **Rollback:** Could we revert this without undoing unrelated work?
6. **Documentation:** Will new assumptions be recorded durably?

A model suggestion is a draft, not an approval.

## Permission categories in context
| Setting | Course example | Why |
|---|---|---|
| Allow | Read code / run tests | Routine development |
| Ask | Install dependencies / push GitHub | Changes external state or dependencies |
| Deny | Open secrets file containing API key | Prevent unnecessary secret exposure |

Never assume a prompt rule like “Don't read secrets” is equivalent to a **deny rule enforced by the agent's tool runtime**.

## Reviewing an AI-generated test suite
A broken application can still have a green suite if the agent changes tests to match incorrect behavior.

Look for:
- Assertions removed.
- Security tests weakened.
- New expected values that merely mirror the bug.
- All mocks, no meaningful integration tests.
- Test skips hiding failures.
- Changed fixture data making failure cases easier.

The engineer's independent run must be paired with **diff review**.

## Example escalation of review seriousness
A pure UI text change may require a simple build and visual check.

A change to **`find_order` authorization** requires scrutiny of verified session identity, another customer's record and nonexistent order behavior.

A change to refund-related code requires validation that **no money-moving capability** became accessible to the agent.

A change to LLM behavior requires replaying **Marco's 40 labeled cases** and examining missed handoffs.

## Recovery example
Phase two was committed. Phase three accidentally rewrites the order tools and makes a previously forbidden read possible.

Correct response:
1. Stop the agent.
2. Inspect the diff.
3. Do not accept green tests if security semantics changed.
4. Restore the last known-good commit if necessary.
5. Rewrite the smaller phase prompt with explicit authorization tests.
6. Try again in a clean session.

**This is why commits after each accepted phase matter**—the recovery scope remains bounded.

## Why Ravi's review is essential
**Ravi Menon and his engineering team** own the production systems. They need to understand:
- What changed.
- Why it changed.
- Which tests were run.
- Where data flows.
- Which operations the agent can perform.

A pull request explanation saying only **“AI generated this”** provides none of that accountability.


---

# FDE Interview Questions & Answers — Comprehensive Practice Bank

Use these prompts for oral practice. Answer aloud before revealing the explanation beneath each question.

<details>
<summary><strong>1. How is a coding agent like the earlier customer-support agent?</strong></summary>

Both use an LLM-plus-tools loop; a coding agent's tools read/edit files and execute commands.

</details>

<details>
<summary><strong>2. Which coding-agent products were mentioned?</strong></summary>

Claude Code, Codex, Gemini CLI, GitHub Copilot and Cursor.

</details>

<details>
<summary><strong>3. What is the core accountability rule?</strong></summary>

The agent writes code, but the developer is responsible for accepting it.

</details>

<details>
<summary><strong>4. What are the three habits?</strong></summary>

Plan in phases, define project rules, demand proof and a checkpoint after every phase.

</details>

<details>
<summary><strong>5. What happens with 'build the assistant' as one huge prompt?</strong></summary>

The lesson warns of an uncontrolled run changing around 30 files without clear decisions or reviewable boundaries.

</details>

<details>
<summary><strong>6. What is Plan Mode for?</strong></summary>

Reading code, asking clarifying questions and proposing a plan without changing files.

</details>

<details>
<summary><strong>7. Why store the plan in the repository?</strong></summary>

It survives session resets and can be reviewed, versioned and used by the next coding-agent session.

</details>

<details>
<summary><strong>8. What belongs in CLAUDE.md?</strong></summary>

Test command, centralized model constant and project rules such as refunds only as drafts and restricted Ledgerline writes.

</details>

<details>
<summary><strong>9. What are the three permission categories?</strong></summary>

Allow routine reads/tests; ask before installs/pushing; deny secret-file access.

</details>

<details>
<summary><strong>10. Why are prompt-based restrictions not enough?</strong></summary>

Instructions guide behavior; actual permissions and code-level checks enforce allowed actions.

</details>

<details>
<summary><strong>11. What does /clear do in the described workflow?</strong></summary>

Resets stale coding-agent conversation context between phases while plans and rules remain in files.

</details>

<details>
<summary><strong>12. Why isn't 'all tests pass' proof?</strong></summary>

It is only model-generated text until the engineer runs tests or inspects trustworthy output.

</details>

<details>
<summary><strong>13. Why review changed tests as carefully as changed code?</strong></summary>

An agent may weaken or rewrite expected results to make incorrect behavior appear to pass.

</details>

<details>
<summary><strong>14. Why replay Marco's 40 examples?</strong></summary>

Code-level tests don't automatically measure model accuracy and missed handoffs after behavioral changes.

</details>

<details>
<summary><strong>15. What is a Git checkpoint?</strong></summary>

A commit after a verified phase, giving a known-good recovery point.

</details>

<details>
<summary><strong>16. What do you do if phase three rewrites too much?</strong></summary>

Revert to the latest verified commit if necessary, tighten the plan and retry a smaller phase.

</details>

<details>
<summary><strong>17. Who reviews DemoStore's engineering changes?</strong></summary>

Ravi Menon's engineering team; 'the agent wrote it' is not a justification.

</details>

<details>
<summary><strong>18. What is the central rule?</strong></summary>

Plan in phases, prove each one, and commit before the next.

</details>

## How to practise with this chapter

1. Explain the lesson's actual business problem in 45 seconds.
2. Recreate the architecture or sequence from memory.
3. Recall exact numbers and named decision-makers without checking the notes.
4. Explain one failure case and the deterministic safeguard.
5. Use the Q&A above to prepare a two-minute FDE interview answer.

