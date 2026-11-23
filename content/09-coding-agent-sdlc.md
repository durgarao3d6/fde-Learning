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
