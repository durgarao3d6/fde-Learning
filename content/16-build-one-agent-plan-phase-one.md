# Complete Study Notes: Planning DemoStore Build 1 with Claude Code and Verifying the First Phase

**FDE Course · Agentic Software Development · Claude Code · Plan Mode · Requirements Specs · Security Gaps · Email OTP · ServiceNow · Evaluation · Git Branches · Pre-Commit Hooks · Testing · Independent Verification**

> **Practical rule:** **Let the agent plan before it codes, read the plan as carefully as code, and accept a phase only after you've run its check yourself.**

This chapter continues from [Chapter 15 — Automated Checks and Pre-Commit Hooks](15-automated-checks-precommit.md). DemoStore's **chat service and widget already work**, but the larger assistant prototype still needs verification, tool security, human handoff and evaluation. Claude Code will implement the remaining work in small phases, under a reviewed specification and automatically enforced checks.

**Source fidelity:** The detailed sequence, exact numbers, decisions, file roles, discovered gaps, tests and phase-one outcome below come from the supplied lesson transcript. Illustrative code, example commands, named paths beyond those explicitly provided, and proposed future test designs are clearly identified as educational examples; they are **not claims about unseen files in the course repository**.

---

# Part 1: What Is This Lesson About?

## 1. From a Working Chat Widget to Build 1

Earlier lessons created and tested a service that can answer messages over the web and a React/TypeScript chat widget that displays them. Having a chat window does **not** mean the complete customer-support assistant is ready.

The full prototype must account for:
- Order and parcel authorization.
- Email one-time-code verification.
- Correct session and widget behavior.
- A working handoff to a person.
- Evaluation of actual required actions on labeled conversations.
- Code quality checks and reviewable Git history.

The lesson uses **Claude Code** to build the missing pieces. The engineer's role remains **planning, reviewing, validating and accepting** rather than passively approving everything the coding agent reports.

## 2. What This Lesson Shows — and What It Does Not

The video shows:

1. **What Claude Code will read** before coding.
2. Creating a **separate working branch**.
3. Asking for a **phased plan in Plan Mode** and evaluating it.
4. Approving three remaining design decisions.
5. Allowing Claude Code to implement **phase one only**.
6. Independently inspecting the plan and running tests.
7. Making the first checked **Git commit**.

The lesson does **not** show phases two through eight being implemented. Those are for the next lesson(s).

### Why the distinction matters

The speaker says the plan includes OTP, ServiceNow and model evaluation—but those features are **future planned work at the point where this lesson ends**. Only the initial security correction is completed and verified.

---

# Part 2: Before the Agent Starts — What It Reads

## 3. Three Sets of Input Documents

Claude Code works from:

- **The specification**, describing business goals, technical implementation and boundaries.
- **`CLAUDE.md`**, providing persistent project rules read at the beginning of every session.
- **The settings file**, enforcing tool permissions.

Existing source code is a fourth essential input: the coding agent reads it to discover mismatches between the spec and the working prototype.

```text
                         Claude Code
                             │
        ┌────────────────────┼──────────────────────┐
        ↓                    ↓                      ↓
   Two-voice spec         CLAUDE.md          Permission settings
   - business lines       - test commands    - allow / ask / deny
   - IT lines             - model adapter    - block secrets / bypass
   - acceptance           - no refunds
   - limits               - ERP limits
        └────────────────────┼──────────────────────┘
                             ↓
                       Existing code
                             ↓
                  Plan reviewed by engineer
```

The key is that the agent reads **durable repository sources**, not the developer's memory of a meeting.

---

# Part 3: The Spec Is the Source of Truth

## 4. Business Voice: What the Customer Must Receive

The spec's first part expresses business requirements using customer language. The transcript repeats two key lines:

**Business rule:**
> A customer who is **not verified sees no order data**.

**Measured goal:**
> At least **36 of 40 labeled support conversations** must get the **right action**.

The fraction is:

```text
36 / 40 = 90%
```

This is an **action-quality threshold**; a system can sound conversationally convincing while choosing the wrong action.

The prototype lesson also discussed **zero missed mandatory handoffs** as a separate illustrative safety criterion. This transcript explicitly repeats the **36/40 correct-action** target; keep the lessons' stated requirements distinct.

## 5. IT Voice: How the Business Rules Are Built

The spec's second part says how to meet the business rules. Each IT line identifies the business requirement it serves.

Examples from the preceding spec lesson include:
- Approved tools `find_order` and `delivery_status`.
- A trusted verified-customer identity from the **application session**.
- `find_order` accepting only the **order number** from the model.
- A central model adapter.
- A **ten-round** agent limit.
- A human support path through **ServiceNow**.

This lesson's plan tests whether the existing code actually follows those instructions.

## 6. Traceability Matters Before Implementation

Each business line has an ID, each technical line refers to a business line, and each test should prove the relevant requirement.

Example (IDs are illustrative):

```text
BR-VERIFY:
Unverified customer receives no order data
          ↓
IT-VERIFY:
Order/parcel tools use verified session identity
          ↓
TEST-VERIFY:
Attempt real fixture lookup with no OTP verification
          ↓
Expected: no protected order or parcel information
```

This gives the coding agent a concrete target and the human reviewer a way to detect scope drift.

## 7. Limits at the End of the Spec

The spec ends with two kinds of boundaries:

**Out of scope:** What this particular build intentionally leaves out.

**Never rules:** Actions that the assistant must not be allowed to perform.

Earlier DemoStore lessons give examples such as:
- No direct money-moving tools.
- No unauthorized Ledgerline table writes.
- No order data before verification.
- No invented delivery dates in place of tool evidence.

The model should not “helpfully” add forbidden capabilities or implement features excluded by the current spec.

---

# Part 4: The Project Rules in CLAUDE.md

## 8. Persistent Rules for Every Session

The source says `CLAUDE.md` contains:

1. **How to run the fast tests**.
2. **The central adapter as the only way to call Claude**.
3. **The refund rule**.
4. **The Ledgerline rule**.

The business rules from earlier chapters mean that refunds must not be executed directly by the AI and Ledgerline writes must stay within owner-approved operations.

### Why keep rules in a file?

Because each new Claude Code session needs a stable source of project instructions. A one-time chat message is not a reliable substitute for persistent project guidance.

### An illustrative project-rules excerpt

```markdown
# DemoStore Engineering Rules

- Use the documented fast-test command before acceptance.
- Route every Claude model call through the central adapter.
- The agent may draft refund recommendations but must not move money.
- Do not write to Ledgerline outside ERP-owner-approved tables.
- Review changed tests and security checks before a commit.
```

The example is **not the exact course `CLAUDE.md`**.

---

# Part 5: Tool Permissions — Allow, Ask, Deny

## 9. What the Settings File Enforces

The lesson describes three permission groups:

| Group | Action | Why |
|---|---|---|
| **Allow** | Run approved tests | Routine validation |
| **Allow** | Make local commits | Checkpoint work |
| **Ask** | Install a package | Dependency/environment change |
| **Ask** | Push to GitHub | External remote state change |
| **Deny** | Read secrets file | Protect API keys and credentials |
| **Deny** | Skip the pre-commit hook | Prevent bypassing automatic checks |

The transcript explicitly says **tests and commits run on their own**, **package installs and pushing ask first**, and **secrets reads and hook bypass never run**.

## 10. Why This Matters in Auto Mode

Later the engineer enables **Auto Mode**, allowing Claude Code to read and write files in the folder without approval for each routine operation.

That does **not** mean every command is permitted.

The settings file still applies restrictions independently of the agent's prose instructions.

### Important distinction

**`CLAUDE.md` tells the agent how to work.**

**Permission settings determine which actions it is allowed to execute.**

**Pre-commit hooks run checks automatically when a commit is attempted.**

Three complementary layers, not interchangeable ones.

## 11. A Commit Is Not a Push

The course allows the agent to make approved **local Git commits** without repeatedly asking.

It still asks before a **remote push**.

These are different actions:
- A commit records a snapshot locally.
- A push sends history to the remote repository.

Separating them prevents a coding agent from publishing unreviewed work automatically.

---

# Part 6: Create a Separate Branch Before Coding

## 12. Why Start a Branch?

The narrator creates a branch named:

```text
build-one
```

This happens **before** Claude Code makes code changes.

A branch isolates the current build from the mainline and provides a reviewable history for its phases.

### Example command — illustrative

```bash
git switch -c build-one
```

The exact command used in the demonstration is not supplied; the branch name **build-one** is.

## 13. The Course Tag as a Recovery Point

The lesson notes that **a tag in the course repository** identifies the version at the **end of this day**.

Why?

Different coding agents may produce different implementations while satisfying the same requirements. A learner who diverges from the instructor's code can use the course's tag to catch up.

**The actual tag name is not stated in the transcript.** Do not invent it.

### Tag vs branch vs commit

| Git concept | Practical meaning in this lesson |
|---|---|
| Branch | Your independent development line for Build 1 |
| Commit | Checked snapshot after a phase |
| Tag | Named reference to the instructor's known course version |

This is general Git context added to explain the transcript.

---

# Part 7: Use Plan Mode Before Code Mode

## 14. What Plan Mode Allows

Claude Code is started in **Plan Mode**.

In this mode it:
- Reads the spec, project rules and code.
- Investigates gaps.
- Asks clarifying questions.
- Proposes a plan.

It **changes nothing yet**.

The engineer uses this separation to prevent premature implementation before reviewing architecture and scope.

## 15. The Planning Request

The user asks Claude Code to plan **Build 1** in **small phases**.

Every phase must end with a runnable check:
- **Fast tests**, or
- **Replay of the forty labeled conversations**.

The plan must be written to a file, with **one checkbox for every task**.

Once approved, Claude Code should implement **phase one only** and then **stop**.

### Reusable plan prompt — reconstruction, not verbatim

```text
Read the Build 1 spec, CLAUDE.md, the agent permission
settings, and the existing code. Work in Plan Mode.

Propose Build 1 as small, independently reviewable phases.
For each phase:
  - list concrete tasks with checkboxes
  - link tasks to requirements
  - state the verification command or the 40-chat replay
  - identify any security and compatibility gaps

Save the approved plan in a repository file.

Ask me to resolve open decisions.
Do not change code until I approve the plan.

After approval, implement phase one only,
show the evidence, and stop.
```

The transcript does not provide the literal prompt or the plan file path.

## 16. Why Checkboxes Matter

A plan written as prose is harder to audit. Task checkboxes make it easier to tell:
- Which pieces belong to the current phase.
- What the agent claims to have completed.
- Which tasks still wait.
- Whether it has stopped at the requested boundary.

But a checked box is still **a claim in a file**. The engineer must verify behavior independently.

## 17. The Tool Reads and Analyzes First

Claude Code reads the spec, rules and code before showing the plan.

The speaker mentions that this reading/planning process takes **about a minute and a half**, and the video skips that wait.

The teaching point is **not** the duration; it is that the agent has time to inspect existing implementation before proposing changes.

---

# Part 8: The Eight-Phase Build Plan

## 18. The Number of Phases

Claude Code proposes **eight phases** for Build 1.

This is a different plan from the **seven phases** used earlier to build the chat service and widget.

| Earlier build | This build |
|---|---|
| Seven phases | **Eight phases** |
| Create chat service and widget | Extend assistant prototype and close safety/integration gaps |
| Shared Pydantic intent, history, FastAPI, widget | Security fixes, OTP, handoff, 40-chat replay, final evaluation |

**Do not merge the two plans.** They belong to different stages of the DemoStore course.

## 19. What the Transcript Tells Us About the Eight Phases

The lesson summarizes the phases rather than listing exact titles for each. The following is a **study-oriented mapping** based on that sequence, **not the verbatim plan file**:

| Phase | Stated subject / goal | What we can verify from the transcript |
|---|---|---|
| **1** | Lock down parcel status access | Implemented and checked in this lesson |
| **2** | Close remaining early code gaps, including central model adapter | Described as upcoming work |
| **3** | Email one-time-code capability | Planned |
| **4** | Code verification in chat service and widget | Planned |
| **5** | Real human-support handoff | Planned |
| **6** | Replay forty labeled chats and diagnose actions | Planned |
| **7** | Improve prompt one change at a time until target holds | Planned |
| **8** | Close out Build 1 and use held-out evaluation | Planned |

**Caution:** This table organizes eight topics, but the transcript does **not explicitly assign each topic to its numbered phase** beyond phase one. The actual eight phase headings and task lists live in the course plan file, which isn't included here. Treat the table as a conceptual reconstruction only.

## 20. The Early Phases Repair Existing Code

Claude Code finds **two real gaps in the current prototype before writing anything**.

This is an important result: planning is already valuable because the agent compares code with specification.

### Gap One: Unprotected Parcel Tool

The **parcel/delivery-status tool** answered anyone, whether verified or not.

That violates the spec's rule that **no private order information** should appear before verification.

### Gap Two: Direct Model Calls in Agent Loop

The **agent loop called Claude directly**, bypassing the required centralized adapter.

That violates the engineering rule:

> Every model call goes through the adapter; the model name is kept in one constant.

**Why detect these early?** They are not harmless implementation choices. One is an authorization gap; the other undermines the project's approved architecture.

---

# Part 9: Why the Parcel Tool Is a Serious Security Gap

## 21. The Order Tool Is Not the Only Private Data Entry Point

Earlier DemoStore lessons implemented customer ownership in `find_order`.

But if `delivery_status` accepts any tracking number and returns parcel details, an attacker can bypass the guarded order lookup.

```text
find_order
  └── owner check protects order details

delivery_status
  └── NO owner check → could reveal shipment information
```

The fact that one tool checks identity does not make **all tools** safe.

This is the recurring guardrail principle:

> **Every tool checks before it acts.**

## 22. Two Conditions the Parcel Tool Must Enforce

**Condition A: The visitor is verified.**

An unverified visitor receives **no parcel status data**.

**Condition B: The requested parcel belongs to that customer.**

A verified customer asking about somebody else's parcel must not learn its delivery status.

### Why do tracking numbers matter?

Tracking numbers are identifiers, not necessarily secure proof of ownership. A person may guess, obtain or reuse an identifier belonging to another account.

Ownership must be validated against **trusted server-side customer identity**.

## 23. Hide Existence of Other Customers' Parcels

The lesson specifies that a parcel belonging to another customer returns the **same answer** as a parcel that does not exist:

> **“No such parcel.”**

This is the same privacy pattern previously used for order lookup.

### The comparison

| Request | Expected user-visible outcome |
|---|---|
| Unverified visitor requests parcel | No protected parcel data |
| Verified owner requests own parcel | Permitted status data |
| Verified customer requests another customer's parcel | **“No such parcel”** |
| Verified customer requests nonexistent parcel | **Same “No such parcel”** |

If the service said “this parcel belongs to another customer,” it would reveal that the parcel exists.

## 24. Illustrative Guarded Implementation

```python
# Educational example — not exact course source.
def delivery_status(tracking_number: str, verified_customer_id: str | None):
    if verified_customer_id is None:
        return {"error": "verification_required"}

    shipment = parcel_database.lookup(tracking_number)

    if shipment is None:
        return {"error": "not_found"}

    if shipment.customer_id != verified_customer_id:
        return {"error": "not_found"}

    return {"status": shipment.status, "last_scan": shipment.last_scan}
```

**Important:** The model should not control the verified customer identity. Real implementations must use trusted session state and avoid returning sensitive metadata on an unauthorized lookup.

---

# Part 10: Why All Claude Calls Must Use One Adapter

## 25. Central Model-Call Architecture

The spec and `CLAUDE.md` require model calls through a centralized adapter, previously named `llm.py`.

The agent loop's direct provider call is an architectural inconsistency.

### Bad shape

```text
intent.py ──→ llm.py ──→ Claude

agent.py ──────────────→ Claude directly
```

### Desired shape

```text
intent.py ───┐
             ├──→ llm.py ──→ Claude
agent.py ────┘
```

Why centralize:
- Model identity/configuration has one source.
- Shared API behavior is easier to test.
- Future model changes need fewer edits.
- The coding agent is less likely to introduce divergent provider calls.

These are engineering explanations consistent with the rule; the transcript explicitly identifies the direct-call violation but does **not** show its repair during phase one.

---

# Part 11: OTP, Handoff and Evaluation — What Comes After Phase One?

## 26. Email One-Time Code

The plan includes adding an **email one-time code**.

The customer enters **her email in the widget**, and the code is sent to **that address**.

Later work must connect code verification to the trusted chat session so that order/parcel tools can unlock only for verified customers.

### Intended flow

```text
Customer enters email in widget
             ↓
Service sends OTP to that address
             ↓
Customer enters received OTP
             ↓
Application checks the OTP
             ↓
Trusted session records verified customer identity
             ↓
Authorized order tools may return owned order data
```

The transcript describes this as a **planned decision**, not a finished email service.

### Never do this

Do not accept the model's claim that the customer has verified her identity. Verification must happen in application code.

## 27. Real Human Handoff

The eight-phase plan includes the **handoff to a person**.

Earlier lessons showed a button that could send a request but the assistant honestly replied that transfer to a human was **not yet built**.

This Build 1 plan seeks to implement that missing capability.

The transcript does not show the ServiceNow case-creation code being implemented here. It is **future planned work**.

## 28. Replay of the Forty Labeled Chats

Later phases evaluate against **forty labeled customer conversations**.

The stated business goal is at least **36/40 correct actions**.

The replay tests whether the assistant takes appropriate actions, not whether its explanations sound plausible.

## 29. Improve the Prompt One Change at a Time

The lesson says to adjust the **prompt**, **one change at a time**, until the goal is met.

Why one change?
- You can attribute changes in results to a specific prompt revision.
- A regression is easier to diagnose.
- The evaluation process remains more reproducible.

If multiple prompt rules change at once, it becomes harder to know which change caused a new failure or improvement.

### Simple experimental log (illustrative)

| Version | One change | Correct actions out of 40 | Missed handoffs | Decision |
|---|---|---:|---:|---|
| v1 | Baseline | Measured result | Measured result | Record |
| v2 | Clarify one escalation rule | Measured result | Measured result | Compare |
| v3 | Clarify one policy condition | Measured result | Measured result | Compare |

No actual numerical post-change results are supplied in the lesson. Do not invent them.

---

# Part 12: Held-Out Evaluation — The Last Twenty Chats

## 30. What Does “Held Out” Mean?

The plan reserves **twenty held-out conversations** for the final phase.

These are not the same as the **forty chats repeatedly used during development**.

The held-out cases are intended as an **unseen evaluation** after tuning.

### Training/tuning versus final check

```text
40 labeled chats
   ↓
Repeated development evaluation
   ↓
Change prompt one item at a time
   ↓
Reach stated development goal
   ↓
Complete build
   ↓
Open 20 held-out chats ONCE
   ↓
Final independent performance signal
```

## 31. Why Only Open Held-Out Chats Once?

If the team repeatedly adjusts the prompt using held-out results, those cases stop being a fair final check. The system can become tuned to the “unseen” benchmark.

The lesson's explicit decision is:

> **Only at the end open the twenty held-out chats, once.**

This preserves some separation between iteration data and final evaluation data.

### Important scope note

The transcript does not say whether the twenty held-out chats are in addition to or sampled from any other larger corpus. It merely distinguishes **the forty used for replay and tuning** from **twenty held-out chats used once**.

## 32. The Evaluation Counts to Memorize

| Purpose | Conversations | When |
|---|---:|---|
| Development replay / optimization | **40** | Across later tuning phases |
| Required correct actions | **36 of 40** | Development acceptance target |
| Final held-out set | **20** | Open once, at the end |

**The held-out score is not reported in this lesson.** Do not claim the final evaluation passed.

---

# Part 13: Three Decisions Claude Code Asks the Engineer to Approve

## 33. Decision One — Where the OTP Goes

**Proposed decision:** The customer enters **her email in the widget**, and the one-time code is sent to that address.

**Human answer:** Approve.

This keeps code delivery associated with the address the customer supplies. In a real system, completing verification must still bind the result to an authorized customer/session; this is an additional engineering implication.

## 34. Decision Two — What Counts as the “Right Action”?

**Proposed decision:** Score the agent based on **which tool actually ran**, not what the model claimed in its words.

**Human answer:** Approve.

This is particularly important for tool-using agents.

### Why words are insufficient

A model may say:
> “I'll open a support case.”

But if the application never calls the ServiceNow case-creation tool, the case was not created.

Or it may say:
> “I checked the carrier.”

Without a corresponding executed tool call, that assertion is not proof.

### Outcome-based grading

**Poor metric:** Did the model *describe* the desired action?

**Stronger metric:** Did the application **execute the correct tool**?

This is a central FDE lesson about measuring **real system behavior rather than generated prose**.

## 35. Decision Three — When to Commit

**Proposed decision:** Commit **one phase at a time**, and **only when the engineer asks**.

**Human answer:** Approve.

This gives the engineer a checkpoint after independent review and avoids an agent silently committing several unverified phases together.

### Why this does not conflict with allowed Git commits

The settings file permits the commit command, but the **workflow** still requires a human instruction and validation before making a phase commit.

Capability and approval policy are distinct.

---

# Part 14: Approve the Plan and Enter Auto Mode

## 36. Human Review Before Implementation

The engineer examines the eight phases and the three outstanding decisions.

They all make sense, so the engineer **approves** the plan.

That order matters:

```text
Inspect source documents
        ↓
Plan Mode discovers gaps
        ↓
Eight phases proposed
        ↓
Three decisions clarified
        ↓
Human approves
        ↓
Auto Mode implementation
```

## 37. What Auto Mode Allows

The lesson describes Auto Mode as allowing Claude Code to **read and write files in the current folder without asking for every routine action**.

But the **settings file remains active**:
- Secrets file cannot be read.
- Hook bypass is denied.
- Package installation and pushing still require approval.

Auto Mode is a convenience for approved work, **not an authorization to ignore business rules or safety controls**.

## 38. The Stop Condition Is Explicit

Claude Code is instructed to:

> **Do phase one only and stop.**

This prevents a large uncontrolled run across all eight phases.

A trustworthy agentic workflow is bounded not only by technical permissions but also by **scope and review checkpoints**.

---

# Part 15: Phase One — Secure the Parcel Tool

## 39. What Was Wrong Before?

The **parcel tool** returned delivery status to **anyone**, even a visitor who had not been verified.

For an enterprise support assistant, this is a security defect.

## 40. What Phase One Changes

The fixed tool now ensures:

1. **Unverified visitor → no parcel status data**.
2. **Other customer's parcel → same “no such parcel” response as a nonexistent parcel**.

This follows the spec's identity and privacy requirements.

### What the transcript does not show

It does not provide the exact changed tool function, its parameters or its database lookup code. The behavior is described at the application level. The illustrative example in Part 9 should not be treated as a literal patch.

## 41. The Four New Fast Tests

Claude Code adds **four fast tests** to cover the parcel-tool correction.

The lesson doesn't list all four test names or exact assertions.

A sensible *illustrative* coverage breakdown is:

| Test scenario | Expected behavior |
|---|---|
| Unverified visitor | No protected parcel status |
| Verified owner | Own parcel status returned |
| Verified customer, foreign parcel | Generic “no such parcel” |
| Verified customer, nonexistent parcel | Same generic “no such parcel” |

**This table is proposed coverage, not a claim that the four original tests used these exact cases.**

## 42. Plan Checkboxes Updated

The agent checks off the tasks of phase one in the plan file and stops.

The narrator later verifies that **all four phase-one boxes are ticked**.

Keep the counts separate:

- **Four task checkboxes** in phase one.
- **Four fast tests added**.
- **Seven total fast tests pass** in the independent run.

The transcript gives these facts; it does not enumerate the exact task names or test IDs.

---

# Part 16: Agent Report versus Human Evidence

## 43. Claude Code's Completion Report

The agent reports phase one is complete and the fast tests pass.

But the course immediately applies the rule from earlier chapters:

> **That report is only text. Check it yourself.**

## 44. First Independent Check — Open the Plan File

The engineer opens the actual saved plan file.

**All four phase-one task boxes are checked.**

This confirms the recorded plan state, rather than relying on the message “phase complete.”

### Why inspect this file especially carefully?

In the earlier seven-phase build, Claude Code claimed it had saved a plan file, but the file contained only its title. This history makes independent artifact inspection essential.

## 45. Second Check — Run the Fast Tests

The lesson shows a feature of Claude Code's terminal interaction:

> A command beginning with an **exclamation mark (`!`)** runs directly in the shell from Claude Code.

The engineer uses that mechanism for the fast test run.

### Actual result

**Seven fast tests passed.**

**Three live tests were excluded.**

The live tests really call Claude, so they do not belong in the quick validation run.

### Compare with the previous lesson

The prior *chat-service completion* lesson reported **13 fast tests** in a different point in the course, whereas this new Build 1 phase shows **7 passed**. These results are from **different demonstration states/branches/lesson contexts**. The transcript does not give enough detail to reconcile the exact test inventories, so the new lesson preserves its own reported count rather than “correcting” it to 13.

## 46. Third Check — Inspect Git Diff Summary

The engineer examines `git diff` with a short summary.

**Four tracked files changed.**

An important caveat is stated:

> **The new plan file is not included in the diff yet, because Git is not tracking it yet.**

This is a classic Git distinction:

- `git diff` normally shows differences for **tracked files**.
- A newly created **untracked file** may not appear in that output.
- The absence of the file in `git diff` does **not** prove it doesn't exist.

### Illustrative commands

```bash
git diff --stat
git status --short
```

`git status` can reveal an untracked plan file that a tracked-only diff misses.

The transcript gives the use of diff and the missing untracked-file explanation, not the exact shell command spelling.

---

# Part 17: Commit the First Verified Checkpoint

## 47. When the Engineer Commits

After inspecting the plan, running the fast tests and reviewing changed files, the engineer saves phase one with **a local Git commit**.

The transcript states:

- The commit is made on branch **build-one**.
- The **pre-commit hook runs automatically**.
- The checks pass.
- The commit is saved.

## 48. What the Hook Runs

During this commit, the hook reports:

- **Ruff: passed**
- **Mypy: passed**
- **Fast tests: passed**
- **Widget check: skipped**

### Why was the widget check skipped?

Because **no widget file changed** during this phase. The hook is configured to avoid unnecessary frontend checking for a change confined to other files.

This illustrates a fast quality gate that runs relevant checks while limiting cost and delay.

### Important distinction

“Widget check skipped because no widget file changed” does **not** mean the widget is known to be correct under every condition. It means the local pre-commit workflow did not need to recheck untouched widget code for this specific change.

## 49. The Commit Produces a Recovery Point

The checked commit is a **checkpoint** on the Build 1 branch.

If a later phase breaks parcel authorization, the team can inspect or restore a known-good version of this phase.

### Full phase-one path

```text
Read spec and rules
      ↓
Create build-one branch
      ↓
Plan Mode: eight phases + identify gaps
      ↓
Human approves three decisions
      ↓
Auto Mode: implement ONLY phase one
      ↓
Secure parcel status authorization
      ↓
Add four fast tests
      ↓
Mark four plan checkboxes
      ↓
Agent stops and reports
      ↓
Engineer reads plan file
      ↓
Engineer runs fast tests → 7 pass, 3 live excluded
      ↓
Engineer examines diff → 4 tracked files changed
      ↓
Engineer commits
      ↓
Hook: Ruff + mypy + fast tests pass
      ↓
Widget check skipped (no widget files changed)
      ↓
Checkpoint recorded on build-one
```

---

# Part 18: Where This Fits in the Course's AI SDLC

## 50. A Systematic Model of Coding-Agent Oversight

This lesson combines previously taught practices:

**From discovery:** Know the actual customer need.

**From the two-voice spec:** Connect customer needs to technical rules and tests.

**From project rules:** Keep important constraints in `CLAUDE.md`.

**From permission settings:** Enforce allow/ask/deny actions.

**From pre-commit hooks:** Run quality checks automatically.

**From phased planning:** Implement one manageable piece at a time.

**From independent review:** Inspect artifacts and run checks yourself.

**From Git:** Commit accepted work at a small recovery checkpoint.

The resulting process is much more robust than asking a coding agent to “build everything and tell me when done.”

## 51. Why the Agent Finding Two Gaps Is Valuable

It discovered both:
- A parcel-status authorization weakness.
- A model-call architecture inconsistency.

These are the kinds of issues that can survive in a prototype if the coding agent simply appends new features without first examining the system.

Plan Mode serves as **architecture/code audit before implementation**.

## 52. Why Only One Phase?

A coding agent may be capable of writing all eight phases in a long run, but then the developer must inspect the whole combined change at once.

Small phases give the FDE a chance to:
- Detect security regressions early.
- Read understandable diffs.
- Run a relevant check.
- Discuss deviations.
- Commit an accepted version.
- Begin the next phase with a clean context.

The course's rule remains: **one phase, one review, one checkpoint**.

---

# Part 19: Mistakes to Recognize

## 53. Mistake — Start Coding Before Reading the Spec

If the agent codes from memory or a vague prompt, it may overlook verification, adapter and action-scoring rules.

**Correction:** Read the spec, `CLAUDE.md`, settings and code before planning.

## 54. Mistake — Forget a Second Tool's Authorization

Protecting `find_order` while exposing parcel status through another tool leaks private data.

**Correction:** Require verified identity and ownership checks on **every relevant tool**.

## 55. Mistake — Let Model Text Define “Correct Action”

The assistant can claim “I opened a case” without actually invoking the tool.

**Correction:** Score **executed tools and outcomes**.

## 56. Mistake — Tune on Held-Out Conversations

Opening the twenty final examples repeatedly during prompt iteration contaminates the final evaluation.

**Correction:** Use the forty development cases for iteration; open twenty held-out cases once at the end.

## 57. Mistake — Trust a Report, Not the Plan

A checked box or completion message may not match source code or real behavior.

**Correction:** Open the plan, run tests and inspect diff before acceptance.

## 58. Mistake — Assume Git Diff Lists Untracked Files

The plan may be newly created and not yet included in `git diff`.

**Correction:** Check Git status in addition to diff.

## 59. Mistake — Assume a Green Hook Proves Production Readiness

Ruff, mypy and fast tests passing mean the configured checks passed. They do not establish that OTP, human handoff or the forty-chat target are achieved.

**Correction:** Keep phase scopes and evidence separate.

---

# Part 20: Facts from the Transcript vs Illustrative Explanations

## 60. Fidelity Table

| Statement | Status |
|---|---|
| Chat service and widget already work | Explicit transcript |
| Full prototype is larger; Claude Code builds remaining work | Explicit transcript |
| Start own branch named build-one | Explicit transcript |
| Course repo has a tag for the end-of-day version | Explicit transcript; tag name not given |
| Spec includes no unverified order data and 36/40 correct actions | Explicit transcript |
| IT lines refer to business lines | Explicit transcript |
| CLAUDE.md includes tests, adapter, refunds, Ledgerline limits | Explicit transcript |
| Settings allow tests/commits; ask installs/push; deny secrets/hook bypass | Explicit transcript |
| Plan Mode reads and asks, changes nothing | Explicit transcript |
| Plan contains eight phases | Explicit transcript |
| Agent finds open parcel tool and direct Claude calls | Explicit transcript |
| OTP, verification, handoff, replay and prompt tuning are planned | Explicit transcript |
| Agent requests three decisions and human approves | Explicit transcript |
| Scoring uses executed tool, not model words | Explicit transcript |
| Final twenty chats are opened once | Explicit transcript |
| Phase one secures parcel tool | Explicit transcript |
| Other user's parcel equals nonexistent result | Explicit transcript |
| Four fast tests added; four phase boxes ticked | Explicit transcript |
| Independent fast test run: 7 passed, 3 live omitted | Explicit transcript |
| Tracked diff summary: four files changed | Explicit transcript |
| Plan file untracked, absent from Git diff summary | Explicit transcript |
| Hook passes Ruff, mypy, fast tests; widget skipped | Explicit transcript |
| Commit saved on build-one | Explicit transcript |
| Exact eight phase headings and paths | Not given |
| Exact test function names and file diff | Not given |
| Specific tag name or commit SHA | Not given |
| Illustrative authorization function and test matrix | Additional engineering explanation |
| Later phases successfully completed | Not established in this lesson |

---

# Part 21: FDE Interview Questions and Answers

## 61. Main FDE Interview Question

**Q: How do you use an AI coding agent to build a secure enterprise prototype without losing control of scope and quality?**

**Strong answer:**

> “I give the agent a reviewed two-voice spec with traceable acceptance tests, persistent project rules and enforced permissions. I work on a dedicated branch and use Plan Mode to inspect the code before any edits. The agent proposes small phases with runnable checks, and I resolve important decisions about verification, outcome scoring and commit approval. Then I allow only one phase. After the agent reports completion, I open the plan file, run the fast tests independently, inspect the diff and Git status, and commit only accepted work. The hook enforces Ruff, mypy and fast tests. In DemoStore, this process caught an unprotected parcel-status tool and direct model calls bypassing the adapter; phase one fixed parcel ownership before OTP and handoff features were built.”

## 62. Detailed Interview Q&A

**Q1. What works before Build 1 starts?**

The chat service and widget. The full assistant prototype remains incomplete.

**Q2. What does the engineer do first?**

Review the spec, `CLAUDE.md`, settings and code, and create an independent branch.

**Q3. What is the branch name?**

`build-one`.

**Q4. Why work on a branch?**

To isolate the build and preserve a reviewable set of phase commits.

**Q5. Why does the course repository have an end-of-day tag?**

Learners with different coding-agent results can resume from the course's known checkpoint.

**Q6. What is the actual tag name?**

Not supplied in the transcript.

**Q7. What business accuracy target is in the spec?**

At least 36 correct actions out of 40 labeled conversations.

**Q8. What is the security business rule repeated?**

An unverified customer sees no protected order data.

**Q9. What do IT lines in the spec reference?**

The business requirement each technical line serves.

**Q10. What does the end of the spec contain?**

Out-of-scope items and never-do rules.

**Q11. What does `CLAUDE.md` specify?**

Fast-test command, centralized model adapter, refund policy and Ledgerline limits.

**Q12. Which actions run without manual permission?**

Approved tests and local commits.

**Q13. Which actions need approval?**

Package installation and pushing to the remote repository.

**Q14. Which actions are denied?**

Reading the secrets file and bypassing the pre-commit hook.

**Q15. What does Plan Mode do?**

Reads, investigates and asks questions without editing files.

**Q16. What is required at the end of each phase?**

A runnable check, such as fast tests or replay of the forty chats.

**Q17. How is the plan represented?**

As a repository file with a checkbox for each task.

**Q18. How many phases does the new Build 1 plan contain?**

Eight.

**Q19. How is that different from the earlier chat-widget plan?**

The earlier chat service/widget plan had seven phases; Build 1 is a new eight-phase plan.

**Q20. What two code gaps did the agent find?**

Parcel status was exposed without verification, and the agent loop called Claude directly instead of using the adapter.

**Q21. Why is an open parcel-status tool dangerous?**

It can disclose shipment information for other customers even if order lookup has ownership checks.

**Q22. What is the correct source of customer identity?**

The trusted verified application session.

**Q23. What is the model adapter rule?**

Every Claude request must use the shared adapter with one central model-name constant.

**Q24. What email-code decision was approved?**

The customer enters her email in the widget and the code is sent to that address.

**Q25. What defines a correct action for evaluation?**

The tool that actually executed, not the model's wording.

**Q26. Why is model wording unreliable for action scoring?**

The assistant can claim it performed an action even when no corresponding tool ran.

**Q27. What was the phase-commit decision?**

Commit one phase at a time only when the engineer requests it.

**Q28. What do later planned phases include?**

OTP, verification in service and widget, human handoff, forty-chat replay, iterative prompt updates and build closure.

**Q29. How should prompt improvements be evaluated?**

One change at a time against the labeled development cases.

**Q30. How many held-out chats are reserved?**

Twenty.

**Q31. When are they opened?**

Once, at the end of the build.

**Q32. What is Auto Mode allowed to do?**

Routine reading and writing in the folder while enforced permission restrictions remain active.

**Q33. How many phases does the agent implement in this lesson?**

Only phase one, then it stops.

**Q34. What does phase one fix?**

The parcel/delivery-status tool's missing verification and ownership protection.

**Q35. How should a foreign parcel and nonexistent parcel appear?**

The same “no such parcel” response.

**Q36. How many new fast tests are added?**

Four.

**Q37. How many phase-one checkboxes are ticked?**

Four.

**Q38. What does the independent test run report?**

Seven fast tests passed; three live Claude tests omitted.

**Q39. How do you run shell commands from the demonstrated Claude Code interaction?**

Prefix the command with `!`, as shown in the lesson.

**Q40. How many tracked files show in the short diff summary?**

Four.

**Q41. Why does the new plan file not appear in the diff yet?**

It is untracked and `git diff` does not include it in the ordinary tracked-file view.

**Q42. Which hook checks pass at commit time?**

Ruff, mypy and fast tests.

**Q43. Why is the widget check skipped?**

No widget file changed.

**Q44. Where is the checkpoint saved?**

In a Git commit on branch `build-one`.

**Q45. What does the lesson prove about planning?**

The agent found two important existing-code gaps **before coding**, allowing the build to prioritize security and architecture.

**Q46. Does the transcript prove OTP delivery is implemented?**

No; it is part of the planned later phases.

**Q47. Does the transcript prove real ServiceNow handoff works?**

No; it is future planned work.

**Q48. Was the forty-chat target achieved during phase one?**

Not established. The lesson reports phase-one fast tests and hook checks, not final evaluation metrics.

**Q49. What do you inspect after an agent says it finished?**

The saved plan file, independent test output, changed files, and Git tracking state.

**Q50. What is the central practical rule?**

**Let the agent plan before it codes, read the plan as carefully as code, and accept a phase only after personally running its check.**

---

# Part 22: Quick Knowledge Check

## 63. Six Self-Test Questions

<details>
<summary><strong>1. Which statement correctly describes the new plan?</strong></summary>

**A.** The earlier seven-phase widget plan is rerun with no changes.

**B.** Build 1 uses a new **eight-phase** plan, with only phase one implemented in this lesson.

**C.** All eight phases are completed and deployed.

**Correct: B.**

</details>

<details>
<summary><strong>2. What security defect did phase one repair?</strong></summary>

**A.** The widget lacked dark mode.

**B.** The parcel tool returned statuses without proper verification or ownership.

**C.** The fast tests used too many terminal windows.

**Correct: B.** The corrected tool hides data from unverified visitors and treats foreign parcels like nonexistent ones.

</details>

<details>
<summary><strong>3. What determines the correct action in the approved evaluation decision?</strong></summary>

**A.** The wording Claude uses to say what it did.

**B.** The actual tool that ran.

**C.** The length of the reply.

**Correct: B.** An action is evidenced by execution, not fluent claims.

</details>

<details>
<summary><strong>4. Why save the twenty held-out conversations for the end?</strong></summary>

**A.** To tune repeatedly on their results.

**B.** To keep a once-only final check separate from the forty development conversations.

**C.** Because those conversations do not contain labels.

**Correct: B.**

</details>

<details>
<summary><strong>5. Why wasn't the plan file included in the four-file diff summary?</strong></summary>

**A.** It was already deleted.

**B.** It was an untracked new file; ordinary `git diff` did not include it.

**C.** It belonged to a different GitHub organization.

**Correct: B.**

</details>

<details>
<summary><strong>6. Which evidence supports accepting phase one?</strong></summary>

**A.** Only Claude Code's success report.

**B.** A checked plan, independently run fast tests, reviewed diff and successful hook-backed commit.

**C.** A promise that all later phases will pass.

**Correct: B.**

</details>

---

# Part 23: Complete Revision Checklist

## 64. Context and Inputs

- [ ] Chat service and widget exist, but full prototype is larger.
- [ ] Code agent writes; human checks every phase.
- [ ] Spec defines business and IT requirements with links.
- [ ] Unverified customer must see no order data.
- [ ] Development goal: **36/40 correct actions**.
- [ ] Spec also states out-of-scope and never-do rules.
- [ ] `CLAUDE.md` has fast tests, adapter, refunds and Ledgerline rules.
- [ ] Settings allow **tests and commits**.
- [ ] Settings ask for **package installs and push**.
- [ ] Settings deny **secrets-file reads and hook bypass**.

## 65. Branch and Planning

- [ ] Create branch **`build-one`** before code edits.
- [ ] Course has an **end-of-day tag**, but its name is not in transcript.
- [ ] Start Claude Code in **Plan Mode**.
- [ ] Plan Mode reads/asks but **changes nothing**.
- [ ] Ask for **small phases** with runnable checks.
- [ ] Checks are fast tests or replay of **40 chats**.
- [ ] Save plan in a file with **task checkboxes**.
- [ ] Approve the plan before implementation.
- [ ] Plan contains **8 phases**, not the previous 7.
- [ ] Agent finds **2 pre-existing code gaps**.

## 66. Two Discovered Gaps

- [ ] Parcel status was returned to **unverified visitors**.
- [ ] Parcel ownership was not adequately checked.
- [ ] Agent loop called Claude **directly**.
- [ ] All model calls should go through the **central adapter**.
- [ ] Phase one fixes **parcel tool security**; later phases address remaining work.

## 67. The Three Decisions

- [ ] Customer enters **email in widget**.
- [ ] OTP goes to **that email address**.
- [ ] Correct action scored by **tool that ran**, not model words.
- [ ] Commit **one phase at a time** only when asked.
- [ ] Human approves all three.

## 68. Future Phases

- [ ] Add one-time-code generation and delivery.
- [ ] Check code in service and widget.
- [ ] Implement actual handoff to a person.
- [ ] Replay **40 labeled chats**.
- [ ] Change prompt **one item at a time**.
- [ ] Reach the development target before final evaluation.
- [ ] Open **20 held-out chats once at the end**.
- [ ] The transcript does **not** show these later phases completed.

## 69. Phase One Implementation

- [ ] Enter Auto Mode only after plan approval.
- [ ] Settings restrictions remain active.
- [ ] Instruct agent: **first phase only, then stop**.
- [ ] Unverified visitor receives **no parcel data**.
- [ ] Foreign parcel and nonexistent parcel return same **“no such parcel”**.
- [ ] Agent adds **4 fast tests**.
- [ ] Agent ticks **4 plan boxes**.
- [ ] Agent stops after phase one.

## 70. Independent Verification and Git

- [ ] Agent's “tests passed” report is **only text**.
- [ ] Open plan file and confirm **all 4 phase-one boxes ticked**.
- [ ] Run command in Claude Code using **`!` prefix**.
- [ ] Independent fast tests: **7 passed**.
- [ ] **3 live tests excluded**.
- [ ] Diff summary shows **4 tracked files changed**.
- [ ] Plan file absent from normal diff because **untracked**.
- [ ] Check Git tracking state, not only diff.
- [ ] Commit reviewed phase on **`build-one`**.
- [ ] Hook: **ruff passed**.
- [ ] Hook: **mypy passed**.
- [ ] Hook: **fast tests passed**.
- [ ] Widget check **skipped**, because no widget file changed.
- [ ] Commit creates a known-good checkpoint.

## 71. FDE Principles

- [ ] Plan Mode is also an **architecture and security audit**.
- [ ] Protect **every** tool that reveals protected data.
- [ ] Verified identity must come from **trusted session state**.
- [ ] Evaluate **executed actions**, not assistant promises.
- [ ] Keep **held-out data** separate from repeated prompt tuning.
- [ ] Read the **plan as carefully as code**.
- [ ] Validate files and tests **yourself**.
- [ ] Keep scope to **one phase** per review checkpoint.
- [ ] Persist tested changes in Git before moving on.

---

# Part 24: Final Summary and Practical Assignment

## 72. Lesson Summary

| Topic | Exact takeaway |
|---|---|
| Existing state | Chat service and widget work |
| New scope | Full assistant prototype, Build 1 |
| Branch | **build-one** |
| Plan mode | Read and ask; no editing |
| New plan | **8 phases** |
| Existing gaps | Open parcel status; direct Claude calls |
| OTP decision | Email entered in widget; code sent to it |
| Scoring decision | Actual tool execution |
| Commit decision | One phase only when asked |
| Development dataset | **40 labeled chats** |
| Target | **36/40 correct actions** |
| Held-out dataset | **20 chats, opened once at end** |
| Phase 1 | Parcel verification and ownership |
| Added tests | **4** |
| Ticked phase-one plan tasks | **4** |
| Independent fast tests | **7 passed** |
| Live tests left out | **3** |
| Diff | **4 tracked files changed** |
| New plan file | Untracked, absent from ordinary diff |
| Git hook | Ruff + mypy + fast tests pass |
| Widget hook | Skipped, no widget change |
| Final evidence | Local commit on **build-one** |
| Next stage | Subsequent phases, one fresh session each |

## 73. FDE Interview Summary

A strong engineer should be able to explain how planning uncovered two real flaws before implementation, why order and parcel data require authorization in each tool, how centralizing model calls avoids drift, how to score executed actions, how held-out evaluation avoids misleading results, and how to audit a phase by checking **file contents, real tests, Git status, hooks and commits**.

## 74. Practice Assignment

**Give the same phased-planning prompt to your coding agent on your own copy of the course repository. Compare its plan against the instructor's plan. Did it find the same gaps?**

Your exercise:

1. Create a dedicated **Build 1 branch**.
2. Ensure the spec, `CLAUDE.md`, permissions and hooks are in place.
3. Use **Plan Mode**, with no editing.
4. Ask for a small phased plan with executable checks and task checkboxes.
5. Inspect whether it finds **parcel authorization** and **direct provider calls**.
6. Review and answer architecture/evaluation/commit questions.
7. Approve **phase one only**.
8. Inspect the resulting plan checkboxes, run fast tests and check both diff and Git status.
9. Accept a checkpoint only after actual verification.

> **Practical rule: Let the agent plan before it codes, read the plan as carefully as code, and accept a phase only after you have run its check yourself.**

**Next lesson:** Claude Code works through the remaining Build 1 phases, **one at a time in fresh sessions**, with checks and reviewed checkpoints.
