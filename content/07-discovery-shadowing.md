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

---

# Expanded Observation Template and Case Analysis

## One sample observation record
The following is a template inspired by the transcript's workflow, not a real customer record.

| Field | Example |
|---|---|
| Case type | Delayed parcel inquiry |
| Frontline role | Support agent |
| Starting system | ServiceNow |
| Order source | Ledgerline |
| Carrier source | Nightly CSV and direct carrier website |
| Human workaround | Copy tracking number, cross-check update, edit template |
| Estimate versus observation | 2 minutes estimated, 4 minutes observed (illustrative) |
| Confidence | Workflow observed; root cause requires confirmation |
| Next question | Can the carrier data be updated more frequently? |

## Why every detail matters
The **manual date edit** is not merely a nuisance: if a template reuses an outdated delivery date, the assistant could make incorrect delivery promises.

The **carrier website cross-check** suggests that the nightly CSV may not be sufficiently fresh for some customer requests. However, we should verify how often this occurs before buying a more expensive real-time integration.

The **personal German returns spreadsheet** may contain important tacit knowledge. It might also be wrong or old. The FDE must have the correct owner validate it before treating its contents as a business rule.

## Meeting behavior: examples
**Open-ended prompt:** “Tell me about the last support case that required more than one system.”

**Follow-up:** “Could you show me the exact next click?”

**Time probe:** “Let's measure that part instead of estimating.”

**Workaround probe:** “Where do you keep the numbers you actually trust?”

**Reflection:** “I heard that the published policy wasn't what you consulted. Is that correct?”

**Summary to Marco:** “Here's what was observed, what is hypothesized and what we still need to confirm.”

## Four-column notes can prevent premature requirements
| Observed | Hypothesis | Possible opportunity | Validation owner |
|---|---|---|---|
| Agent checked carrier site after CSV | Nightly feed may be stale | Improve status freshness | Marco / Ravi |
| German agent consulted personal sheet | Confluence return policy may lag practice | Central approved rules | Marco / policy owner |
| Template date edited manually | Current workflow creates avoidable mistakes | Populate verified dates automatically | Support + engineering |

The “possible opportunity” column is explicitly **not yet an approved requirement**.

## Privacy handling
Transcripts, order IDs, emails and real customer messages may contain personal information. The FDE should use permitted storage and redacted examples, work within Sofia's approvals, and avoid pasting sensitive records into general-purpose notes or unapproved model contexts.

## Why send notes the same day?
The agent can correct a misunderstanding while the event is fresh. Waiting several days increases the chance the team will convert an inaccurate observation into an accepted requirement.

## The bridge to the coding-agent interview
A later lesson says each user story points to a **specific discovery-note line**. Good notes make the coding agent's recommended answers defensible. Poor notes make the agent guess.


---

# FDE Interview Questions & Answers — Comprehensive Practice Bank

Use these prompts for oral practice. Answer aloud before revealing the explanation beneath each question.

<details>
<summary><strong>1. What are the four discovery steps?</strong></summary>

Prepare; ask open questions and listen; shadow real customer cases; record precise evidence and follow up.

</details>

<details>
<summary><strong>2. Who should join the first frontline session?</strong></summary>

Marco Rossi and two or three support agents.

</details>

<details>
<summary><strong>3. Why keep sponsor Lena Brandt out of that session?</strong></summary>

To reduce social pressure and encourage honest discussion of workarounds or problems.

</details>

<details>
<summary><strong>4. What materials are reviewed before the meeting?</strong></summary>

Confluence policy documents, public help pages, earlier support chats and the known workflow.

</details>

<details>
<summary><strong>5. What example time allocation is given?</strong></summary>

An illustrative 90 minutes: 30 minutes interviewing and 60 minutes observing.

</details>

<details>
<summary><strong>6. What's a better question than 'Would a chatbot help?'</strong></summary>

'Walk me through the last return chat you handled'—a concrete recent event.

</details>

<details>
<summary><strong>7. How should an interviewer react to a surprising explanation?</strong></summary>

Listen, pause, paraphrase what they heard and ask to see the actual steps.

</details>

<details>
<summary><strong>8. What is the demonstrated workflow?</strong></summary>

ServiceNow → copy order number → Ledgerline → nightly carrier CSV → live carrier site → saved reply → manual date edit.

</details>

<details>
<summary><strong>9. What did the timing comparison reveal?</strong></summary>

The agent estimated about two minutes; direct observation suggested four minutes, illustrative.

</details>

<details>
<summary><strong>10. Why was the morning carrier file distrusted?</strong></summary>

An agent said 'I never trust the carrier file in the morning', suggesting freshness concerns that need investigation.

</details>

<details>
<summary><strong>11. What was unusual about German returns?</strong></summary>

An agent used a personal spreadsheet with market return windows instead of relying on Confluence.

</details>

<details>
<summary><strong>12. Should the spreadsheet become the source of policy truth immediately?</strong></summary>

No. Validate it with the appropriate policy owner; workarounds may contain useful knowledge but also errors.

</details>

<details>
<summary><strong>13. What should observation notes contain?</strong></summary>

Steps and systems, timings, difficulties and workarounds, and short verbatim quotes without private customer data.

</details>

<details>
<summary><strong>14. Why separate observation from hypothesis?</strong></summary>

A behavior you watched does not by itself prove its root cause or justify a particular technical solution.

</details>

<details>
<summary><strong>15. What privacy role is involved?</strong></summary>

Sofia Lindqvist oversees approved handling of real personal data.

</details>

<details>
<summary><strong>16. What should happen the same day?</strong></summary>

Send Marco a summary so factual misunderstandings can be corrected promptly.

</details>

<details>
<summary><strong>17. How are the notes used in later requirements?</strong></summary>

Every user story points to the relevant line of discovery evidence.

</details>

<details>
<summary><strong>18. What is the central quote?</strong></summary>

Do not stop at asking how the work is done. Ask to see it.

</details>

## How to practise with this chapter

1. Explain the lesson's actual business problem in 45 seconds.
2. Recreate the architecture or sequence from memory.
3. Recall exact numbers and named decision-makers without checking the notes.
4. Explain one failure case and the deterministic safeguard.
5. Use the Q&A above to prepare a two-minute FDE interview answer.

