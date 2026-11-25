# Complete Study Notes: Automated Code Checks and Git Pre-Commit Hooks for Agentic Development

**FDE Course · AI SDLC · Claude Code · Pytest · Ruff · Mypy · TypeScript TSC · Pre-Commit · Git Hooks · CI · Pull Requests · Testing with Fakes · Permissions · Code Review**

> **Practical rule: Don't ask for a check. Make the code run it.**

This chapter follows [Chapter 14 — Two-Voice Specifications and Traceability](14-two-voice-specifications.md). Chapter 14 linked business requirements to technical rules and tests. This chapter addresses the next problem: **a test doesn't help if nobody runs it**.

When a coding agent writes and commits much of the code, an FDE cannot rely entirely on a prompt saying “always run the tests.” Instead, put **automatic, executable checks** in the development workflow. These checks help both humans and coding agents catch defects before a commit is recorded.

**Source fidelity:** The tool names, the examples involving unverified orders and a missing order, the fast-versus-slow distinction, the permission rule, and the practical takeaway come from the supplied transcript. Configuration files, shell commands, type signatures and test code below are **illustrative engineering examples**, not copied from the course repository.

---

# Part 1: What Problem Does This Lesson Solve?

## 1. Tests Exist, but Do They Run?

The specification now links a business criterion such as:

> A customer without completed email one-time-code verification sees **no order data**.

to a technical guardrail and a test. This is a good beginning, but a passing test from yesterday does not tell us whether code changed today is safe.

The course points out a particular risk:

**A coding agent writes much of our code, so we cannot count on it remembering to execute every check.**

Even a diligent human developer can forget. An AI coding agent may run out of context, interpret a request too broadly, skip a command or simply **report that checks passed without having run them**.

Therefore the key change is from **requesting good behavior** to **making the workflow enforce it**.

## 2. The Three Parts of the Lesson

1. **Three kinds of checks:** tests, linting and type checking.
2. **A pre-commit hook:** a program that runs the fast checks before a Git commit.
3. **Enforce with code, not only prompts:** settings prevent known bypasses, while human review still examines suspicious changes.

```text
Developer or coding agent edits code
                ↓
Attempts git commit
                ↓
Git pre-commit hook starts
      ├── Behavior tests
      ├── Static linter
      └── Static type checker
                ↓
          Any failure?
       ├── Yes → block commit; show errors
       └── No  → commit can proceed
                ↓
        Pull request in GitHub
                ↓
       Slower/heavier checks
                ↓
        Human code review
```

The hook is a **gate**, not a replacement for tests, CI or review.

---

# Part 2: Check One — Tests

## 3. What Is a Test?

A **test** executes a small piece of code and compares its actual behavior with an expected result.

At DemoStore, one example test sends a request for an existing order from a chat session that **has not passed the email one-time-code check**.

Expected result:

> **No protected order information is returned.**

The test is tied to the business acceptance criterion from the spec. It does not merely confirm that a function returns a value.

### The full logic

```text
Business rule:
Unverified customer gets no order data
           ↓
Application logic:
Check session verification before order lookup
           ↓
Test setup:
Known order exists in fake Ledgerline;
session has no verified customer
           ↓
Action:
Ask about that order
           ↓
Expected:
No protected order details
```

## 4. Illustrative Pytest Example

```python
# Example only: actual repository APIs may differ.
def test_unverified_customer_cannot_read_order(fake_ledgerline):
    session = {"verified_customer": None}

    response = find_order_for_session(
        order_number="DEMO-100",
        session=session,
        ledgerline=fake_ledgerline,
    )

    assert "order_details" not in response
    assert response.get("error") == "verification_required"
```

The key assertion is about **access control**, not wording. In the actual implementation, the error may be represented differently.

### Why use a *known* existing order?

If the test requests a nonexistent order, the tool might return not-found for reasons unrelated to security. A test using an existing fixture checks that **verification is the reason the data was withheld**.

## 5. What Tests Are Best at Catching

- Wrong branching logic.
- Authorization controls that no longer work.
- Incorrect API output.
- Bugs in parsing or transforming order data.
- A mismatch between the business requirement and observed result.

### What tests may miss

A test only checks the behavior it exercises. The course uses another example: a rarely executed **Ledgerline error branch**. If tests never enter that branch, a basic typo inside it can remain undiscovered.

That is where linting helps.

---

# Part 3: Check Two — Linting

## 6. What Is a Linter?

A **linter** reads source code and looks for patterns that commonly indicate mistakes.

It usually does this **without executing the program**.

The transcript's example is a variable with **one incorrect letter** inside a code branch that handles a Ledgerline error.

That branch rarely runs in tests, so a normal test suite may never encounter the typo.

The linter can still spot a name that is **not defined**.

### Illustrative code

```python
def format_ledgerline_error(error_message: str) -> str:
    # Bug: variable name misspelled.
    return f"Order service error: {error_mesage}"
```

The parameter is `error_message`, but the function body refers to `error_mesage`.

A linter can flag the undefined reference before a customer encounters the error path.

## 7. Which Linter Is Used?

The lesson names **Ruff** for Python.

Illustrative invocation:

```bash
ruff check .
```

Ruff can detect many likely problems, including undefined names, unused imports and other configurable patterns.

**Important:** Different Ruff rule sets check different patterns. A green lint result is not proof of business correctness.

## 8. Why Lint When Tests Already Exist?

Because a test suite covers **selected executions**. A linter reviews **source patterns**, including code that tests might not reach.

The tools complement each other:

| Test | Linter |
|---|---|
| Runs code | Usually analyzes code without running it |
| Needs inputs and expectations | Uses static code patterns and rules |
| Finds wrong observed behavior | Finds suspicious code structure and likely bugs |
| Coverage depends on test scenarios | Can examine otherwise-unexecuted branches |

The error-branch typo is the memorable example of why **linting earns a place alongside tests**.

---

# Part 4: Check Three — Static Type Checking

## 9. What Is a Type Checker?

A **type checker** reads code and reasons about how values are used based on type information.

Like a linter, it usually works **without running the code**.

A type checker asks questions such as:

- Is this value always a string?
- Could the value be missing?
- Does this function return the type the caller expects?
- Is this object used safely after a possible failure?

## 10. Python Has Type Hints Too

The previous TypeScript lesson showed how to describe data shapes. Python offers a related mechanism using **type hints**.

```python
def format_status(status: str) -> str:
    return status.upper()
```

The notation `status: str` expresses an expected type, and `-> str` describes the return type.

The course stresses a crucial difference:

**Python itself normally does not enforce these type hints when the function runs.** A separate type checker reads them and identifies incompatible uses.

This parallels the previous chapter's rule that TypeScript interfaces are **not runtime JSON validators**.

## 11. The DemoStore Missing-Order Example

The transcript provides a concrete danger.

A function that looks up an order returns:
- An order object **when found**, or
- **Nothing** when not found.

Later code reads the order's `status` without checking whether the order exists.

For a valid order, the code works. For another customer's order number, the lookup can return nothing, and the status access crashes.

### Illustrative Python model

```python
from dataclasses import dataclass

@dataclass
class Order:
    number: str
    status: str

def find_order(order_number: str) -> Order | None:
    # May return no visible order.
    ...
```

The annotation `Order | None` means **either an Order or no value**.

### The unsafe caller

```python
order = find_order("DEMO-100")
print(order.status)  # Unsafe: order might be None.
```

A type checker can flag this because `None` has no `status` attribute.

### The safer caller

```python
order = find_order("DEMO-100")

if order is None:
    print("Order not found or unavailable")
else:
    print(order.status)
```

The guard narrows the type: after the explicit `None` check, the remaining branch can safely treat `order` as an Order.

## 12. Why This Bug Matters for Security

The “missing order” condition is not just an ordinary database miss. Earlier DemoStore lessons intentionally make **another customer's order** appear the same as a nonexistent order.

Therefore the code must safely handle **no visible order** during authorization failures.

A crash in that case could:
- Interrupt support flow.
- Produce poor customer experience.
- Reveal technical error details if error handling is unsafe.

Type checking helps prevent a class of these bugs **before those negative cases are run**.

## 13. Which Python Checker Is Named?

The lesson uses **mypy**.

Illustrative command:

```bash
mypy .
```

Actual project configuration may limit the checked directories or require settings for third-party libraries. The transcript does not specify its exact flags.

## 14. What About TypeScript?

For the browser widget, the course names **`tsc`**, the TypeScript checker.

An illustrative command is:

```bash
npx tsc --noEmit
```

The `--noEmit` option runs checking without generating JavaScript output, when supported by the project's TypeScript setup.

For `Chat.tsx`, `tsc` can catch incorrect property types and missing compatible assignments **inside typed frontend code**.

However, remember from the previous chapter:

**TypeScript checking does not prove that the actual JSON from the backend matches the declared interface.** A real-service integration test or runtime validation is still needed.

## 15. The Three Checks Together

| Check | Core question | Python tool | Browser/frontend equivalent |
|---|---|---|---|
| **Test** | Does the code produce the expected behavior? | **pytest** | Frontend tests when configured |
| **Lint** | Does the source contain suspicious patterns? | **ruff** | ESLint or another linter, if chosen |
| **Type check** | Are values used according to their types? | **mypy** | **tsc** |

The transcript specifically names **pytest, ruff, mypy**, and **tsc**. Other languages and stacks offer corresponding tools.

### Memory anchor

**pytest = behavior. Ruff = likely code mistakes. mypy/tsc = type compatibility.**

---

# Part 5: Pre-Commit Hooks

## 16. What Is a Git Hook?

A **hook** is a small program triggered by Git at a specific moment.

A **pre-commit hook** runs **before Git saves a commit**.

The exact idea is:

> **If any required check fails, the commit does not happen.**

This turns a request into an automatic step in the development workflow.

## 17. How a Hook Changes the Process

Without a hook:

```text
Agent edits code
    ↓
Agent may or may not run tests
    ↓
Agent commits
    ↓
Bug may enter history unchecked
```

With a hook:

```text
Agent edits code
    ↓
Agent attempts commit
    ↓
Git launches checks automatically
    ├── pytest
    ├── ruff
    └── mypy / frontend check as configured
    ↓
Any check fails?
  ├── Yes: commit blocked, error printed
  └── No: commit proceeds
```

The same mechanism works when **a human** commits and when **Claude Code** commits.

## 18. The `pre-commit` Tool

The transcript names the small tool **pre-commit**. It helps manage Git hooks using a **short configuration file committed in the repository**.

An illustrative configuration file might be named:

```text
.pre-commit-config.yaml
```

The course does not provide its exact contents. Its purpose is to define **which fast checks run before every commit**.

## 19. Illustrative Hook Configuration

This is a **conceptual example** of using local hooks. The exact commands and environment require adjustment for a real project.

```yaml
repos:
  - repo: local
    hooks:
      - id: python-fast-tests
        name: Fast Python unit tests
        entry: python -m pytest -m "not live" -q
        language: system
        pass_filenames: false
        always_run: true

      - id: python-lint
        name: Ruff lint
        entry: ruff check .
        language: system
        pass_filenames: false
        always_run: true

      - id: python-typecheck
        name: Mypy type check
        entry: mypy .
        language: system
        pass_filenames: false
        always_run: true
```

**Important caveat:** `language: system` means these programs must be installed in the environment. A team may instead use managed environments, pinned versions and narrower paths. The transcript's requirement is the **automatic fast gate**, not this exact YAML.

An illustrative installation step:

```bash
pre-commit install
```

Merely adding the YAML file to Git does not necessarily install hooks on every developer's machine. A real onboarding/run guide needs to ensure the hook is activated.

## 20. How the Agent Receives Feedback

Imagine Claude Code tries to commit a phase containing the unsafe missing-order status access.

1. Agent edits source.
2. Agent requests a Git commit.
3. Hook starts.
4. **mypy** finds unsafe access to an order that may be `None`.
5. Hook exits unsuccessfully.
6. Git blocks the commit.
7. Error output returns to the coding agent.
8. Agent can inspect, fix the code, rerun and retry.

This is more reliable than hoping the agent noticed the same problem while writing code.

### Example failure output (illustrative)

```text
app/orders.py:42: error:
Item "None" has no attribute "status"

mypy: failed
Commit blocked.
```

No actual transcript error message is quoted here; the example shows the kind of feedback.

---

# Part 6: Keep the Hook Fast

## 21. Why Speed Is a Product Requirement for Developer Tooling

The course makes a practical point:

**A pre-commit hook should finish in seconds.**

If a commit takes minutes, the workflow becomes frustrating. Developers and agents are more tempted to skip the checks.

A hook that people habitually bypass may offer less protection than a short, dependable one.

## 22. What Belongs in the Fast Gate?

The lecture places **quick, deterministic checks** in the hook:

- Fast tests using **fake backends**.
- Ruff lint checks.
- Mypy type checks.
- Relevant frontend static checking, such as `tsc`, when practical and fast enough.

The exact tool selection depends on repository size and timing; the principle is to keep the hook reliably short.

## 23. What Must Not Run in Every Commit?

The source gives a specific expensive example:

> Replaying **40 labeled customer conversations against the real model** before every commit.

Why avoid that in the hook?
- Many real API calls.
- Minutes of waiting.
- Money spent on inference.
- Network and provider dependence.
- Friction that encourages bypasses.

These are valuable **evaluation tests**, but they belong to a slower stage.

## 24. Use Fake Ledgerline and Carrier Services

For the fast test suite, the transcript specifically says:

**Use fake versions of Ledgerline and the carrier—never real systems and never Claude.**

This means the developer can run many tests without:
- Production database access.
- Real customer data.
- Carrier outages.
- Model API charges.
- External latency.

### Illustrative fake

```python
class FakeLedgerline:
    def find_order(self, order_number: str):
        if order_number == "DEMO-100":
            return {"status": "shipped", "customer_id": "C-1"}
        return None
```

A real fast unit test can inject this fake into the order service and check its behavior. The code above is an example, not the exact project implementation.

## 25. Where Do the Slow Checks Run?

**On every pull request in GitHub**, says the transcript.

That allows comprehensive checks without making each commit slow.

The slower stage can include:
- Live Claude integration tests.
- The **40 labeled conversations** evaluation.
- Broader integration checks.

The transcript specifically identifies slow checks running **on every pull request**. It does not give an exact GitHub Actions workflow file, provider credentials setup or whether every named slow test runs under all conditions.

### Two-tier validation model

```text
LOCAL COMMIT
   │
   ├── quick unit tests
   ├── lint
   └── type checking
   │
   ▼
Git commit
   │
   ▼
GITHUB PULL REQUEST
   │
   ├── slower integration checks
   ├── model/evaluation checks
   └── independent code review
   │
   ▼
Merge only after accepted checks
```

### Critical limitation

Pre-commit hooks operate in local development environments and can be bypassed or not installed. For a strong enforcement policy, protected branches and required CI checks can provide **server-side gates**. This is additional engineering advice; the transcript's explicit steps are the local hook and PR-level slow checks.

---

# Part 7: Why Code Enforces Better Than a Prompt

## 26. The Weak Approach: Instruction Only

A team writes in `CLAUDE.md`:

> “Always run tests before you commit.”

The project rule may influence agent behavior, but it cannot physically prevent the Git commit.

A coding agent might:
- Forget during a long phase.
- Misinterpret what is required.
- Run only some checks.
- Confidently report that tests passed when they did not run.

Earlier lessons already showed a similar failure: **Claude Code said a plan file had been saved, but the file contained only its title**.

The fact that an agent reported success is not enough.

## 27. The Stronger Approach: A Hook

The pre-commit hook **runs because Git invokes it**, not because the model remembers a rule.

It is the same for people and coding agents.

That turns a fragile process expectation into a **local executable gate**.

### The general security-engineering pattern

The course explicitly connects this to earlier guardrails:

> **Code enforces a rule; the prompt only explains it.**

Order ownership is checked in the tool implementation, not trusted to the model's instruction. Similarly, tests and static checks are tied to the commit process rather than merely requested in the prompt.

## 28. A Comparison

| Prompt-only rule | Hook-backed rule |
|---|---|
| “Please run pytest” | Git invokes configured checks |
| Depends on remembering | Starts at the commit event |
| May be claimed without proof | Actual command exit status blocks the commit |
| Agent may omit an error | Hook returns failure output |
| Can be ignored by the model | Requires a deliberate bypass or missing hook installation |

**Do not overstate the protection:** Git does have ways to bypass local hooks, and tests can be weakened. The next parts address those limits.

---

# Part 8: Prevent the Known Bypass

## 29. Git's Hook-Bypass Option

Git supports a way to skip commit hooks. A common example is:

```bash
git -c core.hooksPath=/dev/null commit ...
```

and the conventional commit bypass option:

```bash
git commit --no-verify
```

These examples identify known bypass mechanisms; the transcript refers to **an extra Git option that skips the hook**, without naming the exact option. The course's requirement is to prevent the coding agent from using the bypass.

## 30. Agent Tool Permissions Can Deny the Bypass

Earlier lessons described a settings file that groups commands into:
- Allowed.
- Ask-first.
- Denied.

The lecture says to put **the hook-bypass command in the group that never runs**.

That makes it less likely that the coding agent can simply bypass validation when a commit fails.

### Why settings rather than CLAUDE.md?

A statement in `CLAUDE.md` is an **instruction**.

A deny rule in the tool's permission system is an **enforced restriction**, subject to how that tool interprets commands.

Use permission configuration to block bypass routes rather than relying on prose.

### Important engineering caution

A deny rule for one exact bypass command is not a universal guarantee that every possible bypass is impossible. The strongest practical setup also uses:
- Required CI checks on pull requests.
- Protected branch rules.
- Review of configuration changes.
- Controlled permissions for agent-executed Git commands.

These are additional safeguards, not all specified in the original transcript.

---

# Part 9: What a Green Hook Does *Not* Prove

## 31. Checks Ran ≠ Code Is Correct

The transcript explicitly warns:

> **A green hook proves the checks ran, not that the code is right.**

This is one of the most interview-worthy distinctions.

Tests are written by people and coding agents; they can contain mistakes, miss scenarios or be altered to pass incorrectly.

A linter cannot verify customer policy. A type checker cannot prove all authorization semantics. A passing test suite can be weak.

## 32. The Test-Weakening Risk

A stuck coding agent may “solve” a failing test by **changing the test**, not the application.

Illustrative example:

```python
# Strong assertion:
assert unverified_result.order_details is None

# Weak, dangerous change:
assert unverified_result is not None
```

The second assertion no longer checks that private data is hidden.

A green hook with the weaker test is not a valid replacement for the original requirement.

## 33. The Checker-Suppression Risk

The agent may also insert a suppression comment telling a checker to ignore a line.

An illustrative Python example:

```python
print(order.status)  # type: ignore[union-attr]
```

That can silence a type checker without handling the possibility that `order` is `None`.

### Why this matters

The code may crash for a foreign or missing order, but the type checker no longer reports it.

Suppressions can be legitimate in tightly reviewed situations; the problem is **unjustified suppression to make a gate pass**.

## 34. The Required Pull-Request Review

The transcript says:

**Every such edit needs a reason in the pull request, and the team reviews it closely.**

“Such edits” include:
- Changed test expectations.
- Deleted or skipped tests.
- Added checker suppressions.
- Changes that remove existing safeguards to obtain a green result.

### A strong review question

> “Did we fix the production code so it satisfies the business rule, or did we make the check weaker?”

This is particularly important when a coding agent generated the change.

## 35. What Should a Human Reviewer Inspect?

| Change | Reviewer asks |
|---|---|
| Test edited | Does it still assert the original business requirement? |
| Test removed or skipped | What valid reason justifies losing coverage? |
| `# type: ignore` added | Why can't the code handle the missing value properly? |
| Ruff suppression added | Which rule is suppressed, and is there a safe alternative? |
| Hook configuration changed | Does the fast gate still run? |
| Model evaluation changed | Were difficult chats or escalation cases removed? |
| Auth logic changed | Can an unverified or foreign customer now see an order? |

**A passing check is part of the evidence. A reviewed specification and meaningful assertions define whether that evidence is sufficient.**

---

# Part 10: Add Hooks Early in the Prototype

## 36. Why the First Day Matters

The lesson concludes with a practical timing recommendation:

> **Add the hook on day one of the prototype.**

Why so early?

If the team waits for months, adding lint and type checking may reveal **hundreds of old warnings at once**.

That creates a disruptive cleanup project. Developers may be tempted to suppress warnings or disable checks rather than fix the backlog.

## 37. Benefits of Day-One Validation

- The codebase starts with a small, manageable baseline.
- Every future phase has a consistent gate.
- Coding agents receive feedback immediately.
- The team avoids merging months of avoidable defects.
- The configuration becomes part of the project's normal setup.

### Practical framing for an FDE

A short prototype still benefits from lightweight guardrails. Quality gates should be **cheap to run, present early and connected to real business risks**.

This is consistent with the earlier five-day prototype lesson: fake surrounding systems, but do **not** fake security or correctness requirements.

---

# Part 11: A Full DemoStore Workflow Example

## 38. One Phase Through the Check Pipeline

Assume Claude Code implements the order-status service phase.

**Step 1 — Change:** It modifies the order lookup function.

**Step 2 — Bug:** It writes `order.status` without checking whether the order exists.

**Step 3 — Commit attempt:** Claude Code invokes Git.

**Step 4 — Automatic pre-commit gate:**

```text
pytest (fast fake-backed tests)
    ✓ authorization behavior remains tested

ruff
    ✓ no detected undefined variable

mypy
    ✗ order may be None; unsafe status access
```

**Step 5 — Commit rejected:** The agent receives a checker error.

**Step 6 — Repair:** It adds an explicit missing-order check.

**Step 7 — Re-run:** The hook passes.

**Step 8 — Commit:** Git records the checked change.

**Step 9 — Pull request:** Slower checks and human review examine the complete work.

This is an **illustrative walkthrough** that combines the transcript's examples. It is not a claim about an actual failed DemoStore commit captured in the video.

## 39. What If the Agent Weakens a Test Instead?

Suppose it changes the unverified-order test so that it only checks for *some* response, not for an absence of order data.

The fast hook may now pass, but the business acceptance criterion is no longer being checked.

At review, compare the test's assertion with the traceable spec line:

```text
Business requirement:
No order data before OTP verification
            ↓
Expected test:
Unverified real-order attempt returns no protected fields
            ↓
Actual modified test:
Only checks response is non-null
            ↓
FAILS REVIEW even if the hook is green
```

**Why the two-voice spec matters:** It gives the reviewer a stable source against which to evaluate the **quality of the check itself**.

---

# Part 12: A Minimal Practical Configuration Strategy

## 40. Fast Local Checks

A lightweight starter might run:

```bash
python -m pytest -m "not live" -q
ruff check .
mypy .
```

For the frontend, when relevant and fast enough:

```bash
npx tsc --noEmit
```

These are **illustrative command choices**; actual project path/configuration matters. An FDE should not force every check across an enormous monorepo without measuring runtime.

## 41. Slow GitHub Pull-Request Checks

In CI, the team can run heavier validation, such as:
- Real Claude calls that are explicitly allowed.
- Replay of the **40 labeled support conversations**.
- More comprehensive integration tests.
- Builds of frontend and backend artifacts.
- Code review of tests and checker configuration.

The transcript says slower checks run on **every GitHub pull request**, not before every local commit.

### Cost/security considerations

Do not give real model credentials to arbitrary untrusted pull-request code without an approved workflow. A safe CI design restricts where secrets are available. This is an additional operational caution.

## 42. Why Not Run Live Tests Locally on Every Commit?

Because a commit might happen many times in a phase. Each live evaluation would:
- Consume API usage.
- Add latency.
- Depend on network availability.
- Disrupt iteration.

A pre-commit gate that takes seconds has a much higher chance of being used consistently.

---

# Part 13: Transcript Facts Versus Additional Engineering Guidance

## 43. Preserve Source Fidelity

| Statement | Classification |
|---|---|
| Three check types are tests, linter and type checker | Explicit transcript |
| Pytest is named as Python test runner | Explicit transcript |
| Ruff is named as linter | Explicit transcript |
| Mypy is named as Python type checker | Explicit transcript |
| TSC checks TypeScript widget code | Explicit transcript |
| Test example checks no order data without OTP | Explicit transcript |
| Linter example is a misspelled/undefined name in rare Ledgerline error branch | Explicit transcript |
| Type checker example is reading status from an order-or-None result | Explicit transcript |
| Hook runs before a Git commit and blocks on failure | Explicit transcript |
| pre-commit tool uses a short repo configuration file | Explicit transcript |
| Fast hooks should take seconds | Explicit transcript |
| Fast tests use fake Ledgerline/carrier and never Claude | Explicit transcript |
| Slow checks run on GitHub pull requests | Explicit transcript |
| Put hook-bypass command in agent's denied permissions | Explicit transcript |
| Green hook does not prove correct behavior | Explicit transcript |
| Agent may weaken tests or add ignore comment | Explicit transcript |
| Review such edits carefully and require PR reasons | Explicit transcript |
| Add hooks on day one of prototype | Explicit transcript |
| Exact .pre-commit-config.yaml content | Illustrative, not provided |
| Exact bypass command argument from video | Not specified |
| CI branch protection configuration | Additional engineering guidance |
| Exact suite runtime or tool versions | Not specified |

---

# Part 14: FDE Interview Questions and Answers

## 44. Main Interview Question

**Q: How do you ensure an AI coding agent runs quality checks before committing code?**

**Strong answer:**

> “I would encode the fast quality gate as a Git pre-commit hook rather than trusting a prompt. For DemoStore, I would run fast fake-backed pytest tests, Ruff linting and mypy type checks; I'd include relevant TypeScript checks as practical. Any nonzero check result blocks the commit and returns useful feedback to the coding agent. I would keep the hook fast—seconds, not minutes—and put live Claude evaluation and other expensive checks in GitHub pull-request CI. I'd deny the agent known hook-bypass commands through its permission settings. Finally, I'd review test changes and checker suppressions against the traceable spec, because a green hook proves only that the checks ran, not that they're meaningful or that the code is correct.”

## 45. Additional FDE Interview Q&A

**Q1. Why isn't a test useful merely because it exists?**

It only checks the current code if somebody actually runs it.

**Q2. Why is this especially important with a coding agent?**

An agent may skip checks, forget instructions in long sessions or report success without execution.

**Q3. Name the three types of checks.**

Behavior tests, linting and static type checking.

**Q4. What does a behavior test do?**

Runs code and compares observed results with expected results.

**Q5. What security example does the transcript use for testing?**

An unverified customer asks about a real order; no protected order data should be returned.

**Q6. Why test with an existing order fixture?**

It isolates whether the verification rule prevents disclosure, not merely whether the order lookup found nothing.

**Q7. What is a linter?**

A tool that analyzes source patterns for likely bugs without normally executing the program.

**Q8. What mistake in the Ledgerline error branch illustrates linting?**

A variable name has one wrong letter and is therefore undefined, even though tests rarely hit that branch.

**Q9. Which Python linter is named?**

Ruff.

**Q10. What is a type checker?**

A static tool that checks how declared/inferred data types are used across the program.

**Q11. Which type-checking tool is named for Python?**

mypy.

**Q12. Which one is named for TypeScript?**

tsc.

**Q13. Do Python type hints enforce themselves at runtime?**

Generally no; tools such as mypy analyze them separately.

**Q14. What does `Order | None` mean?**

A lookup may return an Order or no value at all.

**Q15. What defect does the type checker find in the example?**

Code accesses `order.status` without checking whether `order` is None.

**Q16. Why does the bug show up with another customer's order?**

Authorization hides the foreign order and may return no visible order; reading status then crashes.

**Q17. What is a Git hook?**

A program Git runs automatically at a configured lifecycle moment.

**Q18. When does a pre-commit hook run?**

Immediately before Git records a commit.

**Q19. What happens if one hook check fails?**

The commit is blocked and the error is displayed.

**Q20. Does the hook also run for coding-agent commits?**

Yes, when the agent uses the same configured Git workflow and does not bypass the hook.

**Q21. Which helper tool manages the hook configuration?**

pre-commit.

**Q22. Where does the pre-commit tool read its configuration?**

From a short repository configuration file, commonly `.pre-commit-config.yaml`.

**Q23. Why is a hook faster than running the full evaluation?**

It executes only fast local checks without network/model calls.

**Q24. What is the course's expensive-check example?**

Replaying 40 labeled customer chats through the real model on every commit.

**Q25. Why should real Claude calls be excluded from the hook?**

They can take minutes, cost money and depend on the network.

**Q26. What systems do fast tests use?**

Fake Ledgerline and carrier systems, never the real ones.

**Q27. When are slow checks run?**

Later, on every GitHub pull request.

**Q28. Why is a CLAUDE.md instruction insufficient?**

It can be forgotten or ignored; the file cannot itself block a commit.

**Q29. What is the shared principle with agent guardrails?**

Code enforces the rule; prompts only explain it.

**Q30. What bypass risk remains with a local hook?**

Git can be invoked with an option that skips hooks.

**Q31. How does the lesson constrain the bypass?**

Put that bypass command into the denied-command group in the agent settings.

**Q32. Does a green hook prove the application is correct?**

No. It proves those configured checks ran and returned success.

**Q33. How could an agent cheat accidentally or deliberately to get green?**

Weaken or edit a test assertion, remove a test, or suppress a type/lint warning.

**Q34. What must happen when a test or checker suppression is changed?**

The pull request must explain why, and reviewers must examine it closely.

**Q35. Why compare changed tests with the spec?**

The spec defines the actual requirement; passing a weakened assertion may no longer prove it.

**Q36. When should a prototype add hooks?**

On day one.

**Q37. Why not add all checks months later?**

The backlog may contain hundreds of warnings, encouraging widespread suppression or bypass.

**Q38. How would you decide which checks belong locally?**

Prefer checks that finish in seconds, are deterministic and need no production integrations or real model calls.

**Q39. How do local hooks and CI complement each other?**

Hooks provide rapid feedback before commits; CI runs broader checks in pull requests and can enforce shared repository policy.

**Q40. What is the lesson's practical rule?**

**Do not ask for a check; make the code run it.**

---

# Part 15: Quick Knowledge Check

## 46. Six Questions

Try each question before opening the answer.

<details>
<summary><strong>1. Which check runs code and compares actual versus expected behavior?</strong></summary>

**A.** Linter

**B.** Test

**C.** Type checker

**Correct answer: B.** Tests execute code and check behavior.

</details>

<details>
<summary><strong>2. Which tool is likely to find an undefined variable in an untested error branch?</strong></summary>

**A.** Ruff linter

**B.** The real Claude API

**C.** Git commit message

**Correct answer: A.** Static linting analyzes source patterns even when code paths have not run.

</details>

<details>
<summary><strong>3. Why does `order.status` need a missing-order check?</strong></summary>

**A.** `order` may be `None`.

**B.** Every order has a status.

**C.** Git requires it for all variables.

**Correct answer: A.** A lookup can return no order, especially for unauthorized or missing records.

</details>

<details>
<summary><strong>4. What should happen when a pre-commit check fails?</strong></summary>

**A.** The commit proceeds with a warning.

**B.** The commit is blocked; the developer or agent receives the error.

**C.** All tests are deleted.

**Correct answer: B.**

</details>

<details>
<summary><strong>5. Where should the 40-chat live model evaluation run?</strong></summary>

**A.** Before every quick local commit.

**B.** In slower pull-request checks, not the seconds-long hook.

**C.** Inside the browser widget.

**Correct answer: B.**

</details>

<details>
<summary><strong>6. A hook passes after the agent changes an authorization test. What next?</strong></summary>

**A.** Trust the green result without review.

**B.** Review the changed assertion and PR reason against the business requirement.

**C.** Skip CI because a hook already ran.

**Correct answer: B.** The checks can be made weaker; passing alone is not proof of correctness.

</details>

---

# Part 16: Complete Revision Checklist

## 47. The Central Problem

- [ ] A test is useful only when it runs.
- [ ] Coding agents may skip a requested test or misreport results.
- [ ] **Automatic executable gates** are stronger than prompt requests.
- [ ] The three parts are **checks, hooks and code-level enforcement**.

## 48. Tests

- [ ] A test runs code and compares actual with expected results.
- [ ] DemoStore example: unverified chat requests a real order.
- [ ] Expected result: **no protected order data**.
- [ ] Tests check **spec-defined behavior**.
- [ ] Fast tests use **fake Ledgerline and carrier**.
- [ ] Fast tests **never call real Claude**.

## 49. Linting

- [ ] A linter checks suspicious source patterns without executing code.
- [ ] Example: misspelled undefined variable in rarely used Ledgerline error branch.
- [ ] Python linter is **Ruff**.
- [ ] Linting complements behavioral tests by inspecting otherwise-unexecuted code.

## 50. Type Checking

- [ ] Type checking compares declared value types with usage.
- [ ] Python supports type hints.
- [ ] Python type hints are **not runtime guarantees**.
- [ ] Python checker: **mypy**.
- [ ] TypeScript checker: **tsc**.
- [ ] Order lookup may return **Order or None**.
- [ ] Reading `status` without checking for `None` is unsafe.
- [ ] A foreign order may be hidden as not-found, triggering that path.

## 51. Pre-Commit Hook

- [ ] A Git hook runs automatically at a configured Git event.
- [ ] **Pre-commit** runs before a commit is saved.
- [ ] A failed check **blocks the commit**.
- [ ] The tool **pre-commit** manages configuration in a repository file.
- [ ] Hooks apply to **humans and agents** using the configured workflow.
- [ ] Agent sees checker errors and can repair code before retrying.
- [ ] Hook should finish in **seconds**.

## 52. Fast Versus Slow Checks

- [ ] Keep local hook checks deterministic and cheap.
- [ ] Never run the full **40-chat live model evaluation** before each commit.
- [ ] Avoid real Ledgerline, carrier, Claude and production data in fast tests.
- [ ] Run slow checks in **GitHub pull requests**.
- [ ] Review the code as well as the check results.

## 53. Prompts, Permissions and Review

- [ ] `CLAUDE.md` can request checks but cannot enforce them.
- [ ] A Git hook actually invokes checks and blocks a failing commit.
- [ ] Git has hook-bypass options.
- [ ] Deny the bypass command in the agent's permission settings.
- [ ] A green hook only proves its checks **ran and passed**.
- [ ] An agent could weaken a test to make it pass.
- [ ] An agent could add a type/lint suppression.
- [ ] Such changes need an explicit **PR rationale and careful review**.
- [ ] Compare tests to the **traceable business requirements**.
- [ ] Add hooks **on day one of the prototype**.
- [ ] Late adoption may uncover hundreds of existing warnings.

---

# Part 17: Final Summary and Practice Task

## 54. One-Page Summary

| Area | Lesson takeaway |
|---|---|
| Problem | Tests exist but may not run |
| Check 1 | Pytest checks behavior |
| Check 2 | Ruff catches likely bugs |
| Check 3 | Mypy/tsc check type usage |
| DemoStore security test | No order data without verified email code |
| Lint example | Undefined name in rare Ledgerline error handling |
| Type example | Order or None; unsafe status access |
| Git hook | Runs checks before commit, blocks failures |
| Hook manager | pre-commit with repo configuration |
| Hook speed | Seconds, not minutes |
| Fast dependencies | Fake ERP, fake carrier, no real Claude |
| Slow checks | GitHub pull requests |
| Prompt weakness | CLAUDE.md alone cannot block commits |
| Bypass protection | Deny hook-skip commands to the agent |
| Green hook limit | Passing tests can still be wrong or weakened |
| Review requirement | Explain and inspect test/suppression edits |
| Timing | Add hook on first prototype day |
| Practical rule | Don't ask for a check; make the code run it |

## 55. Practice Task — Set Up a Fast Local Gate

Use a safe sample project or your own DemoStore prototype:

1. Write a **fast test** that asserts an unverified customer cannot see a known order.
2. Add a **lint check** and **type check**.
3. Install a **pre-commit hook** that runs them automatically.
4. Intentionally add an undefined variable. Try committing and observe the block.
5. Fix the name, then introduce a missing `None` check in a typed order lookup. Observe the type-check failure.
6. Ensure **live Claude tests and the 40-case evaluation are not in the local hook**.
7. Configure a GitHub pull-request job for slower checks in the appropriate trusted environment.
8. Review the agent's tool permissions and prevent known hook bypasses.
9. Compare any changed test assertions or suppressions against the business spec before approval.

**What you should be able to say in an FDE interview:** “I don't rely on the agent remembering quality steps. Fast automated checks run before a commit; deeper validation runs in pull requests; human review confirms the checks have not been weakened.”

> **Practical rule: Do not ask for a check. Make the code run it.**

**Next lesson:** Claude Code plans the assistant prototype in phases with these automated checks already in place.
