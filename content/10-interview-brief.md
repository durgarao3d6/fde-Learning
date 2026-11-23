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
