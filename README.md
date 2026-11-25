# FDE Learning

A practical study guide to Forward Deployed Engineering, using the fictional DemoStore case study.

This repository hosts a chapter-by-chapter reading website. All examples are educational and DemoStore is fictional.

## Contents

- AI model evaluation and selection
- Tool calling and structured outputs
- Secure agent loops and authorization
- Choosing the right level of AI
- Build, buy, or extend
- Client discovery and workflow shadowing
- Time-boxed AI prototypes
- Safe coding-agent development
- Interview-driven implementation briefs
- FastAPI chat services and React widgets

## Run locally

Open `index.html` in a browser (or use a local static server). No build process is required.

## Deployment

The site can be hosted by Vercel as a static project, or published through GitHub Pages using the repository root.

## Disclaimer

This is an independent set of educational study notes based on fictional course examples, not an official course publication. Numerical figures and pricing cited in lessons are historical or illustrative unless clearly stated otherwise.

## Comprehensive chapter format

The expanded notes are designed for revision and FDE interviews. Chapters include detailed lesson walkthroughs, technical flows, concepts and trade-offs, DemoStore-specific facts, code examples, operational pitfalls, FDE interview answers and review checklists.

The study collection now spans **16 chapters**, with comprehensive FDE interview practice, knowledge checks and revision lists. Chapters 11–15 follow the full-length reference-note format. The notes also distinguish transcript facts from additional illustrative engineering examples.

The latest two topics are especially detailed:
- [Interview-driven implementation brief](content/10-interview-brief.md) — all 8 original questions, proposed answers, the verification contradiction, 3 manually added requirements, 30 FDE interview prompts and a complete checklist.
- [FastAPI and React widget implementation planning](content/11-chat-service-widget.md) — 5-part brief, 3 clarification questions, 7 phases, phase-one code verification, 36 FDE interview prompts and a complete checklist.

For completeness, see [the study coverage index](STUDY-GUIDE.md).


## Chapter 12 — Review, Run, and Validate the Chat App

[Read the complete study notes](content/12-review-run-chat-service-widget.md). This chapter follows the long-form course format: 23 parts, 58 numbered sections, code walkthroughs, 40 FDE interview questions with answers, six knowledge-check questions, and detailed revision checklists. Its key findings are the nearly empty saved plan file, the human-support button being disabled during loading, the pending-reply fix, and the unresolved first-session-ID race. The course's **DemoStore** app is distinct from this static study website.


## New Lessons: Frontend, Specifications, and Quality Gates

- [Chapter 13 — TypeScript Fundamentals for Browser Chat Widgets](content/13-typescript-browser-fundamentals.md): types, interfaces, async/await, fetch, `response.ok`, API contract validation and keeping Claude credentials on the server. Contains 40 FDE interview Q&A and six knowledge-check questions.
- [Chapter 14 — Two-Voice Specifications](content/14-two-voice-specifications.md): Marco-reviewed business stories, Ravi-reviewed technical contracts, traceability IDs, tests, out-of-scope decisions and never-do constraints. Contains 40 FDE interview Q&A and six knowledge-check questions.
- [Chapter 15 — Automated Checks and Git Pre-Commit Hooks](content/15-automated-checks-precommit.md): pytest, Ruff, mypy, tsc, pre-commit, fast local checks, slower PR checks, bypass restrictions and careful review of changed tests. Contains 40 FDE interview Q&A and six knowledge-check questions.

- [Chapter 16 — Build 1: Plan and Verify Phase One](content/16-build-one-agent-plan-phase-one.md): eight-phase agent plan, two discovered code gaps, approved OTP/scoring/commit decisions, parcel authorization correction, seven passing fast tests, verified plan, and hook-backed first commit. Includes 50 FDE interview Q&A and six self-check questions.
