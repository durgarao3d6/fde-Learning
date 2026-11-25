# Complete Study Notes: TypeScript Fundamentals for DemoStore's Browser Chat Widget

**FDE Course · Frontend Fundamentals · TypeScript · JavaScript · Interfaces · Async/Await · Promises · Fetch · HTTP · JSON · API Validation · Browser Security**

> **The practical rule: Trust the types for your own code, and check everything that comes over the network.**

This lesson follows [Chapter 12 — Reviewing and Running DemoStore's Chat Service and Widget](12-review-run-chat-service-widget.md). The chat service is now answering requests over the web, and the widget exists in a customer's browser. An FDE needs enough frontend knowledge to understand and safely modify that browser code.

The original teaching sequence is deliberately small: **three concepts are enough to start reading browser code—types, async/await, and fetch**. This guide expands each into self-contained explanations, diagrams, worked examples, likely mistakes, security considerations, interview questions, and practical exercises.

**Fidelity note:** Exact lesson claims are distinguished from **illustrative examples** and **additional engineering recommendations**. The transcript discusses a reply with an answer, a human-support boolean and a session ID; it does not supply exact property names or complete production code, so the code below uses explicit example names.

---

# Part 1: Why an FDE Needs to Understand Browser Code

## 1. The Customer Does Not See FastAPI

In the last chapter, we reviewed a working **FastAPI chat service**. But DemoStore customers do not open the backend service or the generated API documentation. They see **a small chat window embedded in DemoStore's website**.

That widget runs in the customer's browser. The browser is responsible for displaying messages, accepting input, keeping its own local UI state, and sending requests to the chat service.

```text
Customer types a message
          ↓
DemoStore website chat widget
     (browser code)
          ↓ HTTP + JSON
DemoStore FastAPI chat service
     (server code)
          ↓
Agent and centralized model adapter
          ↓
Claude / approved tools
          ↓
FastAPI sends a response
          ↓
Browser displays the answer
```

The FDE's job crosses both sides of that boundary. They may not be a specialist frontend engineer, but they must be able to read a React component, change a few lines safely, and recognize when a correct-looking interface hides a data or security bug.

## 2. What TypeScript Is

**TypeScript is JavaScript with a system for describing and checking the shapes of values.**

JavaScript is the language browsers execute. TypeScript adds annotations and checks that help catch certain mistakes **before code runs**.

For this lesson, remember three items:

1. **Types** specify what kinds of values and objects code expects.
2. **`async` and `await`** make it possible to wait for slow operations without stopping the rest of the page.
3. **`fetch`** makes requests from the browser to the chat service.

These ideas are useful in **plain JavaScript**, **React**, **Vue**, **Angular**, and other browser frameworks. React's component syntax is not needed to understand the network boundary.

## 3. Compile-Time vs Runtime

TypeScript checking happens during development/build. The browser itself runs **JavaScript**.

A normal workflow is:

```text
Developer writes TypeScript
          ↓
Type checker finds mismatched values
          ↓
TypeScript is transformed to JavaScript
          ↓
Type annotations are removed
          ↓
Browser executes JavaScript
```

**Critical:** A successful TypeScript build does **not** mean data received over the network is valid. The type declarations do not become automatic runtime validators.

### Interview-ready distinction

**Question:** “Is TypeScript a different language that the browser runs?”

**Answer:** “The developer writes TypeScript, which has JavaScript syntax plus type annotations and checking. The build produces JavaScript for the browser; runtime JavaScript does not enforce the removed TypeScript types.”

---

# Part 2: Types Describe Individual Values

## 4. What Is a Type?

A type describes which values a variable, parameter or field is expected to hold. The transcript focuses on three primitive types:

| Type | Meaning | DemoStore example |
|---|---|---|
| `string` | Text | Customer message or session ID |
| `number` | Numeric value | Count of pending requests |
| `boolean` | `true` or `false` | Whether a human is needed |

The type annotation appears **after the name**, separated by a colon.

### Illustrative TypeScript code

```ts
const message: string = "Where is my order?";
const pendingCount: number = 2;
const needsHuman: boolean = false;
```

The colon notation tells the checker the intended shape. It is not a value assigned to the variable.

## 5. The Mistake the Type Checker Catches

Suppose a customer message must be text, but a later edit assigns a number.

```ts
let customerMessage: string = "Where is my order?";

// Type error: number is not assignable to string.
customerMessage = 12345;
```

The editor can highlight the assignment **before the application runs**.

The lesson contrasts this with plain JavaScript, where the language has no TypeScript checker enforcing the declared `string` contract. A particular bad JavaScript value may not throw immediately—it may cause a bug only when code later uses it as text—so the underlying lesson is **early static detection**, not that all such assignments necessarily crash browsers.

### Why is an early warning valuable?

If an incompatible value reaches a string operation, such as `.trim()`, the browser may fail at runtime:

```ts
function cleanMessage(message: string): string {
  return message.trim();
}
```

A numeric value has no normal string `trim()` method. TypeScript protects well-typed callers from supplying that value by mistake.

## 6. Type Inference

**Additional explanation:** TypeScript can often infer types without explicit annotations:

```ts
const greeting = "Hello"; // inferred as text
const attempts = 3;       // inferred as a number
```

You may see both annotated and inferred types in a real `Chat.tsx` component. The transcript explicitly teaches annotations, but knowing inference prevents confusion when an annotation is absent.

### Revision check

- [ ] I recognize `: string`, `: number`, and `: boolean`.
- [ ] I know type errors may be found before the browser runs.
- [ ] I do not confuse static checking with validating real network responses.

---

# Part 3: Interfaces Describe Whole Objects

## 7. Why a Primitive Type Is Not Enough

The customer message is one text value. A service reply contains **several pieces of information together**.

The transcript's reply interface describes **three fields**:

1. The answer, as **text**.
2. Whether the chat goes to a person, as **true/false**.
3. The **session ID**, also **text**.

An **interface** describes an object's expected fields and the type of each.

## 8. Example `ChatReply` Interface

The following property names are **illustrative**, not exact transcript source:

```ts
interface ChatReply {
  answer: string;
  needsHuman: boolean;
  sessionId: string;
}
```

Meaning:

- `answer` is the text displayed in the chat bubble.
- `needsHuman` tells frontend logic whether human support is indicated.
- `sessionId` identifies the conversation for subsequent requests.

Example object:

```ts
const reply: ChatReply = {
  answer: "I can help with your delivery question.",
  needsHuman: false,
  sessionId: "session-123"
};
```

The checker can flag missing properties or wrong values **when the code constructs or manipulates data under this interface**.

## 9. How the Interface Helps the Widget

Imagine a widget function:

```ts
function displayReply(reply: ChatReply) {
  appendAssistantBubble(reply.answer);
  saveSessionId(reply.sessionId);

  if (reply.needsHuman) {
    showHumanSupportOption();
  }
}
```

Here, `displayReply` expects the three declared fields. TypeScript can help catch mistakes such as accidentally using `reply.session` instead of `reply.sessionId` **within code that the checker sees**.

## 10. Why the Session ID Matters

The next customer message should go to the **same ongoing conversation**.

```text
First message
    ↓
Backend replies with sessionId = "session-123"
    ↓
Browser remembers "session-123"
    ↓
Second message sends same sessionId
    ↓
Backend finds the conversation history
```

This connects to Chapter 12, where the widget has a known **initial session-ID race**: if the first response has not arrived, there may be no ID yet. TypeScript can describe the session ID when present, but types alone do **not** solve that asynchronous workflow problem.

---

# Part 4: The Biggest Interface Misunderstanding

## 11. Does the Interface Validate Server Data?

**No. This is the most important warning in the lesson.**

The interface states what the frontend developer expects. It does **not** automatically examine JSON returned by the server and reject values that violate those expectations.

The TypeScript checker generally sees source code and declared types. It does not know that a deployed server has silently changed its response format.

### The key statement

> **An interface is a promise we make to ourselves, not a runtime check.**

## 12. Why TypeScript Interfaces Disappear

Type annotations are stripped during TypeScript-to-JavaScript transformation.

```ts
// Source TypeScript
interface ChatReply {
  answer: string;
  needsHuman: boolean;
  sessionId: string;
}
```

There is normally no corresponding JavaScript interface object left behind that checks incoming JSON.

```text
TypeScript interface
       ↓ build
No runtime interface validator
       ↓
Browser reads whatever JSON the server actually sent
```

That is why using an annotation such as `as ChatReply` is not verification.

## 13. The Renamed Human-Handoff Field Bug

The transcript's concrete failure scenario is:

- The frontend interface expects a field indicating **whether the chat goes to a person**.
- Someone **renames that field on the server**.
- The frontend TypeScript checker still passes: its local interface has not changed.
- The browser reads a field **that no longer exists**.
- The widget does not correctly tell the customer about the human handoff.

### Illustrative mismatch

Frontend expects:

```json
{
  "answer": "You can speak with support.",
  "needsHuman": true,
  "sessionId": "session-123"
}
```

Backend silently changes the name:

```json
{
  "answer": "You can speak with support.",
  "handoffRequired": true,
  "sessionId": "session-123"
}
```

The browser code might still read:

```ts
if (reply.needsHuman) {
  showHumanSupportOption();
}
```

But the actual response has `handoffRequired`, not `needsHuman`. The missing property becomes `undefined` at runtime; the condition is falsy and the UI fails to show the intended human-support information.

This is a **business-impacting contract mismatch**, not merely a naming preference.

### Connection to the previous chapter

Chapter 12 showed the human-support button was disabled during loading. This chapter shows **another way a customer-facing handoff can fail**: the UI trusts a server response shape that has changed.

Both failures can pass unrelated tests.

## 14. How to Prevent the Contract Mismatch

The lesson's requirement is:

**Keep the frontend and backend data shapes in agreement, and test them against the real service.**

A practical check should call a running backend and inspect the actual JSON fields used by the widget.

### Example runtime validator (additional engineering illustration)

```ts
function isChatReply(value: unknown): value is ChatReply {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const record = value as Record<string, unknown>;

  return (
    typeof record.answer === "string" &&
    typeof record.needsHuman === "boolean" &&
    typeof record.sessionId === "string"
  );
}
```

This is not required source code from the lesson. It demonstrates the difference between **declaring** a type and **checking values at runtime**.

A future production app might instead use a runtime schema library or generated API client based on FastAPI/OpenAPI, but those are **additional engineering options**, not steps the transcript says DemoStore implemented.

---

# Part 5: The Server and Browser Have Two Versions of the Same Shape

## 15. The Python–TypeScript Boundary

DemoStore's backend describes a chat reply with **Python models**. The browser describes the expected reply using a **TypeScript interface**.

So the same contract is written in two languages:

```text
FastAPI / Python                    Browser / TypeScript
Pydantic response model             interface ChatReply
  - answer: str                       - answer: string
  - needsHuman: bool                  - needsHuman: boolean
  - sessionId: str                    - sessionId: string
                 \                  /
                  Same network contract
```

The field names above are illustrative. The previous lesson also described a reply carrying **intent**, which is a different or extended response model from this teaching example. Do not assume the three-field TypeScript example is a complete literal copy of DemoStore's final FastAPI response.

## 16. Why One Shared Pydantic Class Helps but Does Not Eliminate This Risk

Earlier chapters emphasized **one Pydantic intent class** shared across Claude's structured output, the service response and FastAPI documentation.

That helps keep the **Python side** consistent.

This lesson adds another boundary: browser code still needs to **correctly match the server's actual JSON**.

Even a perfect Python class does not automatically update a handwritten TypeScript interface.

### The two rules together

1. **Within the backend:** avoid multiple handwritten copies of the same schema.
2. **Across the network:** verify the actual request/response contract used by the browser.

---

# Part 6: Async and Await — Slow Work Without Freezing the Page

## 17. Why a Chat Request Takes Time

A call to Claude can take several seconds. The lesson uses an example of **three seconds** while the model generates a response.

Imagine the entire page became unresponsive during those three seconds:
- The customer could not continue typing.
- Scrolling would appear frozen.
- The support button might feel broken.
- The website would deliver a poor experience.

Browser JavaScript uses asynchronous operations so it can start a request and continue handling other events.

## 18. The Meaning of `async`

A function marked `async` can use `await` inside it and normally returns a **Promise**.

```ts
async function getChatReply() {
  // Slow operation can be awaited here.
}
```

A **Promise** represents a result that may be available later.

This is useful for:
- HTTP requests.
- Reading response bodies.
- Other asynchronous operations.

## 19. The Meaning of `await`

`await` means:

> **Pause this async function until the awaited operation completes.**

It does **not** pause the entire website.

While one async function waits:
- The browser can render the interface.
- The user can scroll.
- Other event handlers may run.
- The chat widget can display a typing indicator.

### Illustration

```ts
async function handleMessage() {
  showTypingIndicator();

  const reply = await sendToChatService("Where is my order?");

  hideTypingIndicator();
  displayReply(reply);
}
```

While `sendToChatService` is waiting, the browser continues doing other work.

The source describes this as the function continuing **from where it paused** when the reply arrives.

## 20. Why `async` Is Different from Blocking

```text
User clicks Send
       ↓
async handler begins
       ↓
network request starts
       ↓
handler awaits
       ├── Browser remains interactive
       ├── Typing indicator can display
       └── Other interactions remain possible
       ↓
response becomes available
       ↓
handler continues after await
       ↓
UI displays response
```

This is a simplified conceptual sequence, not a claim about the exact JavaScript event-loop scheduling implementation.

## 21. The Missing-`await` Bug

The lesson identifies a frequent mistake: **forgetting `await`**.

```ts
async function sendMessage() {
  // Mistake: this is a Promise, not the finished chat reply.
  const reply = sendToChatService("Where is my order?");

  // Wrong assumption: expecting reply.answer already exists.
  // TypeScript can flag this when the function is correctly typed.
  console.log(reply.answer);
}
```

Instead:

```ts
async function sendMessage() {
  const reply = await sendToChatService("Where is my order?");
  console.log(reply.answer);
}
```

The missing-`await` version has a **Promise object**—effectively a placeholder for future work—not the response data. Trying to read answer text from it may produce `undefined` or trigger a type error, depending on the code.

The transcript's user-visible symptom is **an empty assistant bubble**.

## 22. How TypeScript Can Catch It

If `sendToChatService` returns `Promise<ChatReply>`, then:

```ts
const pending = sendToChatService("Hello");
// pending: Promise<ChatReply>
```

The checker knows `pending` is **not** a `ChatReply` and therefore does not expose ordinary `ChatReply` fields such as `answer`.

```ts
const finished = await sendToChatService("Hello");
// finished: ChatReply
```

This early detection is exactly the value of using explicit types around asynchronous network code.

## 23. Python Has `async` and `await` Too

The video notes that **Python uses the same keywords**, and DemoStore's chat service uses them.

The concepts are similar:
- Mark a function as asynchronous.
- Wait for slow work without treating it as a blocking computation.

However, Python and JavaScript have **different runtime models**. The lecture's aim is to recognize the familiar syntax and idea, not claim their internal event loops are identical.

---

# Part 7: Fetch — Calling the Chat Service

## 24. What Is `fetch`?

`fetch` is the browser API for making network requests, such as HTTP requests to DemoStore's **FastAPI chat service**.

The widget needs to send:
- The **service address**.
- An **HTTP method/action word**, in this case `POST`.
- **Headers** describing the content type.
- A **request body** containing JSON.

The model API is **not** the destination of this browser request.

## 25. The Request Sent by DemoStore's Chat Widget

The transcript specifies a **POST** request with JSON containing:
1. The customer's **message**.
2. The **session ID**.

An illustrative request body:

```json
{
  "message": "Where is my order?",
  "session_id": "session-123"
}
```

For a brand-new chat, session ID handling depends on the backend contract. The transcript does not give the exact nullable or optional field convention.

## 26. Full Worked `fetch` Example

```ts
interface ChatReply {
  answer: string;
  needsHuman: boolean;
  sessionId: string;
}

async function sendToChatService(
  message: string,
  sessionId: string
): Promise<ChatReply> {
  const response = await fetch("https://example.com/chat", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      message: message,
      session_id: sessionId
    })
  });

  if (!response.ok) {
    throw new Error(`Chat service returned status ${response.status}`);
  }

  const data: unknown = await response.json();

  if (!isChatReply(data)) {
    throw new Error("Unexpected chat response format");
  }

  return data;
}
```

**Important:** The URL and property names are placeholders. This example includes **additional runtime validation** through `isChatReply`, which is an engineering safeguard rather than source code from the transcript.

The source's essential flow is the **two awaits** plus a status check.

## 27. Why POST?

The frontend is **sending a new chat message** to the service.

HTTP `POST` commonly carries data in a request body and asks a service to process or create something. Here the action is to process a customer message within a conversation.

The transcript calls the HTTP method the **action word**.

## 28. What the JSON Header Means

```ts
headers: {
  "Content-Type": "application/json"
}
```

This declares that the request body's format is JSON.

The body must be serialized into text:

```ts
body: JSON.stringify({
  message,
  session_id: sessionId
})
```

An ordinary JavaScript object is not automatically equivalent to a JSON request body.

## 29. Why There Are Two `await` Expressions

The lesson deliberately emphasizes **two waits**.

### First await: receive the HTTP response

```ts
const response = await fetch(url, options);
```

This awaits the network operation and returns a **Response** object with status and headers.

### Second await: read and parse JSON

```ts
const data = await response.json();
```

This awaits the reading and parsing of the response body.

The first result is **not yet the parsed chat object**.

### Sequence

```text
fetch(url, options)
      ↓ await #1
Response object arrives
      ↓ inspect response.ok
response.json()
      ↓ await #2
Parsed JSON value
      ↓ validate actual shape
Display answer, update session
```

For this lesson, understanding the two distinct stages matters more than memorizing a single API call.

---

# Part 8: Fetch Error Handling — The Subtle Trap

## 30. Why `fetch` Can Be Misleading

The transcript says:

> **Fetch does not fail just because the server returns an HTTP error status.**

If a server sends an HTTP **500** response, the network request **completed**. So the Promise returned by `fetch` normally **resolves** to a Response containing that status.

A rejected Promise is more typical when the request fails at the network level—for example, when no response can be obtained because of a connection failure. Other request errors may also cause rejection; the key rule is that HTTP error status alone does not.

## 31. What Does `response.ok` Mean?

The Response object includes a field named **`ok`**.

`response.ok` is true for HTTP status codes from **200 through 299** (the **2xx** family).

| HTTP status | `response.ok` | Meaning for this lesson |
|---|---|---|
| 200 | `true` | Successful response |
| 201 | `true` | Successful result under an applicable endpoint |
| 400 | `false` | Client/request error |
| 401 | `false` | Authentication required / unauthorized |
| 404 | `false` | Requested resource missing |
| 500 | `false` | Server error |
| 503 | `false` | Service unavailable |

These status-code interpretations are additional HTTP background; the transcript explicitly illustrates **500** and the **2xx / `ok`** distinction.

## 32. The Mistake: Treating a 500 Body as an Assistant Reply

Bad flow:

```ts
const response = await fetch("/chat", options);
const data = await response.json();
showAssistantBubble(data.answer);
```

If the server returned a JSON error, the code might display an empty bubble or treat a technical error as a customer-facing chat answer.

Correct order:

```ts
const response = await fetch("/chat", options);

if (!response.ok) {
  showHelpfulFailureMessage();
  offerHumanSupport();
  return;
}

const data = await response.json();
showAssistantBubble(data.answer);
```

**The transcript specifically says to check the status first**, and not to display a failed response's body as though it were a valid chat answer.

## 33. What Should the Customer See?

When the status is not successful, the widget should:

1. **Say something went wrong**.
2. **Offer a person** instead of pretending the request succeeded.

Illustrative message:

> “I couldn't get an answer just now. Something went wrong. You can request help from a person.”

The exact wording is not given in the transcript.

### Connection to the prior chapter

The prior chapter found that the **“Talk to a person”** button was disabled while loading and had to be fixed.

This chapter explains why a path to a person must also be part of **error handling**. A failed API response should not leave the customer with a blank bubble and no alternative.

## 34. Handle Both Failure Categories

A robust browser handler distinguishes:

**A. Server responded with an error status**

```ts
if (!response.ok) {
  // 4xx or 5xx response: show controlled failure UI.
}
```

**B. The request itself failed or response parsing failed**

```ts
try {
  // fetch and parse
} catch (error) {
  // Network failure, invalid body, or other thrown error.
}
```

This combined approach is an additional practical recommendation. The transcript's main point is that **checking `response.ok` is required because an HTTP error does not automatically reject `fetch`**.

---

# Part 9: Browser Security and API Keys

## 35. Never Call Claude Directly From the Widget

The transcript states a firm rule:

> **The chat window calls our chat service, never the Claude API.**

Why?

Because **browser code is visible to the customer**. Code downloaded and executed on a user's device cannot safely contain a secret server API key.

A user can inspect browser assets and requests. If a key is embedded in frontend code, the key should be considered exposed.

## 36. Where the Claude API Key Belongs

The API key stays on **the server**, near the code that calls the model through the approved adapter.

```text
SAFE BOUNDARY

Browser widget
(no Claude secret)
       │
       │ request: message + session ID
       ▼
DemoStore chat service
(server-side credentials)
       │
       ▼
llm.py → Claude API
```

This matches earlier course rules requiring all model calls to go through `llm.py`.

### Bad architecture

```text
Customer browser
   └── Claude API directly
       └── secret key embedded in shipped JavaScript
```

The key would be available to anyone who inspected the client.

### Important clarification

Keeping the key server-side is necessary but not sufficient for a secure service. The backend must still enforce authentication, rate limits, authorization, privacy and tool permissions. These are broader engineering considerations from earlier chapters, not new controls shown as implemented by this TypeScript lesson.

---

# Part 10: A Practical Reading Method for Unfamiliar Browser Code

## 37. Three Things to Find First

The lesson proposes a simple order for reading browser code you do not know:

**First: Find the interfaces.**

These tell you what shapes the programmer expects: message, reply, session ID, escalation flag.

**Second: Find the `fetch` call.**

This reveals where the browser sends data and what HTTP method, headers and body it uses.

**Third: Follow the result to the screen.**

Find the code that reads the parsed reply, stores the session ID, adds a chat bubble and reacts to a handoff flag.

### Example reading map

```text
interface ChatReply
         ↓
async function sendMessage(...)
         ↓
await fetch(chatServiceUrl, {...})
         ↓
if (!response.ok) ...
         ↓
await response.json()
         ↓
use reply.answer
         ↓
save reply.sessionId
         ↓
update visible widget UI
```

The transcript says: **Find the interfaces, find the fetch, and follow the data to the screen.**

## 38. Additional Checks for an FDE Code Review

Once those basics are understood, ask:
- Is the reply **actually validated**, or is it only asserted to be a TypeScript interface?
- Does the code use **both awaits**?
- Does it check `response.ok`?
- Does a failure produce a human-readable error?
- Can the customer still reach a person while a request is pending?
- Is the session ID handled consistently?
- Is an API key or other secret accidentally shipped in the bundle?

These checks apply directly to DemoStore's earlier widget defect and remaining session-ID edge case.

---

# Part 11: Complete Worked DemoStore Request

## 39. Request/Response Journey

**Customer message:** “Where is my order?”

**Step 1 — Gather request data**

The widget has the customer's text and, if known, the current conversation ID.

```ts
const message: string = "Where is my order?";
const sessionId: string = "session-123";
```

**Step 2 — Send with `fetch`**

```ts
const response = await fetch("/chat", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    message,
    session_id: sessionId
  })
});
```

**Step 3 — Check HTTP status**

```ts
if (!response.ok) {
  throw new Error("The chat service did not return a successful status");
}
```

**Step 4 — Read JSON**

```ts
const raw: unknown = await response.json();
```

**Step 5 — Validate and display**

Use a shape check or established runtime schema before trusting unknown JSON.

```ts
if (!isChatReply(raw)) {
  throw new Error("The service returned an unexpected response");
}

appendAssistantBubble(raw.answer);
saveSessionId(raw.sessionId);
```

**Step 6 — Respond to handoff information**

```ts
if (raw.needsHuman) {
  showHumanSupportOption();
}
```

**Step 7 — Handle errors**

Show a safe message and offer a person rather than a blank chat bubble.

### What is example versus original source?

Every literal endpoint, response property and function name in this worked example is **illustrative**. The transcript's actual teaching points are the three-field interface concept, POST + JSON with message/session ID, two awaits, `response.ok`, the runtime-trust boundary, and keeping the Claude key on the server.

---

# Part 12: Seven Mistakes an FDE Must Spot

## 40. Pitfall Summary

| Mistake | What goes wrong | Practical safeguard |
|---|---|---|
| Assigning a number to a text variable | Wrong local value | TypeScript type checking |
| Assuming an interface validates server JSON | Renamed/missing fields go unnoticed | Runtime validation and real service contract test |
| Omitting `await` | Code reads Promise instead of reply | Await the async result, check return type |
| Awaiting `fetch` but not `response.json()` | Code receives Promise for parsed data | Use two awaits |
| Treating HTTP 500 as rejected `fetch` | Error body displayed as reply | Check `response.ok` |
| Freezing customer flow during a slow request | UI becomes unusable | Use asynchronous request handling |
| Calling Claude directly from browser | Secret API key exposed | Browser → backend → model |

### Two additional review risks from earlier chapters

**Human-support access:** Even correct `fetch` code cannot compensate for a disabled “Talk to a person” button.

**First-message session race:** Even a correctly typed `sessionId` field may be unavailable before the first response, resulting in messages going to different sessions.

Types help explain the expected structure; they do not enforce complete workflow behavior.

---

# Part 13: FDE Interview Questions and Answers

## 41. Main FDE Interview Question

**Q: How would you safely modify a TypeScript chat widget that calls an AI backend?**

**Strong answer:**

> “I would first locate the TypeScript interfaces to understand the expected message, reply and session shapes. Next I would find the asynchronous `fetch` call, verify that the request uses POST and JSON for the customer message and session ID, and check that both the HTTP response and its JSON body are awaited. I would ensure the code checks `response.ok`, because fetch resolves even for a 500 response, and displays a helpful error with a person option on failure. I would not trust a TypeScript interface to validate data from the server: I'd verify the runtime response shape and test the contract against the real service. Finally, I would keep model API keys server-side and test the actual browser interaction, particularly the customer’s ability to reach a person while waiting.”

## 42. Additional FDE Interview Questions

**Q1. What is TypeScript?**

JavaScript with type annotations and checking that can detect certain mistakes before execution.

**Q2. What does the browser actually execute?**

JavaScript. TypeScript is transformed and its type declarations are removed before the browser runs it.

**Q3. Which three concepts does this lesson teach?**

**Types**, **async/await**, and **fetch**.

**Q4. What does `: string` mean?**

The name is intended to hold a text value.

**Q5. What are three primitive types discussed?**

`string`, `number` and `boolean`.

**Q6. What does an interface describe?**

The expected fields and types of an object.

**Q7. Which three fields does the lecture's reply example contain?**

Answer text, human-handoff boolean and session ID text. Exact source property names are not supplied.

**Q8. Why does the widget need a session ID?**

To send later messages into the same conversation.

**Q9. Does a TypeScript interface validate the server response?**

**No.** It is a static description, not a runtime JSON validator.

**Q10. What happens to types during the build?**

They are removed from the emitted JavaScript.

**Q11. What is the server-field rename bug?**

The server renames the human-handoff field but the frontend interface still expects its old name, so the browser may fail to show the handoff correctly.

**Q12. Why might TypeScript still report no error after a backend field rename?**

Because its checker sees only the declared local interface, not the actual changed response from the deployed server.

**Q13. How can you detect a cross-service contract mismatch?**

Test against the real service and validate the actual parsed response shape.

**Q14. What does `async` mean?**

It marks a function that can await asynchronous work and returns a Promise.

**Q15. What does `await` do?**

It pauses the current async function until an awaited result is ready, while the rest of the page can remain responsive.

**Q16. Does `await` freeze the browser?**

No. It pauses that async continuation, not the entire page.

**Q17. Why does the chat show a typing indicator?**

The model call can take time; the indicator tells customers that a reply is pending.

**Q18. What is a Promise?**

An object representing a value that may become available later.

**Q19. What mistake occurs if you forget `await`?**

The variable contains a Promise rather than the actual response, possibly producing an empty chat bubble when code expects reply text.

**Q20. Can TypeScript help catch missing `await`?**

Often yes: a `Promise<ChatReply>` is not a `ChatReply`.

**Q21. What is `fetch`?**

The browser's API for making HTTP/network requests.

**Q22. What HTTP method does the widget use?**

**POST**.

**Q23. What does the widget send in JSON?**

Customer message and session ID.

**Q24. What is the purpose of `Content-Type: application/json`?**

It declares the request body's data format.

**Q25. Why use `JSON.stringify`?**

To encode a JavaScript object as JSON text for the request body.

**Q26. Why are there two awaits with fetch?**

One awaits the Response object; the second awaits parsing/reading its JSON body.

**Q27. Does HTTP 500 make `fetch` reject?**

Normally **no**. The network request completed and returns a Response with a failing HTTP status.

**Q28. When does `fetch` typically reject?**

When the request cannot complete at the network level, such as loss of connectivity, rather than for an ordinary HTTP error response.

**Q29. What does `response.ok` represent?**

`true` for HTTP status codes in the **200–299** range.

**Q30. What should the widget do on a failed HTTP status?**

Not treat the error body as a normal assistant answer; display a safe failure message and offer a person.

**Q31. Is TypeScript enough to validate response JSON?**

No. Use an actual runtime shape check and/or integration contract test.

**Q32. Why shouldn't the browser call Claude directly?**

The browser's JavaScript and requests are inspectable; embedding a Claude API key exposes the secret.

**Q33. Where should the Claude API key stay?**

On the backend server, near the centralized model adapter.

**Q34. What are the three steps to reading unfamiliar browser code?**

Find the interfaces, find the fetch call, and follow the returned data to the screen.

**Q35. Does React change how JavaScript's async/await works?**

No. The same core JavaScript concepts apply to React, Vue, Angular and plain JavaScript.

**Q36. What does a frontend build prove?**

The configured code compiles/type-checks; it does not prove server data is valid or the user interaction works.

**Q37. How is this lesson connected to the earlier shared Pydantic intent class?**

The backend class keeps its Python-side shapes consistent, but the frontend must still match and validate the actual network contract.

**Q38. What important bug did the previous chapter find in the widget?**

The person button was disabled during loading; TypeScript compilation did not catch the behavioral violation.

**Q39. What two categories of checks are needed for safe frontend changes?**

Static code/type checks and **runtime/real-service/browser behavior checks**.

**Q40. State the practical rule.**

**Trust the types for your own code, and check everything that comes over the network.**

---

# Part 14: Quick Knowledge Check

## 43. Six Questions

Answer before opening the explanation.

<details>
<summary><strong>1. Which statement about TypeScript interfaces is correct?</strong></summary>

**A.** They validate all JSON returned by the server automatically.

**B.** They describe expected shapes during development, but do not validate network data at runtime.

**C.** They run as JavaScript checks in the browser.

**Correct answer: B.** Type annotations are removed during the build.

</details>

<details>
<summary><strong>2. What does `await` do inside an async function?</strong></summary>

**A.** Freezes the whole browser page.

**B.** Pauses the async function until the awaited result is available, while the page can keep working.

**C.** Converts the service response into a string.

**Correct answer: B.**

</details>

<details>
<summary><strong>3. What does a missing `await` often produce?</strong></summary>

**A.** The final chat reply.

**B.** A Promise representing a future reply.

**C.** A guaranteed 500 response.

**Correct answer: B.**

</details>

<details>
<summary><strong>4. Which two waits are needed in the usual fetch-and-JSON flow?</strong></summary>

**A.** Await `fetch(...)` and await `response.json()`.

**B.** Await the API key and await TypeScript compilation.

**C.** Await `JSON.stringify(...)` twice.

**Correct answer: A.**

</details>

<details>
<summary><strong>5. The server responds with HTTP 500. What should happen?</strong></summary>

**A.** `fetch` automatically rejects in every case.

**B.** The browser displays the response body as a normal assistant answer.

**C.** The code checks `response.ok`, shows a controlled error and offers a person.

**Correct answer: C.**

</details>

<details>
<summary><strong>6. Where does the Claude API key belong?</strong></summary>

**A.** In frontend code, because TypeScript hides it.

**B.** On the backend server, not in browser-delivered code.

**C.** In the chat request JSON.

**Correct answer: B.**

</details>

---

# Part 15: Complete Revision Checklist

## 44. Course Context

- [ ] The service works over the web, but the customer sees the **website widget**.
- [ ] Browser code is often written in **TypeScript**.
- [ ] An FDE must be able to **read and safely modify** browser code.
- [ ] TypeScript is **JavaScript with type annotations and checking**.
- [ ] Three lesson topics: **types, async/await, fetch**.
- [ ] Concepts apply to plain JavaScript and **React/Vue/Angular**.

## 45. Basic Types

- [ ] A type describes the kind of value a name can hold.
- [ ] An annotation follows the name with a **colon**.
- [ ] `string` means text.
- [ ] `number` means numeric value.
- [ ] `boolean` means true/false.
- [ ] A wrong type can be highlighted before running.
- [ ] TypeScript checking is **not** proof that runtime data is correct.

## 46. Interfaces and API Contracts

- [ ] An **interface** describes the fields and types of an object.
- [ ] Example chat reply contains **answer text, human flag, session ID**.
- [ ] Session ID associates follow-ups with the same chat.
- [ ] The backend describes the reply in **Python**.
- [ ] The browser describes the reply in **TypeScript**.
- [ ] Types are **removed** when converted to JavaScript.
- [ ] Interfaces do **not** validate actual response JSON.
- [ ] A server field rename can break frontend behavior while type checking passes.
- [ ] The handoff-field mismatch is a serious **customer-impacting bug**.
- [ ] Test against the **real service contract**.
- [ ] Use runtime validation for **untrusted network data** where appropriate.

## 47. Async and Await

- [ ] Model calls can take **several seconds** (lesson example: three seconds).
- [ ] The page should remain **interactive**.
- [ ] `async` marks an asynchronous function.
- [ ] `await` pauses only the current async continuation.
- [ ] A **Promise** represents a future result.
- [ ] A missing `await` can yield a Promise instead of a reply.
- [ ] Reading reply text from a Promise can produce an **empty bubble**.
- [ ] TypeScript can often catch a Promise/Reply mismatch.
- [ ] The widget uses a **typing indicator** while waiting.
- [ ] Python also uses `async` and `await`.

## 48. Fetch Requests

- [ ] `fetch` makes requests from the browser to a service.
- [ ] DemoStore sends an HTTP **POST**.
- [ ] Request contains address, method, headers and body.
- [ ] Header `Content-Type: application/json` identifies JSON.
- [ ] Body includes **customer message and session ID**.
- [ ] `JSON.stringify` creates JSON request text.
- [ ] **First await:** `fetch(...)` returns a Response.
- [ ] **Second await:** `response.json()` gives parsed data.
- [ ] An HTTP **500** does **not automatically reject fetch**.
- [ ] `response.ok` is **true for 2xx status codes**.
- [ ] Check `ok` **before treating JSON as a chat answer**.
- [ ] On error, show a clear failure message and **offer a person**.
- [ ] Network errors also need controlled handling.

## 49. Browser Security

- [ ] Browser code is visible to users.
- [ ] The widget calls **DemoStore's chat service**, never Claude directly.
- [ ] The **Claude API key stays on the server**.
- [ ] Backend keeps provider calls behind `llm.py`.
- [ ] API secrets do not belong in browser JavaScript or request bodies.
- [ ] Backend authorization is still required independently of frontend types.

## 50. Reading Code Safely

- [ ] **Find the interfaces** to understand expected data.
- [ ] **Find fetch** to understand the network request.
- [ ] **Follow the data to the screen** to understand user-visible behavior.
- [ ] Verify that parsed JSON actually matches expectations.
- [ ] Check asynchronous state and missing awaits.
- [ ] Check error status handling.
- [ ] Confirm human-support controls remain useful during failure.
- [ ] Test the frontend with the **real backend** when feasible.

---

# Part 16: Final Summary and Practice Task

## 51. One-Page Final Summary

| Topic | Key lesson |
|---|---|
| TypeScript | JavaScript with static types |
| Types | Describe text, numbers and booleans |
| Interface | Describes an object's expected shape |
| Runtime truth | Interfaces disappear; JSON still needs verification |
| Example contract bug | Server renames human-handoff field; UI silently misses it |
| Async | A function can await slow work without freezing the page |
| Promise | Represents a future result |
| Missing await | Code uses Promise instead of actual chat response |
| Fetch | Browser HTTP request to chat service |
| Request | POST with JSON message and session ID |
| Two awaits | Receive Response; parse JSON |
| `response.ok` | True for 2xx status codes |
| HTTP 500 | Usually resolves fetch but indicates an error |
| Error UX | Don't show error as reply; offer a person |
| API key | Server-side only |
| Reading method | Interfaces → fetch → data to screen |
| Practical rule | Trust own types; validate network data |

## 52. Practice Task: Trace One Message Through a Widget

Open the TypeScript code of a chat widget, ideally DemoStore's `Chat.tsx`.

1. Find the **response interface**. Write out what every field means.
2. Find the **`fetch`** call. Identify the URL, method, headers and JSON body.
3. Highlight both **`await`** expressions.
4. Find the **`response.ok`** check. If absent, explain what a 500 response might do.
5. Follow the resulting JSON to the **assistant chat bubble**.
6. Locate the place where the **session ID** is stored for the next message.
7. Check that there is **no model API key** in browser code.
8. Simulate or inspect a server response with the **human-handoff field renamed**; explain why static types alone would not detect it.
9. Verify the widget gives a **clear error and a path to human support** when the API fails.

### What your FDE answer should demonstrate

You don't have to write a whole frontend framework from memory. You do need to follow a data contract, recognize asynchronous work, understand the browser-to-server boundary, and check the critical customer-facing behavior.

> **Practical rule: Trust the types for your own code, and check everything that comes over the network.**

**Next lesson:** Write the specification that tells the coding agent what to build next.
