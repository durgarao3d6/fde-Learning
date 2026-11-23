# Client Discovery and Workflow Shadowing

## Learning objective
Find the real support workflow through evidence, not hypothetical stakeholder opinions.

## Four steps
1. **Prepare:** Read policy pages, Confluence notes, customer help articles and prior support conversations.
2. **Ask open questions:** Interview **Marco Rossi** and two or three agents without sponsor Lena Brandt in the room, encouraging candid accounts.
3. **Shadow real conversations:** Watch agents perform actual work, not just describe it.
4. **Record precise notes:** Capture sequence, systems, delays, workarounds and short verbatim quotes, without exposing customer personal information.

An illustrative meeting lasts **90 minutes**: 30 minutes interviewing and 60 minutes shadowing.

## Ask for recent reality
Good: **“Walk me through the last return chat you handled.”**

Less useful: **“Would a chatbot help you?”**

Pause, listen and paraphrase the answer back to validate understanding.

## Observed workflow
```text
ServiceNow case
  → copy order number
  → Ledgerline order record
  → nightly carrier status file
  → actual carrier site
  → saved reply template
  → manually edit delivery date
```

One agent estimated a task at **two minutes** but observation suggested **four minutes** (illustrative). A German-market return agent used a personal spreadsheet for return windows instead of trusting Confluence.

Short quote example: **“I never trust the carrier file in the morning.”**

## What to record
- Step, system, role and data source.
- Time spent and where users wait.
- Manual workarounds.
- Exceptions and unexpected cases.
- Evidence versus assumptions.

Separate **observed fact** from **hypothesis**, validate uncertainty later and send Marco a same-day summary. DPO Sofia reviews personal-data handling.

## Review questions
1. Why exclude the sponsor from frontline interviews?
2. Why are observed timings more reliable than estimates?
3. What does a personal spreadsheet reveal?
4. How should you handle customer data while shadowing?

## Remember
**Do not stop at asking how work is done. Ask to see it.**

---

# Detailed notes: learning how work is really done

## 1. Why interviews alone are insufficient
People often describe the official process rather than what they actually do. They also misestimate time and skip details they consider obvious. A Forward Deployed Engineer needs **behavioral evidence**, not only a polished stakeholder summary.

The lesson teaches four stages:
1. Prepare.
2. Ask open questions and listen.
3. Shadow live, real cases.
4. Record the right notes.

## 2. Preparation
Review material before the session:
- DemoStore's **Confluence return policy** pages.
- The public customer-help pages.
- Previously labeled support chats.
- Existing systems and integration descriptions.

The goal is to arrive knowing the vocabulary and with questions about actual workflow, **not** to assume the documentation is fully accurate.

### Who attends?
Meet **Marco Rossi**, head of support, with **two or three frontline agents**. **Lena Brandt**, the sponsor, is not present for the frontline discovery session so employees can more comfortably describe workarounds and friction.

This is not a judgment against the sponsor; it reduces social pressure.

### Illustrative meeting duration
**90 minutes total:** about **30 minutes of interviewing** followed by **60 minutes observing work**. Treat those durations as a lesson example, not a universal scheduling requirement.

## 3. Good versus weak questions
Bad hypothetical:
> “Would a chatbot help?”

It invites optimism and speculation.

Better question:
> “Walk me through the last return chat you handled.”

Other productive prompts:
- “Show me the last time tracking data was wrong.”
- “What did you do next?”
- “What system were you in at that moment?”
- “When did you decide to call a person?”
- “What information was missing?”
- “Can you show me how you verify a return window?”

Ask a **recent concrete example**, then follow the sequence.

## 4. Listening technique
Do not interrupt with solution proposals. Let the agent describe what happened. Pause before asking the next question. Paraphrase:
> “So you check Ledgerline first, then verify the carrier status elsewhere because the overnight file may be stale—is that right?”

Paraphrasing reveals misunderstanding quickly.

## 5. Shadow the workflow
Observed pattern:
```text
ServiceNow: open support conversation
   ↓
Copy customer/order number
   ↓
Ledgerline: locate order record
   ↓
Carrier nightly file: inspect last known status
   ↓
Carrier website: check for more recent information
   ↓
Saved reply: paste standard wording
   ↓
Manually edit date/status
   ↓
Send response
```

This sequence uncovers repeated copy/paste, multiple screens and differences between official data and actually trusted data.

### Timing discrepancy
One agent estimated the sequence takes **two minutes**. Direct observation suggested **four minutes**. Both numbers in the lesson are illustrative.

Why it matters: A business case based on estimates might understate labor costs by a factor of two.

## 6. The German-market spreadsheet
A frontline agent supporting German returns uses a **personal spreadsheet** for return windows, rather than relying entirely on Confluence. This is a critical finding:
- Official documentation may be outdated or inconvenient.
- Agents have built shadow procedures to compensate.
- Country-specific policy may not be standardized.
- Copying a policy page blindly into a chatbot would reproduce existing errors.

Do **not** immediately encode the spreadsheet as an authoritative rule. Investigate and have the correct owner approve the rule source.

## 7. Capture short verbatim quotes
Example:
> “I never trust the carrier file in the morning.”

This short quote tells the project team something that a workflow diagram might hide: the agent knows the nightly source may be stale and uses a second site to cross-check it.

Quotes are evidence, but must be anonymized appropriately.

## 8. Good notes: four dimensions
| Dimension | Record | Example |
|---|---|---|
| Steps and systems | What happened in which app? | ServiceNow → Ledgerline → carrier |
| Timing | How long did it actually take? | ~4 minutes observed |
| Pain and workaround | What was unreliable or manual? | Manually edited shipping date |
| Direct evidence | Short quotation or observation | “I never trust the carrier file…” |

Also record exceptions, different markets, assumptions, potential automation opportunities and unanswered questions.

## 9. Fact versus hypothesis
**Observation:** “Agent opened the carrier website after checking the nightly CSV.”

**Hypothesis:** “The nightly carrier data is too stale for customers to trust.”

**Proposed intervention:** “Integrate a real-time carrier API.”

These are **three different claims**. The first is observed. The second needs confirmation. The third is a design option that needs a feasibility and cost check.

A good FDE does not convert hypotheses into facts by formatting them as meeting minutes.

## 10. Privacy and follow-up
Do not copy actual customer personal information into informal notes or model prompts. **Sofia Lindqvist** governs approved processing and sharing. Keep identifiers redacted or anonymized.

Send **Marco a same-day summary** so he can correct misunderstandings. Unresolved items become explicit questions and later traceable user stories.

## 11. Link to requirements
After discovery, each user story should point back to the line of notes supporting it. That gives the future coding-agent interview **traceability**.

Example (illustrative):
> Story: “As a support agent, I need a reliable shipment status, so I don't quote an outdated delivery estimate.”
>
> Evidence: observation that the agent cross-checked a stale carrier CSV against the actual site.

## 12. Common pitfalls
- Interviewing only a sponsor or manager.
- Asking for general opinions rather than recent cases.
- Trusting official SOPs without observation.
- Recording vague conclusions instead of observed steps.
- Assuming estimated effort is actual effort.
- Copying private customer data into notes.
- Treating a personal spreadsheet as approved business policy.
- Finishing discovery without letting stakeholders verify the summary.

## 13. Interview-ready response
**Question:** “How do you run a client discovery session for an AI integration?”

**Answer:** “I review real policies and cases first, interview frontline staff in a setting where they can be candid, ask for recent specific cases, then shadow the actual systems and timings. I record steps, workarounds, evidence and uncertainty separately, avoid capturing unnecessary personal data, and validate the notes with the process owner before converting them into traceable user stories.”

### Revision checklist
- [ ] Four steps: **prepare, ask/listen, shadow, record**.
- [ ] **Marco + two or three agents**, sponsor Lena absent.
- [ ] **90 minutes: 30 interview + 60 shadow**, illustrative.
- [ ] ServiceNow → Ledgerline → CSV → carrier site → template.
- [ ] **Two-minute estimate vs four-minute observation**.
- [ ] German agent's **personal spreadsheet**.
- [ ] Verbatim quote about **morning carrier file**.
- [ ] Same-day summary to **Marco**.
- [ ] Privacy owner **Sofia**.
