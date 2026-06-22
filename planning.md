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

### Case 1 — The reasoned reaction
**Example post:** "The worst thing VAR has done is make everyone go crazy when fouls 
aren't given for borderline minimal contact decisions. The first couple of years of VAR 
they were way too likely to give penalties for microscopic contact on players. Now 
they're trying to move away from that to make the threshold for contact much higher, 
which is the right thing to do. They just should have done it this way to start with."

Could be `reaction` (expressing frustration about VAR) or `analysis` (makes a structured 
argument about how VAR policy has evolved).

**Decision rule:** If the primary purpose is expressing a feeling about a broader 
frustration, label it `reaction` even if reasoning is present. Reserve `analysis` for 
posts where the argument is the *point*, backed by verifiable evidence.
**Decision:** `reaction`

---

### Case 2 — The tactical observation that reads like a reaction
**Example post:** "A lot of these matches involve 1 team scoring a few goals in the 
first half and then parking the bus for 45+ minutes and I'm getting really really 
tired of it."

Could be `analysis` (tactical observation about match patterns) or `reaction` 
(expressing fatigue/frustration).

**Decision rule:** If the post identifies a specific pattern across multiple matches 
grounded in what's happening on the pitch — even without stats — label it `analysis`. 
If the frustration is the main point, label it `reaction`.
**Decision:** `analysis`

---

### Case 3 — The opinion backed by a mild stat reference
**Example post:** "De Jong actually plays a lot of progressive passes though. If someone 
actually thinks he's not a good player, you should probably disregard their footballing 
input."

Could be `hot_take` (dismissive, provocative) or a borderline `reaction` (responding 
to someone else's opinion).

**Decision rule:** If the evidence is vague and the conclusion is dismissive rather 
than argued, label it `hot_take`. The stat gesture here ("a lot of progressive passes") 
is not specific enough to qualify as analysis, and the post is not reacting to a match 
event.
**Decision:** `hot_take`

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
