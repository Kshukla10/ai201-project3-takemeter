# TakeMeter — r/soccer Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in r/soccer, distinguishing between analytical posts, hot takes, and reactions.

---

## Community Choice

**r/soccer** is one of the largest sports communities on Reddit, with millions of members discussing matches, transfers, tactics, and player performance across all global leagues. It's an ideal fit for a classification task because discourse quality varies enormously in a recognizable way, the community itself uses terms like "hot take" and "actually a good point" to describe post quality. The variation between a detailed tactical breakdown and a pure emotional reaction is high, consistent, and meaningful to regular participants.

---

## Label Taxonomy

### `analysis`
A structured argument supported by specific evidence: statistics, tactical observations, historical comparisons, or detailed reasoning. The claim could be evaluated as true or false based on the evidence provided.

**Example 1:**
> "For this World Cup they made head to head the first tiebreaker over goal differential. If they beat the US, they'll have 3 points, as will the loser of Australia-Paraguay (if no draw). But both Paraguay and Australia hold the head to head tiebreaker over Türkiye. So they have no route to finishing higher than dead last even if they beat the US 15–0."

**Example 2:**
> "He does have a wide passing range but he doesn't spam long passes or through balls especially when he plays as a 6. He focuses more on retention, short passes and passing it into space for the other player to progress the ball."

---

### `hot_take`
A bold, confident opinion stated without meaningful supporting evidence. The post asserts rather than argues. May feel provocative or designed to spark debate.

**Example 1:**
> "God i cant stand steve clarke, this team selection is pants."

**Example 2:**
> "Even if Curacao had one man less, Ecuador would not score."

---

### `reaction`
An immediate emotional response to a specific match event, result, or piece of news. Little to no argument, the post is expressing a feeling in the moment.

**Example 1:**
> "Shocking game but probably the most fun I've had watching outside of the Sweden game. The ref when announcing the red card had me in tears."

**Example 2:**
> "Incredible. Fans and manager crying. This is what tournament ball is about."

---

## Data Collection

**Source:** r/soccer comment sections, collected manually from public match threads during the 2024 FIFA World Cup group stage.

**Thread types used:**
- Live match threads and post-match threads (primary source of `reaction`), e.g., Ecuador vs Curaçao, Turkey vs Australia
- In-thread debate comments on refereeing, tactics, and player performance (primary source of `hot_take` and `analysis`)
- Tournament discussion threads on VAR, cooling breaks, and group stage format (mixed source across all three labels)

**Labeling process:** Each comment was read individually and assigned one label using the decision tree: (1) Is this reacting to a specific event? → `reaction`. (2) Does it cite specific supporting evidence? → `analysis`. (3) Otherwise → `hot_take`.

**Label distribution:**

| Label | Count |
|-------|-------|
| hot_take | 69 |
| analysis | 67 |
| reaction | 66 |
| **Total** | **202** |

---

### Difficult-to-label examples

**Case 1 — The reasoned reaction:**
> "The worst thing VAR has done is make everyone go crazy when fouls aren't given for borderline minimal contact decisions. The first couple of years of VAR they were way too likely to give penalties for microscopic contact on players. Now they're trying to move away from that to make the threshold for contact much higher, which is the right thing to do. They just should have done it this way to start with."

Could be `reaction` (expressing frustration about VAR) or `analysis` (makes a structured argument about how VAR policy has evolved). **Decision:** `reaction` — the post is venting about a broader frustration rather than making a data-backed argument. The reasoning is present but the dominant mode is emotional opinion, not structured evidence.

**Case 2 — The tactical observation that reads like a reaction:**
> "A lot of these matches involve 1 team scoring a few goals in the first half and then parking the bus for 45+ minutes and I'm getting really really tired of it."

Could be `analysis` (tactical observation about match patterns) or `reaction` (expressing fatigue/frustration). **Decision:** `analysis` — the post identifies a specific tactical pattern across multiple matches, which is an observation grounded in what's happening on the pitch rather than pure emotion about a single event.

**Case 3 — The opinion backed by a mild stat reference:**
> "De Jong actually plays a lot of progressive passes though. If someone actually thinks he's not a good player, you should probably disregard their footballing input."

Could be `hot_take` (dismissive, provocative) or `reaction + hot_take` (responding to someone else's opinion). **Decision:** `hot_take` — the post makes a bold dismissive claim ("disregard their footballing input") with only a vague evidence gesture ("a lot of progressive passes"). The evidence is not specific enough to qualify as analysis, and the tone is assertive rather than reactive to a match event.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training setup:**
- Framework: HuggingFace `transformers` + `Trainer` API
- Split: 70% train / 15% validation / 15% test (144 / 31 / 31 examples)
- Hardware: Google Colab T4 GPU

**Hyperparameter decisions:**

| Parameter | Value | Reasoning |
|-----------|-------|-----------|
| `num_train_epochs` | 8 | Default of 3 epochs underfit badly (45% accuracy). With only 144 training examples, more passes are needed to converge. |
| `learning_rate` | 3e-5 | Slightly higher than default (2e-5) to help the model move faster toward a solution on a small dataset. |
| `per_device_train_batch_size` | 16 | Default — appropriate for dataset size. |

The initial run at 3 epochs produced 45.2% accuracy (barely above random). Increasing to 8 epochs improved this to 74.2%, confirming the model needed more training passes on this small dataset.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)

**Prompt used:**
```
You are classifying comments from r/soccer, a large online football discussion community.
Assign each post to exactly one of the following categories.

analysis: A structured argument supported by specific evidence such as statistics,
tactical observations, or historical comparisons. The evidence genuinely supports
the claim being made.

hot_take: A bold, confident opinion stated without meaningful supporting evidence.
The post asserts rather than argues. May feel provocative or designed to spark debate.

reaction: An immediate emotional response to a specific match event, result, or
piece of news. Little to no argument — the post is expressing a feeling in the moment.

Respond with ONLY the label name — one word, nothing else.
Valid labels: analysis, hot_take, reaction
```

All 31 test examples were parseable (100% parse rate).

---

## Evaluation Report

### Overall accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq) | 58.1% |
| Fine-tuned DistilBERT | **74.2%** |
| Improvement | +16.1 pp |

Fine-tuning meaningfully outperformed the zero-shot baseline by 16 percentage points.

---

### Per-class metrics

**Fine-tuned model:**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 0.82 | 0.90 | 11 |
| hot_take | 0.67 | 0.80 | 0.73 | 10 |
| reaction | 0.60 | 0.60 | 0.60 | 10 |
| **macro avg** | **0.76** | **0.74** | **0.74** | 31 |

**Zero-shot baseline:**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 0.82 | 0.90 | 11 |
| hot_take | 0.33 | 0.10 | 0.15 | 10 |
| reaction | 0.42 | 0.80 | 0.55 | 10 |
| **macro avg** | **0.58** | **0.57** | **0.54** | 31 |

The biggest gain from fine-tuning came in `hot_take`: F1 improved from 0.15 to 0.73. The baseline almost completely failed on this label, predicting it rarely. The fine-tuned model learned it well.

---

### Confusion matrix (fine-tuned model)

| True \ Predicted | analysis | hot_take | reaction |
|-----------------|----------|----------|----------|
| **analysis** | 10 | 0 | 1 |
| **hot_take** | 0 | 8 | 2 |
| **reaction** | 1 | 5 | 4 |

The dominant error pattern is **`reaction` being predicted as `hot_take`** (5 out of 10 reaction posts misclassified this way). The `analysis` boundary is cleanly learned — zero confusion with `hot_take`.

---

### Wrong predictions — analysis

**Error 1:**
> "There's Room but not for Ecuador."
- True: `reaction` | Predicted: `hot_take` | Confidence: 0.38

This comment only makes sense as a reaction if you know it was posted during a live match. In isolation, it reads like a bold opinion about Ecuador's tournament chances. The model has no access to thread context — it sees only the text. This is a data problem: some reaction posts are inseparable from their context.

**Error 2:**
> "15 saves on the night so far is crazy"
- True: `reaction` | Predicted: `hot_take` | Confidence: 0.36

This is a factual observation about a live match stat, expressed with surprise. The model read "crazy" as opinion language and flagged it as `hot_take`. The real issue is that short, stat-referencing posts look like analysis or hot_take without knowing they were written mid-match.

**Error 3:**
> "Rob Stone is such an enthusiastic goober and I'm here for it. He's so happy to be repping our country as a broadcaster."
- True: `hot_take` | Predicted: `reaction` | Confidence: 0.36

This is a defensible annotation disagreement. The post expresses warm personal feeling about a broadcaster, which reads more like a reaction than a bold opinion. This might genuinely belong to `reaction` — it's event-adjacent but not making a claim. This is a label boundary problem, not a model problem.

**Systematic pattern:** All 9 wrong predictions had confidence scores between 0.34–0.38, barely above random. The model is appropriately uncertain when it gets things wrong, which is a useful property for a deployed tool.

---

### Error Pattern Analysis

**Pattern: The model classifies by tone, not by event-specificity — `reaction` posts without emotional language are systematically mislabeled as `hot_take`.**

Across all 9 wrong predictions, 7 involve a `reaction` post being mislabeled. The confusion matrix confirms this directionally: 5 of 10 `reaction` posts were predicted as `hot_take`, the single largest error cluster in the matrix.

The cause is identifiable. The model learned `reaction` as a *tone* — warm, emotional, expressive language — and `hot_take` as a *stance*, short, assertive, no hedging. But the `reaction` label is actually situational: a post made in immediate response to a specific match event, regardless of surface tone. When those two things diverge, the model reliably fails.

**Short factual match observations (true: `reaction`, predicted: `hot_take`):**

| Text | Confidence |
|------|------------|
| "You can tell that Curaçao is tired." | 0.37 |
| "Bless the ref, he's just letting them play." | 0.38 |
| "15 saves on the night so far is crazy" | 0.36 |

None of these use stereotypically emotional language. Each is a brief, direct observation made during a live match. Without emotional markers, the model sees only a short assertive claim and routes it to `hot_take`.

**The reverse failure — soft tone mislabeled as `reaction`:**

> "I don't think anyone has ever been against cooling breaks if necessary."
> True: `hot_take` | Predicted: `reaction` | Confidence: 0.35

The hedged phrasing ("I don't think anyone has ever been") reads as low-assertiveness, which the model associates with `reaction`. But the post is making a bold claim about universal consensus with no evidence — a textbook `hot_take`. The tone signal misleads in both directions.

**Why this pattern exists:**

This is a data distribution issue. `Reaction` posts in live match threads are short and observational, but the training examples likely skewed toward the more expressive end (fans cheering, expressing disbelief). The quieter observational reactions were underrepresented, leaving the model without signal for that subtype.

**What would fix it:**

Collecting 20–30 additional `reaction` examples from the short, observational end of the spectrum, factual match comments without exclamation points or emotional vocabulary — would give the model the signal it currently lacks. A label definition that explicitly names "brief factual match observations" as a `reaction` subtype would also reduce annotation inconsistency in future data collection.

---

## Confidence Calibration

Calibration measures whether the model's confidence scores correspond to actual
accuracy, a well-calibrated model that says "70% confident" should be right ~70%
of the time.

| Confidence bin | Examples | Accuracy |
|----------------|----------|----------|
| 0.4–0.5        | 20       | 60.0%    |
| 0.5–0.6        | 2        | 100.0%   |
| 0.6–0.7        | 1        | 100.0%   |
| 0.7–0.8        | 6        | 100.0%   |
| 0.8–0.9        | 2        | 100.0%   |

**Finding:** The model is perfectly calibrated at moderate-to-high confidence,
every prediction above 0.5 confidence is correct (11/11, 100%). Errors are
entirely concentrated in the low-confidence band (0.4–0.5), where accuracy drops
to 60% (12/20 correct). No predictions fell below 0.4 or above 0.9, reflecting
the small test set size (31 examples).

This means confidence is a reliable signal for a deployed tool: predictions above
0.5 can be trusted; predictions in the 0.4–0.5 range should be treated as
uncertain. All 9 wrong predictions in the test set fell in this low-confidence
band, consistent with the model "knowing when it doesn't know."

---

### Sample classifications

| Text | True label | Predicted | Confidence | Notes |
|------|------------|-----------|------------|-------|
| "2 games, 61 shots, 0 goals." | analysis | analysis | — | Correct — concise stat-based observation, model correctly identifies factual evidence |
| "God i cant stand steve clarke, this team selection is pants." | hot_take | hot_take | — | Correct — bold opinion, no supporting evidence, clear assertive tone |
| "Incredible. Fans and manager crying. This is what tournament ball is about." | reaction | reaction | — | Correct — pure emotional response to a match moment, no argument made |
| "There's Room but not for Ecuador." | reaction | hot_take | 0.38 | Wrong — context-dependent match comment misread as opinion without thread context |
| "15 saves on the night so far is crazy" | reaction | hot_take | 0.36 | Wrong — mid-match stat observation misread as hot take |

---

## Reflection: What the model learned vs. what I intended

I intended the model to learn three discourse *modes* — the way a post reasons (or doesn't). What it actually learned is closer to three *surface patterns*: long posts with numbers → `analysis`; short assertive posts → `hot_take`; emotional language → `reaction`.

This works well most of the time because those surface patterns correlate strongly with the real distinctions. But it breaks down for short, context-dependent match comments — posts that are genuinely `reaction` but lack the emotional language the model associates with that label.

The model also didn't learn that `reaction` requires event-specificity — it learned `reaction` as a tone (warm, emotional) rather than a situational category (responding to something that just happened). That's the core gap between my intended label and what the model captured.

To close this gap I would need more examples of short, factual-sounding reactions from live match threads, explicitly labeled to teach the model that event-context matters — not just emotional tone.

---

## Spec Reflection

**One way the spec helped:** The spec's emphasis on label design before data collection was the most valuable guidance in the project. Writing decision rules for edge cases (especially the "one-stat hot take") before annotating prevented a lot of inconsistency, I had a rule to apply rather than making a fresh judgment each time.

**One way implementation diverged:** The spec suggests the fine-tuned model should outperform the baseline, but my initial run at default hyperparameters produced the opposite (45% vs 58%). The spec doesn't discuss what to do when fine-tuning underperforms, I had to diagnose the underfitting problem myself (too few epochs for a small dataset) and adjust. In practice, hyperparameter tuning is a necessary step the spec treats as optional.

---

## AI Usage

**1. Label stress-testing:** I used Claude to generate 10 borderline posts sitting between `analysis` and `hot_take` before annotating. Several of the generated examples (one-stat posts with sweeping conclusions) were genuinely hard to classify, which confirmed my decision rule needed to be explicit about "evidence that supports the claim" vs. "evidence that decorates the claim." I tightened the definition before starting annotation.

**2. Failure pattern analysis:** After fine-tuning, I pasted all 9 wrong predictions into Claude and asked it to identify common themes. It correctly identified the context-dependency pattern (reactions that only make sense in a live thread) and the low-confidence pattern across all errors. I verified both patterns by re-reading the examples myself — both held up. Claude also suggested sarcasm as a potential pattern, which I checked and found didn't apply to this set of errors.

---

## Repository Contents

```
├── README.md
├── planning.md
├── data.csv
├── confusion_matrix.png
└── evaluation_results.json
```
