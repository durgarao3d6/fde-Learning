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
