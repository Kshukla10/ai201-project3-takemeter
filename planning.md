# TakeMeter — Planning Document
## Community: r/soccer

r/soccer is one of the largest sports communities on Reddit, with millions of members
discussing matches, transfers, tactics, and player performance across all global leagues.
The discourse ranges from emotional matchday reactions to detailed tactical breakdowns,
making it an ideal fit for a classification task — the variation in post quality is high,
the text is abundant, and the distinctions between post types are ones the community
itself recognizes and talks about explicitly.

---

## Label Taxonomy

### analysis
A post that makes a structured argument supported by specific evidence: statistics,
tactical observations, historical comparisons, or detailed reasoning. The claim could
be evaluated as true or false based on the evidence provided.

**Example 1:**
"Rodri's progressive pass rate this season (8.3 per 90) puts him in the top 2% of
defensive midfielders in Europe's top 5 leagues — his absence explains exactly why
City's build-up has broken down."

**Example 2:**
"People forget that Ancelotti has won the CL with three different clubs using completely
different systems. His tactical flexibility is genuinely underrated — the 2022 Madrid
side ran more counter-press than any of his previous CL winners."

---

### hot_take
A bold, confident opinion stated without meaningful supporting evidence. The post
asserts rather than argues. May feel provocative or designed to spark debate.

**Example 1:**
"Haaland would be exposed in La Liga. He only scores because City create 40 chances
a game. Put him at Atletico and he disappears."

**Example 2:**
"The Premier League is genuinely the worst top league for actual football quality.
It's just pace and physicality, no real tactical sophistication."

---

### reaction
An immediate emotional response to a specific match event, result, or piece of news.
Little to no argument — the post is expressing a feeling in the moment.

**Example 1:**
"I cannot believe we just conceded in the 94th minute. This club will actually kill me."

**Example 2:**
"THAT BELLINGHAM GOAL. INSANE. ONE OF THE BEST IN UCL HISTORY."

---

## Hard Edge Cases

### The one-stat hot take
**Example post:** "Bellingham has more goal contributions than any English player in
La Liga history — he's clearly the best English player ever."

This cites a real stat but uses it to support a sweeping, poorly-reasoned claim.

**Decision rule:** If the evidence genuinely supports the claim (removing it would
weaken the argument in a meaningful way), label it `analysis`. If the evidence is
decorative — one cherry-picked number propping up a much larger claim — label it
`hot_take`. The Bellingham example above is `hot_take`: the stat is real but doesn't
remotely support "best English player ever."

### The calm reaction
**Example post:** "Gutted by tonight. We've been shipping goals from set pieces all
season and nobody's fixed it."

This is emotional but also makes a mild tactical observation. Decision rule: if the
primary purpose is expressing a feeling about a specific event (win, loss, goal,
transfer news), label it `reaction` even if it contains a passing observation.
Reserve `analysis` for posts where the argument is the *point* of the post.

---

## Data Collection Plan

**Source:** r/soccer comment sections and post bodies, collected manually.

**Target distribution:** ~70 examples per label (210 total), aiming for roughly equal
representation across all three.

**Collection strategy:** Pull from a mix of:
- Match threads (rich source of `reaction` posts)
- Tactical/analysis discussion posts (rich source of `analysis`)
- Hot-take-bait posts and transfer rumors (rich source of `hot_take`)

**If a label is underrepresented:** If any label falls below 55 examples after
initial collection, I will go back and specifically target post types that generate
that label (e.g., seek out "unpopular opinion" threads for more `hot_take` examples).

---

## Evaluation Metrics

**Primary metric:** Per-class F1 score for all three labels.
Accuracy alone is misleading here because a model that learns to predict `hot_take`
constantly could score 33% without learning anything. F1 per class reveals whether
the model has actually learned each boundary, not just the majority class.

**Secondary metrics:**
- Confusion matrix: to identify which label pairs are being confused and in which direction
- Overall accuracy: for comparison with the zero-shot baseline

---

## Definition of Success

A classifier I'd consider "good enough" for real use:
- Per-class F1 ≥ 0.70 for all three labels on the test set
- Fine-tuned model accuracy meaningfully exceeds the zero-shot baseline (by at least 10 percentage points)
- No single label pair dominates the confusion matrix (no single off-diagonal cell > 20% of that class's examples)

If `analysis` vs `hot_take` confusion is high, the classifier isn't capturing the
distinction that matters most — that would be a failure even at acceptable overall accuracy.

---

## AI Tool Plan

### Label stress-testing
I'll give Claude my label definitions and edge case descriptions and ask it to generate
10 posts that sit at the boundary between `analysis` and `hot_take` — the hardest
boundary in this taxonomy. If I can't cleanly classify the generated posts, I'll
tighten the definitions before annotating 200 examples.

### Annotation assistance
I may use an LLM to pre-label a batch of 50–80 examples using my label definitions,
then review and correct every assignment before accepting it. I will track which
examples were pre-labeled in a `notes` column in my CSV and disclose this in the
AI usage section of my README.

### Failure analysis
After fine-tuning, I'll paste my misclassified test examples into Claude and ask it
to identify patterns — post length, sarcasm, topic signals, etc. I'll verify any
identified patterns by re-reading the examples myself and include my findings
(including corrections) in the evaluation report.
