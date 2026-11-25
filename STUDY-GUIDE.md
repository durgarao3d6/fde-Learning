# FDE Learning — Comprehensive study guide and coverage index

This companion is a map to the full chapter notes and FDE interview questions. Chapters 11–15 follow the long-form study-note style, with carefully numbered explanations, detailed walkthroughs, FDE interview answers, knowledge checks and revision checklists. The website's **FDE Interview Q&A** controls jump to each chapter's collapsible practice bank. For every question, answer aloud before revealing the written response.

## Coverage and interview count

| Chapter | Detailed study material | FDE interview questions |
|---|---|---:|
| [01 — Model selection and evaluation](content/01-model-evaluation.md) | 40 labeled chats, 21 handoff cases, 360 requests, 20 workers, pricing assumptions, schema field pitfall | 15 |
| [02 — Tool calling](content/02-tool-calling.md) | Tool schemas, stop reasons, tool-use IDs, Ledgerline middleware, nightly carrier CSV, verified identity | 15 |
| [03 — Agent guardrails](content/03-guardrails.md) | Per-tool authorization, not-found privacy, refund proposals, 10-round maximum, ServiceNow fallback | 15 |
| [04 — Agent loop](content/04-agent-loop.md) | JSON/CSV fixtures, dispatcher, message state, Leeds depot, missing tracking, production hardening | 15 |
| [05 — Choosing whether to use AI](content/05-choosing-ai.md) | Four levels, self-service, deterministic return rules, structured extraction, full agent decision | 15 |
| [06 — Build vs buy vs extend](content/06-build-buy-extend.md) | ServiceNow/Now Assist, vendors, $0.99 vs $7, weekly/yearly arithmetic, real seven-day resolution | 17 |
| [07 — Discovery and shadowing](content/07-discovery-shadowing.md) | Marco interviews, 90-minute example, actual systems, estimates vs observation, German spreadsheet | 18 |
| [08 — Time-boxed prototypes](content/08-prototype.md) | Five-day limit, 36/40 + zero missed handoffs, real/fake boundary, 20 test orders, shortcut ledger | 19 |
| [09 — Coding agents and AI SDLC](content/09-coding-agent-sdlc.md) | Phase plans, CLAUDE.md, allow/ask/deny, /clear, independent tests, Git, Ravi's review | 18 |
| [10 — Interview-driven brief](content/10-interview-brief.md) | Full eight-question walkthrough, order-verification conflict, three missed requirements, brief file | 30 |
| [11 — Chat service and widget](content/11-chat-service-widget.md) | Uploaded full notes: five-point brief, three questions, seven phases, Pydantic fields, four tests and phase-one commit | 17 main additional Q&A |
| [12 — Reviewing and running the chat app](content/12-review-run-chat-service-widget.md) | Seven phases finished, 13 fast and 3 live tests, failed plan-file copy, disabled handoff button, session-ID race, browser test and FastAPI docs | **40 interview Q&A + 6 knowledge-check prompts** |
| [13 — TypeScript browser fundamentals](content/13-typescript-browser-fundamentals.md) | Interfaces, async/await, fetch, runtime response checks, HTTP errors and browser secrets | 40 + 6 knowledge checks |
| [14 — Two-voice specs](content/14-two-voice-specifications.md) | Business and IT voices, traceability, scope exclusions, never rules, PR-based updates | 40 + 6 knowledge checks |
| [15 — Automated checks and pre-commit hooks](content/15-automated-checks-precommit.md) | pytest, Ruff, mypy, tsc, Git hooks, fast vs slow tests, bypass and weakening risks | 40 + 6 knowledge checks |
| **Total** | **15 lessons** | **354 numbered FDE Q&A (counted across chapter banks), plus knowledge checks** |

## What each chapter includes

- **Detailed topic walkthrough:** The reasons behind the design choices rather than only a high-level summary.
- **Named stakeholders and facts:** DemoStore, Marco Rossi, Lena Brandt, Ravi Menon, Sofia Lindqvist, ServiceNow and Ledgerline, as relevant.
- **Precise numbers and constraints:** Evaluation volume, time boxes, per-resolution examples, authorization boundaries and release stages.
- **Architecture and code:** Tool loops, schemas, API examples and controls, with invented snippets labeled **illustrative**.
- **Risks and common errors:** What goes wrong, why it matters and how to detect or prevent it.
- **FDE interview Q&A:** Questions and model answers specifically connected to each lesson.
- **Revision checklist:** Key points to recall before interviews and implementation.
- **Links to earlier lessons:** How model evaluation, discovery, guardrails and coding-agent engineering connect.

## How to use this site

1. Read a chapter from the beginning; don't skip its decision or failure examples.
2. Reproduce the architecture in a notebook without looking at the diagram.
3. Open the **FDE Interview Q&A** section; answer each prompt before expanding it.
4. Mark off the chapter's revision checklist.
5. Practice a two-minute response to “How did you make this decision for a client?”
6. Revisit the exact numbers and deliberately deferred features.

## Fidelity and limitations

These are extensive **reconstructed study notes**, not verbatim official course transcripts or exact copies of every previous ChatGPT response. Original lesson content and exact figures known in this conversation are retained. Additional examples are marked as illustrative; model pricing cited by the course is historical or educational, not a current rate.

Only the original video transcripts and saved responses can establish an exact line-by-line completeness audit. If you want strict fidelity to every sentence of an older chapter, preserve its corresponding transcript alongside the notes for comparison.

## Hosting

Source is [GitHub](https://github.com/durgarao3d6/fde-Learning). The static reader is deployed to [Vercel](https://fde-learning-peach.vercel.app). Automatic GitHub-triggered redeployment requires granting Vercel's GitHub integration access to the new repository.
