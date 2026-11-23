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
