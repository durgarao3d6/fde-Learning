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
