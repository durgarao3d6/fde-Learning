# Complete Study Notes: Reviewing and Running DemoStore's Chat Service and Widget

**FDE Course · Claude Code · Seven-Phase Build · Git History · FastAPI · React/TypeScript · Code Review · Live Tests · Human Handoff · Browser Testing**

> **Central practical rule:** **Test what the customer can do, not only what the code returns. Check every report from the coding agent against the files it names.**

This lesson continues directly from [Chapter 11 — Planning the Chat Service and Widget](11-chat-service-widget.md). The previous lesson inspected **phase one** of a seven-phase Claude Code plan. This lesson looks at the result after the remaining six phases, independently checks the service and widget, finds a real user-experience bug, fixes it, and tests the application in the browser.

**Source-fidelity note:** The facts, counts, sequence, file names and actual observed behavior below come from the supplied lesson transcript. Example code and proposed extra tests are marked **illustrative** because the transcript does not show the exact source implementation or test commands.

---

# Part 1: What Is This Lesson About?

## 1. From the Plan to the Finished Internal Build

The earlier brief called for a **FastAPI chat service**, an **AI disclosure**, one shared **Pydantic intent class**, a **React/TypeScript widget**, and **opt-in live tests calling Claude**.

In the previous lesson, the team watched Claude Code implement **phase one**: replacing the handwritten intent schema with a Pydantic class and adding the `language` field. The engineer ran **four fast tests**, inspected `intent.py`, and committed that phase.

Now the narrator explains that **Claude Code completed phases two through seven**:

- Each phase was done in a **fresh Claude Code session**.
- After each phase, the engineer ran the **fast tests**.
- Once the frontend widget existed, they **also built the widget after each phase** to ensure it still compiled.
- After verification, they made a **Git commit**.

This lesson does **not** show the six implementation sessions in full. It inspects the resulting artifacts and performs a real usage test.

### The three checks in today's plan

1. **Review history and tests**: verify the plan file, commits, fast tests and live tests.
2. **Review code ourselves**: inspect the FastAPI service and the React widget, not just the agent's summary.
3. **Run a first browser chat**: verify the actual experience, especially the path to a person while the assistant is responding.

**Key concept:** Code produced by an agent is not automatically accepted software. The developer needs evidence that the requested behavior exists and that users can actually perform the critical actions.

---

# Part 2: Reconstruct the Seven-Phase Plan

## 2. The Phases from the Previous Lesson

| Phase | Work | State by the beginning of this lesson |
|---|---|---|
| **1** | One Pydantic intent class and generated schema | Completed in the previous lesson |
| **2** | Conversation history support | Completed |
| **3** | FastAPI chat service | Completed |
| **4** | Opt-in tests making real Claude calls | Completed |
| **5** | First widget implementation | Completed |
| **6** | Additional widget work | Completed |
| **7** | Short run guide / README instructions | Completed |

The transcript states that the other six phases were built **one at a time, each in a fresh session**. It does not detail the code written within each phase or show the exact per-phase test logs.

### Why small phases still matter now

Because the work was split:
- A failure is easier to associate with a specific phase.
- Each step had its own validation opportunity.
- Git contains checkpoints that can be inspected.
- The review focuses on real product behavior instead of an enormous unexplained diff.

## 3. What Git Log Shows

The engineer checks `git log` and sees:
- A commit for the **brief**.
- A commit for **phase one**.
- A commit for **each later phase**.
- **One extra commit between phases one and two** containing the plan written again.

That extra commit reveals a failure that would have been easy to miss.

**Do not infer exact commit hashes, branch names or messages:** the transcript does not give them.

---

# Part 3: The Failed Plan-File Copy

## 4. What Claude Code Reported

After phase one, Claude Code said that it had **saved the plan into its file**.

The team relied on this report initially.

When the engineer opened the actual plan file, it contained only:

> Its title.

The rest of the seven-phase plan had **not been copied into the file**.

The copying operation had failed, and Claude Code's completion report **did not mention that failure**.

### Why there was another Git commit

The plan needed to be written to the file again, so a separate commit appeared **between phases one and two**.

This is not evidence of a mysterious eighth implementation phase. It is evidence of a **repair to an incomplete artifact**.

## 5. Claude Code Acknowledged the Mistake

The transcript says Claude Code also recognized what should have happened:

**It should have inspected the file before committing.**

This is one of the most important lessons because it shows that **agent self-reporting can be inaccurate even when the agent sounds confident**.

### Three separate claims

| Claim | Evidence needed |
|---|---|
| “I planned seven phases.” | Read the actual plan text |
| “I saved the plan.” | Open the plan file and check its contents |
| “I committed the plan.” | Inspect Git history and the committed file |

A commit proves **something was recorded**. It does **not** prove the content was complete.

### The precise principle

> **A report is only text. The file is the proof.**

### Illustrative independent checks

```bash
# These commands are examples; the transcript gives no exact commands.
git log --oneline
git show --stat HEAD
git show HEAD:path/to/plan.md
```

For a plan-file claim, inspect the **file contents**, not just the commit subject.

---

# Part 4: Fast Tests and Live Tests

## 6. Fast-Test Results

After the seven phases, the team runs the **fast tests**.

<details>
<summary><strong>Actual result: 13 fast tests pass</strong></summary>

The transcript reports **13 passed**. These are the fast checks suitable for routine execution. We are not given their names or assertion code.

</details>

## 7. Three Live Tests Are Excluded from the Fast Run

The fast suite leaves **three tests out**.

Why?

Because these are the **live tests** that really call Claude.

Live model calls:
- Need working external API access.
- Can cost money.
- Depend on network/provider availability.
- May be slower or less deterministic than local tests.

The team runs them **deliberately**, not as part of every quick edit-and-test cycle.

### Actual live-test result

**All three live tests pass.**

The video describes what their behavior demonstrates:

1. **Conversation history works**: the service remembers an order number supplied in an earlier message.
2. **Disclosure is first-turn-only**: the first reply says the user is talking to AI, without repeating the notice on every reply.
3. **Structured intent reaches the response**: the reply includes intent and its **language**.

### Do not conflate the numbers

| Test category | Reported result |
|---|---|
| Fast tests | **13 passed** |
| Live tests omitted from fast run | **3** |
| Live tests run intentionally | **3 passed** |

The source does not say the suite has only these 16 tests under every configuration, nor does it provide exact test function names.

## 8. Why the Results Matter

### History is now working

A customer may mention an order number in turn one and ask a follow-up without repeating that number. Earlier, `agent.answer` did not accept history. The service now has a working conversation context path.

### AI notice appears only once

A notice repeated at the start of every assistant answer would interrupt the conversation. The service deliberately injects it only into the **first reply**.

### Intent includes language

The Pydantic class introduced in phase one is used throughout the system. The **`language`** field reaches the structured response without maintaining independent schema copies.

### What these tests do *not* prove

They do not guarantee that the “Talk to a person” **button** can be clicked while loading. That is a frontend interaction behavior, not necessarily covered by service tests.

---

# Part 5: Review the FastAPI Chat Service

## 9. Open `app.py`

The team begins its own code review with **`app.py`**, which contains the FastAPI chat service.

The narrator describes one endpoint as **“one door”**:

- A customer message enters.
- An assistant reply returns.
- The response also contains a **session ID**.
- And the response carries the **intent**.

**The exact URL path and source code are not shown**, so do not claim the endpoint is necessarily `/chat` unless the actual repository verifies it.

## 10. What the Session ID Does

The **session ID** allows subsequent messages to be associated with the same conversation.

Example flow (illustrative):

```text
Message 1: "Where is order DEMO-100?"
         ↓
FastAPI creates/uses a session
         ↓
Reply + session_id="session-1"
         ↓
Message 2: "Has it shipped?"
         ↓
FastAPI looks up history for "session-1"
         ↓
Assistant can interpret the follow-up
```

The earlier brief specified **in-memory sessions**, so a service restart can discard them. Nothing in this lesson announces that persistent storage was added.

### A session ID is not customer verification

Even with a valid session ID, the order tools must require a separately verified customer. The upcoming browser chat shows that identity verification is still incomplete.

## 11. Reuse One Pydantic Intent Class Everywhere

The lesson revisits the architectural decision from phase one.

**The same intent class serves three consumers:**

1. **Claude's answer** in the required structured shape.
2. **Backend response to the widget**.
3. **FastAPI-generated documentation page**.

Because the source class includes **`language`**, that field reaches all three. The team does not need to remember to update multiple handwritten dictionaries.

### Conceptual architecture

```text
                       Shared Pydantic intent model
                                 │
             ┌───────────────────┼─────────────────────┐
             ↓                   ↓                     ↓
       Claude schema       Widget API response    FastAPI docs
             │                   │                     │
       intent + language   intent + language     intent + language
```

**Engineering significance:** The single-source-of-truth pattern prevents schema drift between model output, API validation and documentation.

## 12. AI Notice Is Prepended Only to the First Reply

The code in `app.py` adds the AI notice **in front of the assistant's first reply only**.

There is an easy-to-miss detail:

> **The notice is not saved in the conversation history.**

Therefore, **Claude does not see the AI notice as part of the message history**.

### Why is this distinction useful?

The application owns the disclosure behavior rather than depending on the LLM to reproduce it. The transcript states this implementation outcome; it does not claim that the model never sees any other disclosure instruction in its system prompt.

### Two separate representations

| Customer-facing response | Internal conversation history |
|---|---|
| Includes the AI notice on first reply | Does **not** store the prepended notice |
| Shows the assistant's actual reply | Preserves conversational content needed for future turns |
| First-turn disclosure appears once | No repeated notice accumulating as conversation content |

This is a product design decision: a required interface disclosure need not become part of the model's conversation state.

### Illustrative logic — not transcript source code

```python
reply = agent_reply

if is_first_reply:
    visible_reply = AI_NOTICE + "\n\n" + reply
else:
    visible_reply = reply

# Save the conversation content according to the app's history design;
# do not automatically treat the separately prepended notice as dialogue.
```

The actual code must determine “first reply” consistently within its session logic.

---

# Part 6: Review the React Widget

## 13. Open `widget/Chat.tsx`

The engineer opens the widget's **`Chat.tsx`** file inside the **widget folder**.

This is the **React + TypeScript** component responsible for the visible chat controls.

The reviewer looks for the requirement stated in the brief:

> **The “Talk to a person” button must remain available at all times.**

This requirement is not decorative; it addresses customers' fear of being trapped behind an automated system.

## 14. The Bug: Human-Support Button Disabled During Loading

The first implementation has **two interacting problems**:

1. The **“Talk to a person” button is disabled while an assistant reply is loading**.
2. The **send function returns immediately** when the widget is waiting for an in-flight reply.

The combined effect is worse than a cosmetic issue.

### The failure sequence

```text
Customer sends an order question
            ↓
Widget starts loading
            ↓
"Talk to a person" button is disabled
            ↓
Send function refuses another message
            ↓
Customer cannot request human support
            ↓
The exact moment a person is most needed is blocked
```

### Why this violates the brief

The brief requires the button to be **always visible and usable**. It says customers particularly fear being unable to reach a human when automation stalls.

The implementation preserved a loading state but unintentionally allowed that state to **block an essential escape action**.

### Important nuance

A button being **visible** is not enough if it is **disabled**. Likewise, enabling the button is not enough if its click handler calls a send function that immediately returns.

Both the UI control and its underlying action must be reviewed.

## 15. Why the Tests Did Not Find the Bug

The lesson explicitly explains that the tests focus on the **service**.

They do **not** test the button's browser behavior.

Consequently:
- **13 fast tests can pass.**
- **3 live model tests can pass.**
- The build can compile.
- **The customer can still be unable to request a person while waiting.**

This is the central technical and FDE lesson:

**A passing backend test suite does not prove that a customer can use the actual frontend.**

### Testing layers

| Test | What it can demonstrate | What it may miss |
|---|---|---|
| Service unit tests | Session rules, response shapes, backend logic | Disabled UI controls |
| Live Claude tests | Real model integration and response data | Button interaction during latency |
| Frontend build | Code compiles | Behavioral accessibility of a button |
| Browser interaction test | Can user click a specific control while waiting? | Depends on test coverage |
| Full human workflow test | Can a person actually take over? | Not possible before handoff exists |

---

# Part 7: Request a Targeted Fix

## 16. What the Engineer Tells Claude Code

The team does not vaguely ask “improve the widget.”

It provides:
- **Observed problem:** The person button becomes disabled and send refuses a new message while loading.
- **Desired behavior:** **The button must work at all times.**

This is a much better coding-agent instruction because it defines an externally observable acceptance condition.

### Reusable fix prompt (adapted from the lesson)

> In `widget/Chat.tsx`, the “Talk to a person” button is disabled while a reply is loading, and the send handler returns when loading is true. This violates the requirement that customers can request a person at any time. Change the widget so that button remains functional during an in-flight assistant response. Preserve a correct typing indicator for overlapping requests, run the widget build, report remaining edge cases, and stop for review.

The prompt above is a practical reconstruction, **not** a verbatim transcript.

## 17. What Claude Code Changes

The updated widget:

1. **No longer disables the person button** while a reply is loading.
2. **Allows another message to be sent** while a reply is still pending.
3. **Counts pending replies** rather than using a simple on/off loading state.
4. Keeps the **typing indicator visible until the last pending reply is complete**.

The transcript describes these behaviors but does not provide the implementation source code.

---

# Part 8: Why a Pending-Message Counter Matters

## 18. The Concurrency Problem

A single boolean `loading` may work when only one message can be pending. Once overlapping requests are permitted, it becomes insufficient.

Example:

```text
T1: Customer sends "Where is my order?"
    → pending replies = 1
    → typing indicator ON

T2: Customer presses "Talk to a person"
    → pending replies = 2
    → typing indicator remains ON

T3: First reply arrives
    → pending replies = 1
    → typing indicator remains ON

T4: Second reply arrives
    → pending replies = 0
    → typing indicator OFF
```

If the widget simply sets `loading = false` when **the first** reply arrives, the indicator turns off even though a second response is pending.

The counter fixes that distinction.

### Illustrative implementation

```tsx
const [pendingReplies, setPendingReplies] = useState(0);

async function sendMessage(message: string) {
  setPendingReplies(count => count + 1);

  try {
    await sendToChatService(message);
  } finally {
    setPendingReplies(count => Math.max(0, count - 1));
  }
}

const showTypingIndicator = pendingReplies > 0;
```

This is **explanatory TypeScript**, not a copy of the file. A production implementation must also consider ordering, errors and session creation.

### Why functional state updates matter

When requests complete in any order, updates such as `setPendingReplies(count => count - 1)` avoid the stale-state problem of reading an old captured value.

**This is an engineering explanation**, not something the transcript explicitly discusses.

## 19. Completion Order Is Not Necessarily Send Order

Once overlapping messages are allowed, a response to message two might arrive before the response to message one.

The transcript **does not say** how the app handles out-of-order replies. That remains a separate concern for later review.

Possible additional safeguards (not claimed as implemented):
- Identify each outgoing request.
- Match each reply to the right user message.
- Preserve correct conversation ordering.
- Avoid mixing session identifiers.
- Handle failures without leaving the counter above zero.

Do not mistake the reported counter fix for proof that all concurrent-chat edge cases have been solved.

---

# Part 9: An Unresolved Session-ID Race

## 20. Claude Code Names a Remaining Gap

After making the person button work during loading, the agent flags an important unresolved edge case.

**The widget receives its session ID with the first assistant reply.**

That means it might not have a session ID while the first request is still waiting.

### The problematic sequence

```text
Customer sends first message
         ↓
Widget has no session ID yet
         ↓
First assistant response is still pending
         ↓
Customer presses "Talk to a person"
         ↓
Widget sends another request with no session ID
         ↓
Service starts a NEW conversation
         ↓
Messages are now split across two sessions
```

This is a **session-creation race condition**, not merely a loading indicator issue.

## 21. Why the Gap Matters to a Future Human Agent

Suppose the customer originally writes:

> "My order is delayed, and I need help urgently."

They press **“Talk to a person”** before the first reply arrives.

If the second request creates a **different session**, then the future human-support workflow may be attached to a conversation missing the **first message**.

The narrator explains the consequence:

**The person who takes over would not see the original customer message.**

This is a context-continuity problem.

### The transcript's actual decision

The team **does not implement the complete fix here**.

Real human handoff belongs to a later build. Instead, the engineer asks Claude Code to **record this open issue in the plan**.

**Important:** “A gap was documented” does not mean “the gap was solved.”

## 22. Possible Future Designs — Illustrative Only

The lesson does not prescribe a solution. A future team could evaluate:

- Creating a session **before** sending the first chat message.
- Returning a session ID immediately from a separate initialization call.
- Generating a client-side conversation ID that the backend validates.
- Queueing a second request until initial session creation is confirmed.

Each option has trade-offs. None should be attributed to the video as implemented behavior.

The correct FDE habit is to **make the limitation visible, put it in the plan, and protect the later handoff design from losing conversation context**.

---

# Part 10: Verify the Widget Fix Yourself

## 23. Run the Widget Build

After Claude Code changes the UI, the team independently builds the widget.

**The build passes.**

The source does not specify the exact command, package manager or build output.

### What a successful build establishes

It can establish that:
- The React/TypeScript code compiles through the configured build pipeline.
- Imports and type checks required by that build succeed.
- The updated widget is syntactically and structurally compatible with its tooling.

### What the build does not establish

A compile result alone cannot establish:
- That the person button works during loading.
- That messages reach the server under every race condition.
- That a real ServiceNow case gets created.
- That handoff preserves the first message.
- That the UI correctly handles every failure path.

The browser test is still essential.

## 24. Commit the Fix

After independently confirming the widget build, the engineer commits the change.

The lesson's completed review loop is:

```text
Identify observable bug
       ↓
Give precise instruction to coding agent
       ↓
Coding agent changes files and reports remaining gap
       ↓
Engineer inspects / builds
       ↓
Engineer commits reviewed fix
```

This illustrates **small, targeted corrections**, not a broad rewrite.

---

# Part 11: Run the Service and Widget Locally

## 25. The README Is the Run Guide

The seventh planned phase created a **README** with startup instructions.

The team opens it and finds a section titled:

> **Run the chat**

The transcript says this section explains how to launch the application.

### Two processes are required

- **Terminal one:** Start the **FastAPI service**.
- **Terminal two:** Start the **React/TypeScript widget**.

The narrator says both were already started before recording the demonstration.

### Why two terminals?

The backend receives chat HTTP requests and runs the assistant logic. The frontend runs a local development web server and displays the widget.

These are **separate programs**.

## 26. Open the VS Code Browser

The user opens the widget in the browser **built into VS Code**.

The address is in the README's **Run the chat** section.

The narrator explains:

- **`localhost`** means the local computer.
- The widget uses **port 5173** (spoken “five one seven three”).

Illustrative address:

```text
http://localhost:5173
```

This is the frontend development address described by the transcript. The exact backend startup command and port are **not stated** in the supplied transcript.

**Don't confuse DemoStore's running application with the FDE study-notes website hosted on Vercel.** This lesson reviews the app built inside the DemoStore course repository, not the documentation site you're currently reading.

---

# Part 12: First Browser Chat — The Actual Demonstration

## 27. Customer Sends an Order Question

A customer asks **where her order is**.

The first reply begins with the **AI notice**:

- She is talking to an AI assistant.
- She may ask for a person at any time.

This confirms that the customer-visible first-turn disclosure is present.

## 28. The Assistant Requests an Email One-Time Code

The assistant asks for the **one-time code from her email**, because the application has **not verified her yet**.

The lesson makes an important implementation boundary clear:

> **Checking the code comes later.**

In the prior interview-brief lesson, verification was deferred. This browser run remains an **internal incomplete build**, not a functioning end-to-end authenticated order lookup.

The assistant must not return private order details merely because a user asks.

### What the test shows versus doesn't show

| Observed | Not yet demonstrated |
|---|---|
| First-turn AI disclosure appears | Complete identity-verification flow |
| Assistant asks about verification | An email code is actually sent and validated |
| Order number history works in live tests | Protected order lookup for real customers |

The transcript explicitly says verification is not complete; it does not give a working OTP mail-service integration.

## 29. Press the Person Button While Loading

The customer asks again.

**Before the next assistant reply arrives**, she presses:

> **Talk to a person**

The button **works**.

That is the browser-level check the engineer intended after finding the disabled-button problem.

### Why this test is different from normal chat testing

A button could work perfectly when no model request is active. The requirement is stronger: it must work when the assistant is slow, stalled or still preparing a reply.

Therefore, the correct test must deliberately click it **during an in-flight request**.

---

# Part 13: The Response Is Honest but the Handoff Is Missing

## 30. Read the Assistant's Answer

After the human-support request, the assistant says **it cannot transfer the customer to a person**.

This might seem like a failure to someone watching the demo, but the video separates two issues.

### What works

- The button is clickable during loading.
- The widget allows the second message.
- The assistant processes it.
- The assistant **truthfully acknowledges the system's current limitation**.

### What remains unfinished

- **A real transfer to a human has not been implemented.**
- The next build must add that human handoff.

### Why honesty matters

Pretending a case has been opened would deceive the customer.

A message such as “You're now connected to support” would be wrong if there is no ServiceNow case or live support queue.

The proper FDE conclusion is not “the chatbot is production-ready.” It is:

**The widget escape action works, but the product still lacks real human support integration.**

### Product readiness distinction

| Capability | Observed status |
|---|---|
| Human-support control visible | Present |
| Control clickable during pending response | Fixed and verified |
| Additional message allowed during loading | Fixed |
| Loading indicator tracks pending replies | Implemented |
| Actual human transfer | **Not built** |
| Session-continuity edge case | **Documented, not fixed** |

---

# Part 14: Check the Generated FastAPI Documentation

## 31. Open the Service's `/docs` Page

Finally, the team opens the **service address** with **`docs`** appended to its path.

FastAPI creates an API documentation page automatically from the application definitions.

An illustrative address:

```text
http://localhost:<backend-port>/docs
```

The transcript does not state the backend port, so **do not substitute port 5173** here: **5173 is the widget/frontend port**.

## 32. What the Documentation Displays

The page describes shared request/response models, including:

- The **chat request**.
- The **chat response**.
- The **intent model**.

This confirms the architecture from the earlier lessons: the same classes that define backend data also drive generated documentation.

### Why this matters to integration teams

A typed API contract helps frontend developers and integrators understand what the backend expects and returns.

When the shared intent model gains `language`, the generated documentation can reflect that field without a separate manual schema being rewritten.

### The three consumers of intent, revisited

```text
One intent class
      │
      ├── Claude structured output
      ├── Backend response sent to widget
      └── FastAPI documentation
```

The code review and docs-page inspection corroborate that design at different levels.

---

# Part 15: Two Things Human Review Found That Tests Missed

## 33. Finding One — The Plan File Was Nearly Empty

**Agent report:** The plan is saved.

**Actual artifact:** Only the title was in the plan file.

**How it was found:** An engineer **opened the actual file**.

**Fix:** The plan was written again and an additional commit was made between phases one and two.

**Lesson:** Inspect the claimed output artifact, not just the assistant's completion text or commit message.

## 34. Finding Two — A Waiting Customer Was Blocked

**Agent-created widget:** Disabled the human-support button while loading, and its send function rejected further sends.

**Service test result:** Tests passed.

**How it was found:** The engineer **read `Chat.tsx` and compared implementation against the customer-facing requirement**.

**Fix:** Enable the button during loading; allow overlapping sends; count pending replies to keep the typing indicator correct.

**Remaining gap:** If the first reply has not returned, no session ID exists yet, so a second request can create a separate conversation.

**Lesson:** A backend test cannot substitute for testing a user interaction.

## 35. Compare the Two Failures

| Failure | Source of false confidence | Actual evidence | Generalizable principle |
|---|---|---|---|
| Plan only contained title | Agent reported file saved | Open the plan file | Validate artifacts |
| Person button disabled | Fast/live tests and build passed | Inspect UI code and click while pending | Validate user actions |

Together these show why FDEs need **artifact verification** and **workflow verification**.

---

# Part 16: Implementation Ideas for Tests That Would Have Caught the Bug

## 36. Service Tests Are Necessary, Not Sufficient

The existing service tests already check conversation memory, disclosure and intent. They should remain.

But the new behavior requires tests that explicitly exercise the widget during loading.

### Example browser test design (illustrative)

```text
Given a newly opened chat widget
And a simulated backend response that is intentionally delayed

When the customer sends "Where is my order?"
Then the typing indicator should appear

When the customer clicks "Talk to a person"
Before the first response has completed

Then the control must be enabled
And a new outgoing message should be accepted

When the first response finishes
But the second remains pending

Then the typing indicator should remain visible

When both responses have finished
Then the typing indicator should disappear
```

These are **proposed acceptance conditions**. The transcript does **not** say the team added an automated browser testing framework during this lesson.

## 37. A Test for the Unresolved Session Gap

The team intentionally defers the session race, but can record how to validate a future solution.

### Proposed test (not yet passing in the lesson)

```text
Send first message with no session ID.
Before first response arrives, request a person.
After both requests finish, check they belong to one conversation.
When a human case is eventually created,
ensure the first customer message is present.
```

The last step depends on the **future human-handoff integration**. It cannot be proven end to end in the current version.

---

# Part 17: Technical Concepts Worth Remembering

## 38. Unit Tests vs Integration Tests vs End-to-End Testing

**Unit tests** isolate smaller application functions. They are fast and often use mocks.

**Live integration tests** verify a real external dependency—in this case, Claude.

**Build checks** verify the frontend can compile.

**Browser/end-to-end testing** verifies interactive paths from a user's point of view.

They are complementary. A successful result in one category does not imply success in every other category.

## 39. The First-Response Disclosure Pattern

An app-level notice is distinct from the assistant's generated conversation.

The implementation prepends the disclosure **only to the first customer-visible reply** and **does not store the notice in model history**.

Benefits include predictable UX and avoiding repeated first-turn copy in later context. These benefits are engineering interpretations; the source directly establishes the actual behavior.

## 40. Race Conditions in UI State

The two relevant asynchronous facts are:

- **Several requests can be pending at once**.
- **A session ID is not available until the initial reply**.

The pending-counter fix addresses the **indicator/UI interaction** part of concurrency.

It does not by itself fix **initial session identity**, **reply ordering** or **future human handoff continuity**.

An FDE should be able to distinguish those concerns.

## 41. Build Artifacts vs Operational Outcomes

A compile succeeds; a chat interface loads; the assistant answers. But does a customer accomplish the intended outcome?

The video demonstrates the difference:
- **Build succeeded**.
- **Live tests succeeded**.
- **Human-support button could finally be pressed**.
- **A real human could not yet take over**.

Operational completion is defined by the **customer's goal**, not merely by running code.

---

# Part 18: Key Files and Exact Facts

## 42. File Map

| File or component | Role in the lesson |
|---|---|
| Plan file (name not specified) | Records seven phases and completion; first copy contained only title |
| Git log | Shows the brief, phase commits and corrective plan commit |
| `app.py` | FastAPI chat service implementation |
| `Chat.tsx` in widget folder | React/TypeScript chat UI and loading/person-button behavior |
| `README` | Instructions under **Run the chat** |
| FastAPI `/docs` | Generated API docs for request, response and intent |
| `llm.py` (earlier) | Central Claude adapter |
| `intent.py` (earlier) | Shared Pydantic intent definition |

**Do not invent** an exact plan path, Git commit SHA, backend port, or frontend startup command. Those are not supplied by this transcript.

## 43. Exact Numbers and Sequence

| Fact | Value |
|---|---|
| Planned implementation phases | **7** |
| Phases completed after the first | **6** |
| Fresh coding-agent sessions for those phases | One per phase |
| Extra plan-fix commit | **1**, between phases one and two |
| Fast tests passed | **13** |
| Live tests skipped by fast run | **3** |
| Live tests passed when deliberately run | **3** |
| Frontend development port | **5173** |
| AI notice | First reply **only** |
| Intent model consumers | **3** |
| Corrected person-button behavior | Works during loading |
| Remaining handoff capability | Not yet implemented |

---

# Part 19: Facts from Transcript vs Engineering Extrapolations

## 44. Keep Source Claims and Examples Separate

| Claim | Status |
|---|---|
| Claude Code implemented the other six phases in fresh sessions | Explicit transcript |
| Fast tests were run after each phase | Explicit transcript |
| Widget builds were run after the widget existed | Explicit transcript |
| Plan file had only a title after an earlier claimed save | Explicit transcript |
| A separate plan-fix commit appeared between phases 1 and 2 | Explicit transcript |
| 13 fast tests passed | Explicit transcript |
| 3 live Claude tests passed | Explicit transcript |
| Order number remembered from first message | Explicit transcript |
| First AI notice not stored in history | Explicit transcript |
| `app.py` contains chat service code | Explicit transcript |
| `Chat.tsx` disabled person button during loading | Explicit transcript |
| Pending-reply counter was added | Explicit transcript |
| First-reply session ID race remains | Explicit transcript |
| Frontend runs on localhost port 5173 | Explicit transcript |
| Backend port is 8000 | **Not stated** |
| Human transfer works | **False at this stage** |
| Automated browser tests were added | **Not stated** |
| Out-of-order responses are fully handled | **Not stated** |
| Session-ID race is fixed | **False; explicitly deferred** |
| Example pending-counter code matches actual file | **Illustrative only** |

The habit of distinguishing **observed**, **reported**, **implemented** and **proposed** details is essential to reliable FDE work.

---

# Part 20: FDE Interview Questions and Answers

## 45. Main FDE Interview Question

**Q: How would you validate an AI coding agent's completed chat application before showing it to customers?**

**Strong answer:**

> “I would treat the agent's completion message as a claim, not proof. First, I would inspect the plan file and Git history to make sure all seven implementation phases are actually recorded. Then I'd independently run the fast tests and opt-in live model integration tests, and review the FastAPI service and the React widget against the business requirements. I'd specifically test user interactions under latency: the ‘Talk to a person’ button must work while a response is pending. If I find a violation, I'd give the agent a narrowly scoped fix, rebuild the widget, review the change and commit. Finally, I'd run the service and widget in the browser, validate the actual disclosure and handoff behavior, document unresolved session and verification gaps, and not call the system production-ready until they are addressed.”

## 46. Additional Interview Q&A

**Q1. What happened after phase one?**

Claude Code implemented the other **six phases**, each in a fresh session, with human tests and commits after every phase.

**Q2. How many phases were in the original plan?**

Seven.

**Q3. What unexpected commit appeared in Git history?**

An additional commit between phases one and two to rewrite the plan file.

**Q4. Why was an extra plan commit needed?**

The earlier supposed copy saved only the plan's **title**, even though Claude Code said the plan was saved.

**Q5. What should the agent have done before committing?**

Open/read the saved file and verify its contents.

**Q6. What is the lesson's principle about reports?**

**A report is only text; the file is the proof.**

**Q7. How many fast tests passed?**

Thirteen.

**Q8. How many live tests were omitted from the fast run?**

Three.

**Q9. How many live tests passed?**

All three.

**Q10. What do the live tests demonstrate?**

Remembering an order number across turns, first-reply AI disclosure, and response intent including language.

**Q11. Which file contains the FastAPI service?**

`app.py`.

**Q12. What three values leave the service after a message?**

Reply, session ID and intent.

**Q13. Why is a session ID needed?**

It connects follow-up messages to the existing conversation.

**Q14. Which three consumers share the intent model?**

Claude's answer, the widget API response and generated API documentation.

**Q15. What happens to the AI notice after it is displayed?**

It is prepended to the first visible reply but **not** saved in the conversation history.

**Q16. Which widget file was reviewed?**

`Chat.tsx` in the widget folder.

**Q17. What did the engineer find wrong with the person button?**

It was disabled while loading, and the send function refused a second message while a reply was pending.

**Q18. Why was that more than a cosmetic defect?**

It blocked an explicit customer requirement: the ability to request human assistance at any time.

**Q19. Why didn't the tests detect it?**

They tested the service, not the button's interactive behavior.

**Q20. What did the engineer ask Claude Code to fix?**

Keep “Talk to a person” working at all times, including during a pending assistant response.

**Q21. What changed in the widget?**

The button was enabled while loading and new messages could be sent before prior replies finished.

**Q22. Why add a count of pending replies?**

So the typing indicator stays visible until the **last outstanding reply** arrives.

**Q23. What remains wrong with initial session creation?**

The session ID arrives only with the first reply, so a second early request may start a different session.

**Q24. What might a future human-support agent lose because of this race?**

The customer's **original first message**.

**Q25. Was the session issue fixed in this lesson?**

No. It was noted in the plan for future work.

**Q26. What was verified before committing the widget fix?**

The engineer ran the **widget build**, and it compiled successfully.

**Q27. Where are startup instructions recorded?**

In the README, under **Run the chat**.

**Q28. How many terminals are needed in the demonstrated setup?**

Two: one for the service and one for the widget.

**Q29. Which browser was used?**

The browser built into VS Code.

**Q30. What is the widget development port?**

**5173**.

**Q31. What happens when the customer asks for an order update?**

The first response discloses AI and the ability to ask for a person, then the assistant requests an email one-time code because identity is not verified.

**Q32. Is one-time-code verification complete?**

No. The lesson says checking that code comes later.

**Q33. What did the customer test during the second reply?**

Pressing **“Talk to a person” while the reply was loading**.

**Q34. Did the button work?**

Yes, after the fix.

**Q35. Did the assistant transfer her to a real person?**

No. It honestly said it could not, because handoff was not implemented.

**Q36. What is the first feature in the next build?**

A **real human handoff**.

**Q37. How do you open FastAPI-generated documentation?**

Use the service's URL with `/docs` appended.

**Q38. Which schemas appear in FastAPI docs?**

The request, response and intent models.

**Q39. What did the human find that tests did not?**

An almost-empty plan file and a disabled human-support control during loading.

**Q40. What is the practical rule?**

**Test what the customer can do, not only what the code returns; check every agent report against the files it names.**

---

# Part 21: Quick Knowledge Check

## 47. Six Questions for Revision

Try to answer before opening each explanation.

<details>
<summary><strong>1. Why was an extra commit needed between phases one and two?</strong></summary>

**Answer:** The earlier plan-file copy failed. It contained only the title even though Claude Code had reported success. The team wrote the plan again and committed the correction.

</details>

<details>
<summary><strong>2. Which test results were reported?</strong></summary>

**Answer:** **13 fast tests passed**, **3 live tests were excluded** from that fast run, and **all 3 live tests passed** when run intentionally.

</details>

<details>
<summary><strong>3. What two code behaviors prevented requesting a person while waiting?</strong></summary>

**Answer:** The button was disabled during loading **and** the send function returned immediately if a response was in progress.

</details>

<details>
<summary><strong>4. Why was a pending-reply counter needed?</strong></summary>

**Answer:** Overlapping requests can finish at different times. The typing indicator should remain visible until **all** replies have completed, not switch off when the first finishes.

</details>

<details>
<summary><strong>5. What session problem remains after the widget fix?</strong></summary>

**Answer:** If the first response hasn't returned, the widget has no session ID yet; another message may start a separate conversation and later human support may lack the original message.

</details>

<details>
<summary><strong>6. Did the working person button mean human handoff was complete?</strong></summary>

**Answer:** **No.** The button worked, but the assistant truthfully stated that it could not transfer the customer. A real handoff belongs to the next build.

</details>

---

# Part 22: Complete Revision Checklist

## 48. Starting Point and Development History

- [ ] The previous lesson showed only **phase one** of a **seven-phase** plan.
- [ ] Claude Code completed the other **six phases** in fresh sessions.
- [ ] **Fast tests** were run after each phase.
- [ ] Once widget code existed, it was **built after each phase**.
- [ ] Each reviewed phase ended in a **Git commit**.
- [ ] Git log shows the brief, phase commits and a **separate plan repair commit**.

## 49. Plan-File Failure

- [ ] Claude Code reported that the plan file was saved.
- [ ] The file actually contained **only a title**.
- [ ] The copy operation failed without being reported.
- [ ] The plan was written again **between phases one and two**.
- [ ] The agent acknowledged it should have inspected the file before committing.
- [ ] **A report is only text; the file is the proof.**

## 50. Test Results

- [ ] **13 fast tests passed**.
- [ ] **Three** live tests were excluded from the fast run.
- [ ] All **three live tests passed** when run deliberately.
- [ ] Order number remembered from **first message**.
- [ ] AI disclosure appears **only on first reply**.
- [ ] Response carries **intent and language**.

## 51. Service Review

- [ ] `app.py` contains the chat service.
- [ ] The service has **one message endpoint**.
- [ ] Response includes **reply, session ID and intent**.
- [ ] Session ID supports **multi-turn continuity**.
- [ ] The single intent class is reused in **Claude, widget response and docs**.
- [ ] The **language** field propagates consistently.
- [ ] First-reply AI notice is prepended by the app.
- [ ] **AI notice is not stored in model conversation history**.

## 52. Widget Bug and Fix

- [ ] Widget code is in **`Chat.tsx`**.
- [ ] The person button was disabled while loading.
- [ ] The send handler rejected new messages during loading.
- [ ] Service tests could not catch the **frontend interaction** bug.
- [ ] Engineer asked for the button to **work at all times**.
- [ ] Button remains active during a pending reply.
- [ ] Widget allows another message while one reply is pending.
- [ ] Pending-reply count keeps typing indicator on until the **last reply**.
- [ ] Engineer independently **built the widget successfully**.
- [ ] Engineer **committed the fix**.

## 53. Open Session and Handoff Gaps

- [ ] First session ID arrives **with first reply**.
- [ ] Before that reply, an early second message may start a **new conversation**.
- [ ] Future human support could lose the **first customer message**.
- [ ] The issue was **recorded in the plan**, not fixed.
- [ ] A **real ServiceNow/human handoff** is still absent.
- [ ] Assistant must **not pretend** to have transferred the customer.

## 54. Browser and API Documentation

- [ ] README includes a **Run the chat** section.
- [ ] Start backend and widget in **two terminals**.
- [ ] Demo uses VS Code's **built-in browser**.
- [ ] Widget runs on **localhost port 5173**.
- [ ] First order question shows **AI disclosure**.
- [ ] Assistant requests an email **one-time code** because customer is not verified.
- [ ] OTP checking **comes later**.
- [ ] Customer clicks person button **before next response arrives**.
- [ ] Button **works** after the fix.
- [ ] Assistant states it **cannot transfer to a person yet**.
- [ ] FastAPI documentation is at the service's **`/docs`** path.
- [ ] Docs include **request, response and intent** schemas.

## 55. FDE Principles

- [ ] Check **artifact content**, not only agent reports.
- [ ] Review **code against the user requirement**, not only against tests.
- [ ] Know what backend tests, live tests and builds **cannot prove**.
- [ ] Exercise critical UI controls **under the failure/latency condition**.
- [ ] Distinguish **button works** from **human handoff works**.
- [ ] Explicitly record deferred issues and scope limitations.
- [ ] Commit small, independently verified fixes.
- [ ] **Test what the customer can do, not only what the code returns.**

---

# Part 23: Final Summary and FDE Takeaways

## 56. Summary Table

| Topic | What this lesson establishes |
|---|---|
| Seven implementation phases | Completed, with per-phase checks and commits |
| Plan artifact | Initially incomplete; repaired and separately committed |
| Fast tests | **13 passed** |
| Live tests | **3 passed intentionally** |
| Session history | Order number remembered across turns |
| API source file | **`app.py`** |
| API response | Reply, session ID, intent |
| AI notice | First reply only; excluded from history |
| Shared schema | Claude, widget, FastAPI docs |
| Widget source | **`Chat.tsx`** |
| Bug | Person button and send blocked during loading |
| Fix | Button always usable; overlapping sends permitted; pending counter |
| Remaining race | No session ID before first reply |
| Local widget | **localhost:5173** |
| README | **Run the chat** |
| Human handoff | Not implemented; next build |
| API docs | **`/docs`** |
| Most important process rule | Verify files and actual customer behavior |

## 57. What an FDE Should Be Able to Explain

At the end of this lesson, you should be able to explain:
- Why an agent's completion message can be wrong about what it saved.
- How to verify a multi-phase coding-agent project using repository history.
- Why **13 green backend tests plus three green live tests** still missed a serious UI requirement.
- How UI state can inadvertently block an escape action.
- Why a **pending-request counter** is better than a single loading boolean when requests overlap.
- Why the **first-response session-ID timing** creates a possible split-conversation race.
- Why the first browser run proves that the widget works **without** proving the handoff feature is complete.
- Why FastAPI-generated documentation benefits from shared Pydantic classes.

## 58. Course Practice Task

**Open the chat widget. Send a message and press “Talk to a person” before the assistant's reply arrives. Then read what the assistant answers.**

When evaluating your own implementation, record:
1. Was the button usable while loading?
2. Did the second message actually send?
3. Did the typing indicator behave correctly with multiple pending replies?
4. Was the conversation history preserved?
5. Did the assistant honestly state whether a real human transfer is available?

Different coding agents may produce different implementations. That is expected. The important thing is whether the **observed behavior** matches the business requirement.

> **The practical rule: Test what the customer can do, not only what the code returns. And check every report of the agent against the files it names.**
