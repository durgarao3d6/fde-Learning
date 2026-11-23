# Let Claude Code Interview You Before Writing the Brief

## Learning objective
Turn discovery and user stories into a short, approved implementation brief through structured clarification.

## Starting artifacts
- Discovery notes grounded in frontline workflows.
- User stories stating **who wants what and why**.
- A reference from each story to its discovery-note line.
- A list of unresolved questions.
- Existing code and project rules.

The target is a **one-page brief** describing what to build, constraints, exclusions and open questions.

## Interview process
Run Claude Code in **Plan Mode**. Ask it to read requirements, notes, rules and code first. Then ask **one question at a time**, each with a recommended answer and reason, until important uncertainties are resolved. The engineer can accept, reject or supply another answer.

This interview prompt may also be saved as a reusable Claude Code **skill**.

## Eight decisions
1. **First capabilities:** recommend order status plus a human-support option. Four in ten chats ask about status and tools already exist. Defer returns while policy rules remain open.
2. **Human handoff:** agent offers ServiceNow case, contact form or live queue. Engineer chooses none yet; the button only sends a message. Real ServiceNow handoff comes later.
3. **Order data:** use sample files behind existing tools because Ledgerline is not accessible.
4. **Verification:** not implemented; session's verified-customer field remains empty.
5. **Conflict:** order lookup requires a verified customer, so it cannot succeed. Agent suggests a configured demo customer. Engineer rejects bypass and chooses to disclose that lookup is unavailable.
6. **Prototype aim:** show an end-to-end flow and clearly state limitations.
7. **Storage:** sessions in memory only; no durable conversation storage.
8. **Language:** test in English; include a language field in intent for future handoffs.

## Three things only the engineer added
**AI notice:** first response discloses AI; customers can request a person. The lesson connects disclosure to EU AI Act transparency and treats the person option as an additional DemoStore requirement.

**Single intent model:** one Python/Pydantic class determines Claude's output shape, the widget response, and generated service documentation.

**Loading feedback:** the widget shows that a reply is on its way.

## File and final scope
After approval, Claude Code writes **`docs/brief-chat.md`**. The engineer opens and reads the actual file.

The implementation is reduced to five deliverables: **service, AI notice, one intent class, widget, opt-in tests that really call Claude**.

No persistent sessions, no order data without verified identity, and no Ledgerline writes. Real ServiceNow handoff, one-time-code delivery, and returns wait.

## Deliberate difference
The interview brief says the chat cannot look up orders. The build preserves the two-tool agent loop and verification guardrail, so it can request a one-time code **before the service can actually send one**. This version is **internal only**; do not misrepresent it as a ready customer experience.

## Practice
Ask Claude Code to interview you about a feature, one question at a time. Note which required answers were **not** in the documentation.

## Remember
**Let the agent interview you before it writes a brief; add what only you know.**

---

# Deep dive: getting a brief from an interview

## 1. Previous lesson to this lesson
The coding agent will eventually produce software, but the user must first tell it **what it should build**. Writing that from a blank page increases the chance of forgetting discoveries or silently inventing decisions.

At DemoStore, the team already has:
- Discovery notes from real frontline support work.
- User stories described as **who wants what and why**.
- A **line-level reference** from each user story to its source note.
- A list of unresolved questions.
- Existing scripts and repository rules.

The deliverable is **one short page** with what will be built next, limitations and what waits for later.

## 2. The interview protocol
Use Claude Code in **Plan Mode**: read and ask, but do not change code yet. Prompt it to:
1. Read requirements.
2. Read discovery notes.
3. Read project rules.
4. Read existing code.
5. Ask **one question at a time**.
6. Attach a **recommended answer and rationale** to each question.
7. Continue until important decisions are clear.
8. Draft the brief with scope, constraints, exclusions and open questions.

This can be supplied as a **plain prompt** or packaged as a reusable Claude Code **skill**—a saved instruction set.

### Why sequential questions?
The answer to one question may create the next question. An adaptive interview can discover contradictions that a fixed questionnaire misses.

### Why recommended answers?
The recommendation anchors the conversation in available evidence and reduces friction. But the user can reject it or answer in their own words.

## 3. Eight questions explained in depth

### Question 1 — What does the first version answer on its own?
**Agent recommendation:** **Order status** and a route toward a person.

**Evidence:** About **four in ten** support chats ask where an order is; `find_order` and delivery-status tools already exist.

**Decision:** Accept the recommendation. **Returns** wait because their policy rules are unresolved.

**Engineering significance:** Start with a frequent use case and available integration, not the broadest possible assistant.

### Question 2 — What happens on “Talk to a person”?
**Options offered:** ServiceNow case, contact form or a live support queue.

**Decision:** None yet. The first version's button **sends a message only**. An actual ServiceNow case will be implemented in the following build.

**Significance:** Do not pretend a real escalation path exists when it has not been integrated. The human is free to reject every suggested option.

### Question 3 — Where does order data come from?
**Agent recommendation:** Sample files behind existing tools, because direct **Ledgerline** access is not yet available.

**Decision:** Accept.

**Significance:** Stable tool interfaces let a mock backend become a real backend later.

### Question 4 — How is the customer verified?
**Decision:** Not in this version. Each conversation's session object has a **verified-customer field**, but it remains **empty**.

**Significance:** You can design for later authentication without falsifying the current state.

### Question 5 — The authorization contradiction
Here the agent catches something genuinely valuable:

1. The order tool reveals records only to verified customers.
2. No customer is verified yet.
3. Therefore every order lookup will fail as **not verified**.
4. Order status was meant to be the first supported capability.

**Agent suggestion:** Put a demo customer into configuration.

**Engineer decision:** Reject that shortcut. The chat should clearly disclose **order lookup is unavailable yet**.

**Why it matters:** A realistic-looking demonstration should not come at the price of breaking the security model. The scope of the prototype legitimately becomes narrower than the original feature wish.

### Question 6 — What is the first build really meant to show?
**Decision:** An end-to-end workflow that operates honestly and clearly states its limitations.

**Significance:** A vertical slice of a product can be a worthwhile technical milestone even when the principal business action remains disabled.

### Question 7 — Where is conversation history stored?
**Decision:** In memory only; **no persistent storage**.

**Consequence:** A service restart discards sessions.

**Significance:** Keep the first implementation small, but never confuse an ephemeral conversation with production-ready durability.

### Question 8 — Which languages?
**Decision:** Test in **English**, and add a **language field** to the intent result, so future human handoffs receive the customer's language.

**Significance:** A field can prepare a future integration without claiming multilingual readiness.

## 4. The three requirements the agent missed
Even after eight questions, the engineer sees three missing constraints. They are not in the documents, so the agent had no reason to infer them reliably.

### Missing requirement A — disclosure and human access
The first assistant response states **the user is talking to an AI** and can ask for a person at any time.

The lesson cites the **EU AI Act's transparency rationale** for telling people they are interacting with AI. The **path to a person** is an additional DemoStore rule, not automatically equivalent to the Act's text.

**Implementation caution:** Do not claim that a message-only prototype button has already transferred the conversation.

### Missing requirement B — one intent class
Represent intent as **one Python class** rather than separate schema copies.

Three users of the definition:
1. Structured response required of Claude.
2. JSON response from backend to the widget.
3. Self-generated API documentation.

A field such as `language` is added once, not maintained separately in multiple schema dictionaries.

### Missing requirement C — visible pending state
While an LLM request is processing, the widget shows **that a reply is on its way**. A chat interface without loading feedback can appear broken.

## 5. Save the actual artifact
Once the human approves the brief, Claude Code writes:
```text
docs/brief-chat.md
```

The engineer then **leaves Claude Code**, opens the file, and independently checks what was actually written.

Important contents:
- What to build, including all three human-added requirements.
- **No stored sessions**, no order data without verified identity, no Ledgerline writes.
- Defer **real ServiceNow handoff, one-time code and returns**.
- Carry forward open questions without pretending they are solved.

A chat response claiming “saved” is not evidence until the repository file has been read.

## 6. The five-point implementation brief
For the next lesson the team intentionally narrows to:
1. **Chat service**.
2. **AI notice**.
3. **One intent class**.
4. **Website widget**.
5. **Opt-in real-Claude integration tests**.

Live tests only run when explicitly requested. Scope discipline beats padding the first version with secondary features.

## 7. The deliberate difference to remember
The interview says the first chat cannot look up orders. The actual following build **preserves the existing two-tool agent loop and its verification check**. The assistant may request a one-time code even though the following build must still implement code delivery.

**Until then, only the project team uses this chat.** It is not an honest customer-ready experience.

Don't quietly smooth over this inconsistency in study notes; the lesson calls it out explicitly.

## 8. Requirements engineering patterns
### Traceability
Each story points to source notes. This lets reviewers ask “why do we need this?” and find an observed customer-support need.

### Explicit deferral
A future feature should be listed as **out of scope**, not silently absent.

### Conflict discovery
An interview can reveal incompatible decisions before money is spent on code.

### Human-owned assumptions
The engineer must add policy, UX and architecture rules not present in the available notes.

### File-based approvals
A small committed document is easier to review, hand over and feed into later coding sessions.

## 9. Illustrative interview prompt
```text
Read the user stories, linked discovery notes, open questions,
CLAUDE.md and the current code. Do not modify code.

Interview me about the next build one question at a time.
For each question, recommend an answer and cite the evidence.
Allow me to reject or rewrite the answer.
Continue until all decisions needed for the next build are clear.

Then propose a one-page brief: build now, limits,
explicitly out of scope, and open questions.
Show it to me before saving docs/brief-chat.md.
```

## 10. Interview-ready explanation
**Question:** “Where should a coding-agent brief come from?”

**Answer:** “From traceable user stories and discovery evidence, not from the agent inventing the requirements. I have the agent review notes, rules and code, then interview me one question at a time with recommended answers. I resolve conflicts—such as order lookup requiring verification that is not built—and add missing UX, compliance and schema requirements myself. We approve and inspect the resulting brief before implementation.”

### Revision checklist
- [ ] User stories **who / what / why** and **source-note line**.
- [ ] **Plan Mode** reads but does not edit code.
- [ ] **One question at a time** plus recommendation.
- [ ] **Eight interview questions** and their answers.
- [ ] **Four in ten** order-status chats.
- [ ] Handoff choice: **message-only**, real **ServiceNow later**.
- [ ] Mock **Ledgerline data**.
- [ ] Empty verified-customer session field.
- [ ] **Reject demo customer** bypass.
- [ ] **English + language field**.
- [ ] **Three added requirements**.
- [ ] Saved brief path **`docs/brief-chat.md`**.
- [ ] **Five final implementation points**.
- [ ] Internal-only mismatch between brief and preserved agent loop.

---

# Expanded Study Companion: Complete Walkthrough of the Interview Lesson

This section is organized to preserve the small decisions and transitions in the original teaching sequence. It separates **what the transcript explicitly says** from **engineering examples** added to explain it.

## Part A — Why the brief does not start on a blank page

**Opening question:** “Where does a brief for the coding agent come from?” Answer: **an interview**. This is a deliberately different workflow from writing a giant specification alone and hoping the agent reads it correctly.

The team has already held discovery sessions. It has translated those results into **user stories**. The story format is about a person, a need and the reason:

> As a [user], I want [capability], so that [outcome].

Crucially, **each story names the particular line in the discovery notes that supports it**. This is requirements traceability. If a project reviewer disputes a feature, the FDE can trace it back to an actual workflow observation or support-agent account. It also has a **separate list of open questions**—uncertainty is visible rather than disguised as specification.

### What is the difference between four documents?
| Artifact | Question it answers | Example |
|---|---|---|
| Discovery notes | What did people actually do and say? | Agent checked a carrier website because the CSV might be stale |
| User stories | Who needs which outcome, and why? | Customer needs timely shipment status |
| Open questions | What is still unknown? | Are return-policy dates confirmed for all markets? |
| Next-build brief | What must be built now, with what limits? | API, disclosure, schema, widget, live tests |

The **brief should be one short page**. Length is part of the discipline: it defines a *specific next slice of work*, not an encyclopedia of all future features.

## Part B — How the agent interview is started

Launch **Claude Code in Plan Mode**. The teaching distinction is explicit: in this mode **it can read and ask, but changes nothing**. The engineer does not let it implement requirements before unresolved questions are settled.

The input instruction has an exact sequence:
1. Read the requirements.
2. Read the discovery notes.
3. Read repository rules.
4. Read the code that already exists.
5. Interview the engineer **one question at a time**.
6. Provide a **recommended answer for every question**.
7. Continue until **nothing important for this brief is unclear**.
8. Write a concise brief: build now, limits, out of scope, remaining unknowns.

The lesson also mentions a **saved skill**, meaning a reusable package of such instructions for Claude Code. In this lesson, the team instead types a **plain prompt**. A skill is a reusable convenience; it is not a separate source of requirements.

### Why must it inspect code before interviewing?
Because the answer to “Can we do order status?” depends on whether `find_order` already exists, what its input fields mean, and what it enforces. Asking without examining existing implementation creates questions that don't match reality.

### Why one question at a time?
The interview is **adaptive**. A later question can point out a contradiction created by earlier answers. A long questionnaire delivered all at once might not notice the dependent relationship between verification and order lookup.

## Part C — Interview question 1: first autonomous use case

**Question:** Which customer questions should version one answer by itself?

**Recommendation:** Order status, plus an easy way toward a person.

**Evidence:** Roughly **four in ten** support chats involve “where is my order?” and relevant tools have already been built. This connects implementation scope to observed support volume and available software capability.

**Decision:** The engineer accepts.

**Deferred:** Returns. The relevant return rules remain **open questions**. The team will not encode policy assumptions as if approved.

**Subtle point:** “A way to a person” at this stage means a *visible path*, not necessarily a completed automated transfer. That distinction becomes central to question two.

## Part D — Interview question 2: human-handoff destination

**Question:** Where does a user go when asking for a person?

**Agent's three options:**
1. Create a **ServiceNow case**.
2. Send them to a **contact form**.
3. Send them to a **live queue** of support agents.

**Human's answer:** None of them **in this version**. The **“Talk to a person” button only sends a message for now**. The *real* handoff—creating a **ServiceNow case**—belongs to the **next build**.

**Why this matters:** The engineer can reject all AI-suggested options and respond in original words. The options are suggestions, not an enforced menu.

**UX implication (engineering guidance):** Label the button's current behavior honestly; don't tell the user a case has been opened if the system only sends a message.

## Part E — Interview question 3: order-data source

**Question:** Where does the order data come from?

**Agent recommendation:** **Sample files behind the existing tools**.

**Evidence:** No production access to **Ledgerline** yet.

**Human decision:** Agree.

**Architectural consequence:** Keep the same tool names and contracts while the storage implementation changes later. The assistant should not need to know whether `find_order` is backed by a JSON fixture or a guarded ERP middleware call.

## Part F — Interview question 4: customer verification

**Question:** How is the customer verified?

**Answer:** **Not in this version.**

The **session**, described as the record associated with one chat, includes a field for the **verified customer**. The field is defined but **stays empty**.

**Crucial difference:**
- A **session ID** identifies an ongoing chat.
- A **verified customer identity** establishes the caller's entitlement.
- Having a session ID is **not** proof of ownership of an order.

This design leaves space for the later identity-verification feature without pretending it already works.

## Part G — Interview question 5: a real contradiction

Claude Code connects two earlier statements:
1. The **`find_order`** tool only returns an order to a **verified customer**.
2. No session has a verified customer in this first version.

**Deduction:** Every order lookup will return a version of **“not verified.”** The initially prioritized order-status feature therefore cannot succeed.

The model asks **what the application should do** rather than silently bypassing a security rule.

**Agent recommendation:** Put a **demo customer in configuration**. This would make the demonstration superficially functional.

**Engineer choice:** Reject that shortcut and say **plainly that the chat cannot look up orders yet**.

**Outcome:** The first version will answer **no protected order-status questions**, even though order status was its first proposed use case.

### Why this is not an accidental inconsistency
It is a stage decision. The prototype can prove that a widget, service and agent communicate end to end **while keeping the authorization guarantee intact**. A fake preverified identity might confuse later developers into copying an insecure approach.

### A useful test (illustrative)
```python
def test_order_lookup_requires_verified_customer():
    session = {"verified_customer": None, "history": []}
    result = find_order("ORDER-DEMO-1", customer=session["verified_customer"])
    assert result["error"] == "verification_required"
```

The exact function signature and error shape depend on the repository. The invariant is **no verified customer → no private order data**.

## Part H — Question 6: scope of the first delivery

The transcript summarizes three quick answers after the conflict.

First: **Version one runs end to end and states clearly what it cannot do.**

“End to end” means a user message can travel through the widget and service and get a response. It does **not** mean every support workflow is available.

A functional vertical slice is useful for internal testing of interfaces, session routing and service shape.

## Part I — Question 7: session persistence

Second quick answer: **Nothing is persisted**. Sessions stay **in memory inside the running application**.

**What that means:**
- Conversation history is lost on restart.
- It is not a multi-instance, durable session store.
- It is a prototype choice, **not production persistence**.

**What it does not necessarily mean:** That nothing is ever logged by infrastructure or sent to the model provider. The transcript's limit specifically concerns application session storage; privacy must be assessed separately.

## Part J — Question 8: language behavior

Third quick answer: **Test in English**, and **add a language field to the intent structure**.

The customer language can later be forwarded to a human support agent. This is especially relevant to DemoStore's multi-market support context.

**Important distinction:** Capturing `language` does not equal certified multilingual model performance. The initial tests are **English**.

An *illustrative* future intent result:
```json
{
  "summary": "Customer asks for a tracking update.",
  "intent": "order_status",
  "order_number": "ORDER-123",
  "needs_human": false,
  "language": "en"
}
```
The actual six intent labels are **not named in this transcript**.

## Part K — The eight-question decision table

| No. | Question | Agent's suggestion | Human-approved answer |
|---|---|---|---|
| 1 | Which questions first? | Order status + human path | Accept; returns deferred |
| 2 | Where to send human requests? | ServiceNow case / form / queue | None yet; message-only button |
| 3 | Where does order data come from? | Existing tools backed by sample files | Accept |
| 4 | How is identity verified? | Clarification required | Not built; field stays empty |
| 5 | Resolve lookup-verification conflict? | Configured demo customer | Reject; clearly state order lookup unavailable |
| 6 | What counts as first build? | Clarify prototype goal | End-to-end, honest limits |
| 7 | Persist sessions? | Clarify storage | Memory only |
| 8 | What languages? | Clarify language | Test English; add language field |

The source says the eight questions occurred in this sequence. It presents the last three answers more briefly than the first five; this table respects that distinction.

## Part L — The three missing requirements

After eight answers, Claude Code shows the proposed brief. The engineer notices **three absent requirements**, because **the underlying discovery documents never stated them**. The model cannot reliably discover unstated client/product principles.

### 1. First-message AI notice and human-support rule
First assistant reply clearly says **this is an AI assistant** and the customer can **request a person at any time**.

The lesson references the **EU AI Act** for AI-interaction transparency. The *person-access* requirement is **DemoStore's own rule**. Do not misattribute the company's design requirement to every clause of the law.

### 2. One Python intent class, shared in three places
Define the shape of intent **once** as a **Pydantic/Python class**.

The three consumers:
- Claude's required structured response.
- Service response sent to the widget.
- Service's automatically generated documentation.

A new field, e.g. `language`, should propagate because all derive from one source. With duplicate schemas, adding a field in only one could break chat processing.

### 3. Widget pending-response feedback
The chat interface must visually indicate **a reply is on its way**. This is the **typing/loading indicator** requirement.

It is a small UX detail, but customers need to know the message was received while an LLM call takes time.

## Part M — Approval, file creation, independent review

After the engineer adds the three requirements and approves, Claude Code writes a Markdown file:

```text
docs/brief-chat.md
```

The engineer **exits Claude Code and opens that file directly**. This prevents confusing the agent's completion message with the actual saved specification.

Check the sections:
1. **Build now**, including the three human-added requirements.
2. **Limits:** no persistent sessions; no private order information without verification; no writes to Ledgerline.
3. **Later builds:** real ServiceNow handoff, sending/verification of one-time codes, returns.
4. **Open questions:** explicitly carried forward rather than guessed away.

## Part N — The five implementation points

The team further narrows the next coding task to:
1. **Chat service**.
2. **AI notice**.
3. **One intent class**.
4. **Chat widget**.
5. **Tests that actually call Claude**, but **only when asked**.

This is deliberate scope control. A brief is not permission to add every interesting integration immediately.

## Part O — Preserve the intentionally incomplete agent behavior

One subtle distinction should remain explicit in your revision material:

**Interview brief:** Clearly say order lookup cannot currently be performed.

**Actual next build:** Keep the previous agent loop, its **two tools**, and the check for a **verified customer**. The assistant can ask for a **one-time code**, but the next build has **not yet implemented sending the code**.

**Operational rule:** **Nobody except the internal team uses the chat yet.**

The difference is not a hidden bug in the notes. The lesson **calls it out on purpose**. Do not describe this as a completed order-status customer feature.

## Part P — Requirements-engineering insights

**Traceability:** Every requirement needs a path back to evidence or an explicit human decision.

**Adaptive questioning:** Sequential interviews reveal conflicts; bulk forms can miss them.

**Recommendations are not authority:** The human can select “none of these.”

**Security before demo polish:** Reject an unsafe simulated identity even when it would make the prototype look more complete.

**Explicit deferred scope:** Write down what comes in later builds.

**Single source of truth:** Schemas should be generated from one model.

**Human completeness review:** AI cannot guarantee it asked about every unstated requirement.

**Durable artifact:** The final brief lives in the repository, not in a transient chat.

## Part Q — Interview question bank, with answers

**Q1. What is a coding-agent brief?** A short next-build specification stating scope, limits, out-of-scope work and remaining questions.

**Q2. Why not begin with a blank page?** Discovery notes and user stories provide real evidence, reduce rework and support traceability.

**Q3. What does Plan Mode do here?** Reads and asks; does not edit code.

**Q4. Why one question at a time?** The next question can depend on the previous answer, making contradictions visible.

**Q5. Why give a recommended answer?** To ground options in evidence and make decisions efficient, while preserving human control.

**Q6. Why prioritize order status?** About **40%** of cases involve it and relevant tools already exist.

**Q7. Why defer returns?** Return rules still contain unresolved policy questions.

**Q8. Which handoff options were proposed?** ServiceNow case, contact form or live queue.

**Q9. Which was chosen?** None in the first version; the button sends a message.

**Q10. What data backend is used?** Sample files behind existing tools because Ledgerline access is missing.

**Q11. What state does the chat session include?** A verified-customer field, empty for now.

**Q12. What conflict was discovered?** A verified customer is required for order lookup, but nobody is verified.

**Q13. What workaround was rejected?** A demo customer automatically set in configuration.

**Q14. What safer option was accepted?** State clearly that order lookup is not available.

**Q15. Is the first version useless?** No; it demonstrates end-to-end connectivity and honest limitations.

**Q16. Are chat sessions persisted?** No; they live in memory and disappear on restart.

**Q17. What language is tested?** English.

**Q18. Why include `language`?** Human support can later see the customer's language.

**Q19. What did Claude Code fail to ask?** AI disclosure/human-access rule, one intent class, typing indicator.

**Q20. What motivates AI disclosure?** The lesson cites EU AI Act transparency requirements.

**Q21. Is a path to a person described as a separate rule?** Yes, DemoStore's own requirement.

**Q22. What three places consume the intent class?** Claude output, widget API response, generated API docs.

**Q23. Where is the brief saved?** `docs/brief-chat.md`.

**Q24. Which features wait?** Real ServiceNow handoff, code verification/delivery, returns.

**Q25. Are live tests automatic?** No; they are opt-in.

**Q26. Why is the first build internal only?** It cannot yet send verification codes or make a real handoff.

**Q27. What is the practical rule?** Let the agent interview you, then add what only you know.

## Part R — Full revision checklist

- [ ] I can distinguish discovery notes, traceable stories, questions and brief.
- [ ] I know the agent reads requirements, notes, rules and **existing code** first.
- [ ] I can state why it asks **one question at a time** with a recommendation.
- [ ] I can reconstruct **all eight questions** without looking.
- [ ] I know **four in ten chats** drive the initial order-status suggestion.
- [ ] I can explain why returns are deferred.
- [ ] I can name **all three proposed handoff destinations**.
- [ ] I remember the button is **message-only** in version one.
- [ ] I remember **sample data behind tools**, not Ledgerline.
- [ ] I can distinguish session identity from verified customer identity.
- [ ] I can explain the **verification contradiction**.
- [ ] I can justify rejecting a **configured demo customer**.
- [ ] I know the version **cannot answer protected order requests**.
- [ ] I know sessions are **in-memory** only.
- [ ] I know the first language for testing is **English**.
- [ ] I remember all **three human-added requirements**.
- [ ] I can explain why **one Pydantic class** feeds **three outputs**.
- [ ] I know the approved file path **`docs/brief-chat.md`**.
- [ ] I can state the **five build points**.
- [ ] I can distinguish the brief's limitation from preserved agent-loop behavior.
- [ ] I know **only the team** is allowed to try this incomplete build.

**Practical task from the course:** Interview yourself with Claude Code about a new feature, one question at a time. Record every decision that could not be found in your existing notes.
