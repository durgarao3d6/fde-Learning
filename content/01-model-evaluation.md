# Model Selection and Evaluation

## Learning objective
Choose a model for a real customer-support workflow using measured evidence, not benchmark reputation.

## DemoStore scenario
DemoStore needs an AI assistant to identify customer intent, select the correct action and request a human handoff when necessary. Marco Rossi, head of support, supplies **40 real, labeled customer conversations**. The project sponsor is Lena Brandt.

Each example identifies the intent, the correct action, and whether a human must take over.

## Candidate models
The comparison considers Anthropic **Haiku, Sonnet and Opus**. Quality is not the only criterion: compare **accuracy, missed handoffs, unnecessary handoffs, latency, cost and consistency**.

A missed required handoff is more dangerous than an unnecessary escalation. An assistant that confidently answers when it should transfer a sensitive case creates an operational risk.

## Evaluation workflow
The lesson builds `llm.py`, `intent.py`, and `compare.py`. Forty conversations are evaluated with three models and three repetitions:

```text
40 conversations × 3 models × 3 runs = 360 requests
```

Use `ThreadPoolExecutor(max_workers=20)` to run requests concurrently. Repetition helps reveal inconsistency and nondeterminism.

### Structured-output subtlety
The initial field name `reasoning` caused trouble with Opus because it could be interpreted as referring to the model's private reasoning. Renaming it `summary` avoids the ambiguity and makes the desired public explanation explicit.

## Illustrative economics
Course-era example input costs per million tokens: Opus **$4**, Sonnet **$2**, and Haiku **$0.10**; output tokens roughly five times the input rate. The lesson estimates about **$3 per 1,000 chats** for Opus versus **under $0.10 per 1,000** for Haiku. These are illustrative historic figures, not live prices.

## Observed risk
All three model candidates missed roughly half of the **21 handoff-required cases** across runs, suggesting the escalation prompt was too narrow. Model selection alone cannot repair an inadequate definition of escalation.

## Decision
Haiku is a tentative choice for low-cost intent detection, **conditional on improving the handoff prompt and rerunning evaluation**. The choice is evidence-dependent, not a universal recommendation.

## Review questions
1. Why are missed handoffs weighted more heavily than extra handoffs?
2. Why run each labeled chat more than once?
3. What does a three-model comparison prove, and what does it not prove?
4. How would you improve the escalation prompt before a pilot?

## Remember
**Evaluate on representative, labeled customer work. Optimize for the errors that matter to the business.**

---

# Detailed lesson notes

## 1. Business framing: what are we optimizing?
DemoStore is **not** selecting a general-purpose chatbot on the basis of a leaderboard. The narrowly defined first task is **support intent classification**. The system should understand what the customer wants, determine the appropriate next action, and flag messages that need a person. Marco Rossi's team already has labeled real conversations, and sponsor Lena Brandt needs an evidence-based decision.

A model can produce a natural-sounding reply and still fail the business task. A helpful-looking answer to a fraud or complex policy case is an error if the correct action was human escalation. This explains why escalation accuracy is more important than how polished the answer sounds.

### Dataset and labels
For each of the **40 real customer-support conversations**, the evaluation needs at least:
- Customer conversation text, preserved realistically, including messy and ambiguous writing.
- Ground-truth intent.
- Ground-truth required next action.
- Ground-truth flag for human handoff.

The label is the reference for grading; the model output is only a prediction. Do not grade on whether the model's reasoning sounds plausible.

### Three model candidates
The lesson compares **Haiku, Sonnet, Opus**. Test all three on the **same** 40 conversations and grading rules. Otherwise, differences might reflect the prompt or dataset rather than the models.

| Metric | What it measures | Why the FDE cares |
|---|---|---|
| Intent accuracy | Intent matches human label | Routes chats correctly |
| Correct action | Next step matches expected action | Prevents workflow mistakes |
| Missed handoffs | Human was needed but AI did not escalate | Potentially high-risk false negatives |
| Unnecessary handoffs | AI escalated a case it could handle | Increased support workload |
| Latency | Time before a useful result | Customer experience |
| Cost | Estimated token spend per interaction | Scalability |
| Consistency | Repeated executions make similar decisions | Operational reliability |

**Safety-weighted judgment:** Treat missed required handoffs separately rather than hiding them inside an average accuracy score. A model may be 90% accurate overall but still miss the cases that matter most.

## 2. Why repeat the evaluation?
The evaluation performs **3 runs per model per case**. The arithmetic is:

```text
40 cases × 3 models × 3 repetitions = 360 model requests
```

A model can return slightly different classifications on separate runs. Repeated trials show which cases are unstable and whether a model is predictably correct. A single good sample is not enough.

### Implementation outline (illustrative)
```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def grade_one(example, model_name, run_number):
    prediction = classify(example["conversation"], model=model_name)
    return {
        "model": model_name,
        "run": run_number,
        "case_id": example["id"],
        "predicted_intent": prediction["intent"],
        "missed_handoff": example["needs_human"] and not prediction["needs_human"],
    }

jobs = [(row, model, run)
        for row in labeled_chats
        for model in ("haiku", "sonnet", "opus")
        for run in range(3)]

with ThreadPoolExecutor(max_workers=20) as pool:
    results = list(pool.map(lambda job: grade_one(*job), jobs))
```

This pseudocode illustrates the lesson's concurrency pattern; the actual model IDs, function signatures and dataset keys are not provided. A concurrency limit avoids launching all 360 requests at once.

## 3. The surprising schema problem: `reasoning` versus `summary`
An early output schema asked for a field named `reasoning`. Opus sometimes treated that as though it were a request for internal/private chain-of-thought, rather than a short customer-support explanation. The field was renamed **`summary`**.

The engineering lesson is bigger than the field name: **output schemas and descriptions are part of the prompt**. Vocabulary may change model behavior. For auditable intent systems, request a concise user-facing *summary or decision explanation*, not hidden internal reasoning.

Illustrative structured result:

```json
{
  "intent": "delivery_status",
  "order_number": "EXAMPLE-1024",
  "needs_human": false,
  "summary": "Customer asks whether a shipped order is delayed."
}
```

The exact six allowed intent values are defined in a later lesson and are **not enumerated in the supplied transcript**.

## 4. Cost comparison
The course uses historic **illustrative** input pricing per million tokens: **Opus $4**, **Sonnet $2**, **Haiku $0.10**. Approximate output pricing is **5×** input pricing. Its simplified chat comparison estimates **Opus ~$3 per 1,000 chats** versus **Haiku <$0.10 per 1,000 chats**.

Do not use these as current vendor quotations. Real cost depends on input/output token volumes, caching, prompt size, tool rounds, price changes and billing. A useful financial model separates:
1. Requests per week × tokens per request.
2. Input and output unit rates.
3. Retry and evaluation traffic.
4. Cost of escalations and mistakes.
5. Human review and maintenance effort.

The cheapest model is not automatically best: a missed escalation may cost much more than its API savings.

## 5. What the comparison actually revealed
There were **21 escalation-required cases** in the 40-case set. Across repetitions, **all three models missed approximately half of the necessary handoffs**. This means a stronger or more expensive model was **not** an adequate substitute for better instructions and testing.

A likely issue was the **narrow escalation prompt**. The course treats **Haiku** as a promising low-cost first choice for intent detection, but only after expanding the handoff definitions and reevaluating. It does **not** conclude that Haiku is safe to deploy immediately.

### A safer next evaluation cycle
- Review every missed-handoff case with Marco's agents.
- Write explicit trigger descriptions for scenarios requiring a human.
- Include borderline and multi-issue conversations.
- Improve examples without exposing personal information.
- Repeat the same 40-case evaluation across models.
- Report both aggregate success and worst-case safety metrics.
- Obtain stakeholder acceptance before moving into a pilot.

## 6. Failure patterns to recognize
**Failure: choose the flagship model automatically.** A more powerful model may bring cost without correcting task instructions.

**Failure: report only accuracy.** One number conceals false negatives on critical cases.

**Failure: test only friendly synthetic messages.** Production customers type long, emotional, multilingual and incomplete messages.

**Failure: use one run as proof of consistency.** LLM outputs can vary.

**Failure: copy pricing into an investment decision without recalculation.** Prices and workload assumptions change.

## 7. Interview-ready explanation
**Question:** “How would you choose a model for customer-support routing?”

**Answer:** “I would first agree on the business decision, obtain representative labeled conversations, run each candidate on identical cases multiple times, score action accuracy and missed handoffs separately, then compare cost and latency among candidates that meet the safety bar. For DemoStore the initial test showed all three models missed many required escalations, so I would improve and rerun the evaluation before selecting a deployment candidate.”

### Revision checklist
- [ ] Remember **40 labeled conversations**, **3 models**, **3 repetitions**, **360 requests**.
- [ ] Remember concurrency setting **20 workers**.
- [ ] Distinguish missed and unnecessary handoffs.
- [ ] Remember the **21 handoff-required cases**.
- [ ] Explain why `summary` replaced `reasoning`.
- [ ] Keep pricing explicitly **illustrative/historical**.
- [ ] State why Haiku was a **tentative**, not final, choice.

---

# Additional Line-by-Line Teaching Notes and Worked Examples

## From business objective to measurable model choice
The exercise does not ask, “Which Claude model is smartest?” It asks which candidate is an appropriate and economical **classifier for DemoStore support routing**. That matters because model-selection work without a well-defined operational action can optimize the wrong metric. The support team, not a public leaderboard, defines what a correct response means.

**Marco Rossi** supplies the labeled material because he and the support agents know what actual support outcomes should be. **Lena Brandt** sponsors the work and needs a defensible decision. The distinction between **subject-matter owner**, **business sponsor** and **engineer** is important for FDE interviews.

### A reproducible grading record (illustrative)
```json
{
  "case_id": 17,
  "ground_truth": {
    "intent": "order_status",
    "next_action": "human_handoff",
    "needs_human": true
  },
  "prediction": {
    "intent": "order_status",
    "next_action": "lookup",
    "needs_human": false
  },
  "grade": {
    "intent_correct": true,
    "action_correct": false,
    "missed_handoff": true
  }
}
```

Even though the **intent** is correct, the **action is wrong**. This is the key reason not to score an assistant only on intent classification.

The JSON keys and values are illustrative, not a claim that the course's underlying files use those exact names.

## False-negative versus false-positive escalation
For the human-handoff flag:

| Ground truth | Prediction | Interpretation |
|---|---|---|
| Human required | Human required | Correct escalation |
| Human required | No human | **Missed handoff** — safety/quality concern |
| No human required | Human required | Unnecessary handoff — workload concern |
| No human required | No human | Correct automation |

The lesson's striking result—**around half of 21 required handoffs missed by each candidate across runs**—cannot be hidden behind good performance on simpler tracking chats.

For improvement, inspect cases where models consistently decline to escalate; ask Marco whether the trigger rules are under-specified. Do not immediately buy the most expensive model.

## Reproducibility notes for evaluation code
**`llm.py`** isolates the Anthropic request. **`intent.py`** defines the extraction task and output shape. **`compare.py`** orchestrates models, cases and repetitions, then computes metrics. This modular organization makes it easier to swap prompts or models without rewriting data loading and scoring.

The maximum thread workers value is **20**, not 360. The total work is 360 requests; the concurrency setting bounds how many run at once.

Useful execution records (additional best practice) include model identifier, prompt version, case ID, run ID, elapsed time, token counts, output schema validity and the resulting judgment. Without these, a headline percentage may be impossible to reproduce.

## Cost is an outcome, not just a token rate
A cheap model that escalates most chats may have higher **all-in cost** than its API bill suggests. A costly model that handles more cases safely may still make economic sense, but it must **actually demonstrate** improved behavior.

Think in three layers:
1. **Inference price:** tokens and API requests.
2. **Support operations:** human escalations, repeat contacts, rework.
3. **Risk exposure:** missed mandatory handoff, wrong promises and privacy errors.

In the provided lesson, the low-price Haiku choice is **provisional** precisely because behavior needs improvement.

## Distinguish a development target from a release gate
A later prototype lesson uses the illustrative criterion **36/40 correct actions with no missed mandatory handoffs**. That is a separate stage-specific example, not a claim that the first model-selection run passed it. Keeping those lessons separate prevents accidental overstatement.

## Answering a challenge question
**“Why run 360 calls if you have only 40 cases?”**

Because the experiment compares **three models** and also checks **within-model stability through three repeated executions**. Forty independent cases alone cannot expose inconsistent outcomes when the model is run multiple times on the same chat.

**“Why did the word `reasoning` matter?”**

Because a schema field name is part of the model instruction. The lesson found that `summary` better signals a short shareable explanation, whereas `reasoning` sometimes conflicted with how Opus treated private reasoning.

## Final memory anchors
**40 chats; 21 handoff cases; Haiku/Sonnet/Opus; three repeats; 360 calls; 20 workers; rename `reasoning` to `summary`; approximately half of required handoffs missed; improve prompting before accepting Haiku.**
