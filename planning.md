# TFT Discourse Classifier — Project Plan

## Community

r/teamfighttactics is the primary Reddit community for Teamfight Tactics (TFT), Riot Games'
auto-battler game. The subreddit hosts a wide range of post types: detailed comp guides with
stage-by-stage instructions, mechanical explanations, balance complaints, and casual opinions
about the meta. This diversity makes it well-suited for a classification task — the discourse
spans from highly substantive (step-by-step reroll guides) to near-zero-signal (venting about
RNG). The game's complexity also creates natural variation in depth: some posts explain
interactions that require real game knowledge to interpret, while others are accessible to
anyone. The mix of expertise levels and post intents produces labels that are meaningfully
distinct rather than arbitrary.

## Labels

Four labels, ordered high to low discourse quality:

**guide** — A guide post is primarily instructional: it explains how to execute a specific
comp, mechanic, or play pattern with the intent to change what the reader does in-game.
Must be actionable and specific; vague encouragement or general tips do not qualify.

- *Example 1*: "Roll on spikes — at 3-5 roll for 2-star if paired, this will save a lot of
  health; slow roll stage 4; all-in by 5-1 or 7 copies whichever comes first. If you miss
  it's 8th."
- *Example 2*: "Morgana should be switched to Rhaast — only go Morgana if Nunu is gone and
  Tahm is contested. Then rotate hexes: Morgana→Nunu, Nunu→Blitzcrank, Blitzcrank→Fiora."

**analysis** — An analysis post objectively conveys or examines information about the game,
ranging from stating a mechanic directly to reasoning about gameplay data or interactions.
The primary intent is to explain or understand, not to instruct.

- *Example 1*: "Bard copying the PvE round gives you Pengu." (mechanic statement)
- *Example 2*: "At some point playing more isn't the answer — consuming patch notes and
  streams breaks assumptions you didn't know you had, like whether it's worth sacking gold
  to hold a pair on stage 2." (conceptual examination)

**opinion** — An opinion post asserts a subjective position on game balance, meta, unit
design, or strategy without grounding it in evidence. The post contributes a claim to the
conversation but does not argue for it rigorously.

- *Example 1*: "Apex Primordian is the best Prismatic augment of all time."
- *Example 2*: "Kench feels more like a 5-cost unit — he's just too impactful for his
  price point."

**complaint** — A complaint post is primarily an expression of frustration or negativity.
The emotional response is the point; any game-related claim is secondary to the venting.

- *Example 1*: "I rerolled 22 times and got 0 Bows. What the F*** is going on with this
  game."
- *Example 2*: "I'm so sick of losing to the same broken comp every game, this patch is
  terrible."

### Decision Rules

**Guide vs. analysis**: Does the post want the reader to do something specific and
actionable in-game? If yes, label it guide. If it is examining or explaining without
instructing, label it analysis.

**Analysis vs. opinion**: Strip the opinion framing. If a real, evaluable claim grounded
in gameplay observation remains (a stat, a mechanic, a systematic pattern), label it
analysis. If the support is itself another untested assumption, label it opinion.

**Opinion vs. complaint**: Strip the emotional language. If a claim about the game remains
(even without evidence), label it opinion. If nothing is left but the frustration, label
it complaint.

When genuinely uncertain between two adjacent labels, choose the lower-quality one — the
bar for guide and analysis should be high.

## Hard Edge Cases

The most genuinely ambiguous boundary is **opinion vs. analysis**. Many posts offer a
position supported by a reason, but the reason is an untested intuition rather than a
systematic observation. For example: "I think the AP scaling is better because you can
tack on attack speed from augments and Rageblade" — this gives a reason, but it is a
general game principle rather than a data point or verifiable observation. Apply the
decision rule: strip the framing and ask whether the remaining claim is evaluable. If the
support is itself just another opinion, label the whole post opinion.

A secondary edge case is **guide vs. analysis** for meta-improvement advice. A post
explaining *how* to get better at TFT (e.g., "consume content rather than just grinding
games — you'll break assumptions you didn't know you had") examines a meta-question rather
than teaching a specific play pattern. The decision rule: does the post want the reader to
do something specific and actionable in-game? If yes, guide. If it is analyzing a question
about learning or the game system, analysis.

Hard case on file:

> "I hit masters every set and I feel like at some point playing more isn't the answer and
> instead your time is better spent consuming content whether it's patch notes, recaps, or
> watching streamers. And more often than not you learn things that you otherwise never
> questioned — like, wow was it really worth sacking one gold to hold a pair on stage 2?"

Labeled **analysis**: examines a meta-question about how players improve rather than
teaching a specific play pattern, even though the intent is to help the reader.

## Data Collection Plan

**Source**: r/teamfighttactics, collected via the Reddit API (PRAW) targeting top and new
posts plus comment threads on high-engagement posts (questions, comp showcases).

**Target**: ~50 examples per label (~200 total). Current counts after 119 labeled examples:

| Label     | Count |   %  |
|-----------|------:|-----:|
| opinion   |    65 | 54.6 |
| complaint |    22 | 18.5 |
| analysis  |    24 | 20.2 |
| guide     |     8 |  6.7 |

Note: the 15 posts previously labeled `fact` have been merged into `analysis` to bring the
label count within the 4-label constraint. Both labels cover objective, non-instructional
content; the distinction between stating a mechanic and reasoning about it is a spectrum.

**If a label is underrepresented after 200 examples**: Guide is already at risk. If it
remains below ~30 examples, I will actively target posts containing comp-specific keywords
("roll at", "fast 8", "slow roll", "BIS items") and comment threads responding to "how do
I play X" questions, where instructional replies are common. If opinion remains
overrepresented above ~40%, I will cap collection from opinion-heavy threads (patch
reaction megathreads) and prioritize guide and analysis sources.

## Evaluation Metrics

**Primary metric: macro F1**. Macro F1 averages per-class F1 scores without weighting by
class size, which penalizes the model equally for failing on minority classes (guide,
analysis) as for the dominant one (opinion). Accuracy alone is insufficient — a model that
always predicts *opinion* scores 54.6% accuracy while being completely useless.

**Per-class F1**: reported for each label individually to expose which distinctions the
model fails on. A high macro F1 that hides a near-zero guide F1 is not acceptable for a
tool meant to surface quality content.

**Confusion matrix**: to identify systematic misclassification patterns — e.g., whether
the model conflates opinion and complaint (emotionally similar surface features) or guide
and analysis (both substantive, both non-emotional).

The majority-class baseline (always predict *opinion*) achieves ~54.6% accuracy and macro
F1 of ~0.18 (non-zero only for the opinion class). Any useful model must substantially
beat this on macro F1.

## Definition of Success

A classifier is genuinely useful for a community tool — e.g., surfacing high-quality posts
or filtering low-signal content — if it can reliably separate substantive content (guide,
analysis) from noise (complaint) and unsubstantiated takes (opinion).

**Minimum acceptable threshold ("good enough for deployment")**:
- Macro F1 ≥ 0.65
- Per-class F1 ≥ 0.55 for guide and analysis (the minority classes that matter most for
  surfacing quality content)
- Per-class F1 ≥ 0.70 for opinion and complaint (the majority classes where errors are
  most costly at scale)

**Target ("genuinely useful")**:
- Macro F1 ≥ 0.75
- No per-class F1 below 0.60

These thresholds are objectively measurable on a held-out test set. If macro F1 is ≥ 0.65
and no per-class F1 falls below 0.55, the classifier meets the deployment bar. Anything
below 0.65 macro F1 means the model is not reliable enough to trust as a community tool
without heavy human review.

## AI Tool Plan

### Label Stress-Testing

Before annotating the remaining unlabeled examples, I will give Claude the four label
definitions and both edge case descriptions, and ask it to generate 5–10 posts that sit at
the boundary between opinion/analysis and between guide/analysis. If it produces posts I
cannot classify cleanly using the current decision rules, the definitions need tightening
before annotation continues. Specifically:

- Ask for 5 posts that could be either *opinion* or *analysis*
- Ask for 5 posts that could be either *guide* or *analysis*

If more than 1–2 per set are genuinely ambiguous, revise the relevant decision rule and
re-test with a fresh batch before annotating.

### Annotation Assistance

I will use Claude (claude-sonnet-4-6) to pre-label the unlabeled examples in
`tft_unlabeled.csv` before reviewing them myself. The prompt will include the full label
definitions and decision rules from this document. Each pre-labeled example will be tracked
with a `prelabeled` column in the CSV so the final dataset clearly distinguishes
human-only labels from human-reviewed AI labels. I will review every pre-labeled example
and correct any I disagree with before training.

### Failure Analysis

After evaluation, I will pass the full list of wrong predictions — with the true label,
predicted label, and post text — to Claude and ask it to identify patterns in the
misclassifications. Specifically, I will ask:

- Are errors concentrated in a specific label pair (e.g., opinion predicted as complaint)?
- Is there a surface feature (post length, presence of numbers, hedging language) correlated
  with errors?
- Do the misclassified examples share a structural pattern the label definitions do not
  currently cover?

I will verify any pattern Claude identifies by manually counting whether it holds across
the full error set before including it in the writeup.
