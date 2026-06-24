BASELINE RESULTS
🎯 Baseline accuracy: 0.607  (evaluated on 28/28 parseable responses)

Per-class metrics (baseline):
              precision    recall  f1-score   support

       guide       0.00      0.00      0.00         3
    analysis       0.33      0.17      0.22         6
     opinion       0.63      0.92      0.75        13
   complaint       0.80      0.67      0.73         6

    accuracy                           0.61        28
   macro avg       0.44      0.44      0.42        28
weighted avg       0.54      0.61      0.55        28

seemed to have a terrible time determining guide and alanlysis. Got opinion correct often, but guessed it very often as well. complaint is mostly solid.


Fine-tuned mistakes
Wrong predictions: 15 / 28

--- #1 ---
Text:      Is there any cool/unique comps with bilge water in pengu?

I already played something with pretty much all the traits but i can't seem to build something with bilge water to make it work. Has anyone o...
True:      guide
Predicted: opinion  (confidence: 0.36)

--- #2 ---
Text:      TF 3 in a maxxed out The Crew team
True:      analysis
Predicted: opinion  (confidence: 0.33)

--- #3 ---
Text:      Leblanc arbiter

Im always confused which arbiter is better for leblanc, attack speed or ap, and if you need two as items if you pick up as from arbiter or is it better to go something like rageblade ...
True:      guide
Predicted: opinion  (confidence: 0.36)

--- #4 ---
Text:      Some subreddits are just an ego clown fest. Instead of actually helping and being a community lmao.
True:      complaint
Predicted: opinion  (confidence: 0.35)

--- #5 ---
Text:      Finally hit 6 ANIMA

Finally hit 6 ANIMA + perfect comp... but I just couldn't 3-star Fiora 😭
True:      complaint
Predicted: opinion  (confidence: 0.36)

--- #6 ---
Text:      I took thresh’s boon 2-4 with Apex primordian augment -> mini recombobulate

I did not consider this possibility 🤣 waiting for 3-2 to FF as i type this
True:      analysis
Predicted: opinion  (confidence: 0.34)

--- #7 ---
Text:      I did almost all of them with Pengu's Party, really helpful getting a 3* 5 cost.
True:      guide
Predicted: opinion  (confidence: 0.34)

--- #8 ---
Text:      5 new units for next set revealed

Saw these during the state of unreal showcase. Thought i’d share.
True:      analysis
Predicted: opinion  (confidence: 0.36)

--- #9 ---
Text:      Same way you can get elder dragon, by having bard steal the end stage boss
True:      analysis
Predicted: opinion  (confidence: 0.35)

--- #10 ---
Text:      not calculated ?

what did i even die from ??
True:      complaint
Predicted: opinion  (confidence: 0.34)

--- #11 ---
Text:      Congrats challenge is insane
True:      complaint
Predicted: opinion  (confidence: 0.35)

--- #12 ---
Text:      Anyone how i lost with 6 anima that i got on stage 4-2?

I got a feeling it's about some nova strike that he used but still, i got no clue how that beats my board.
True:      complaint
Predicted: opinion  (confidence: 0.36)

--- #13 ---
Text:      Did they change how nunu angles his snowball?

This Patch I have seen it multiple times that my nunu just rolls his snowball towards my side of the board while missing the entire enemy team.

&#x200B;...
True:      complaint
Predicted: opinion  (confidence: 0.35)

--- #14 ---
Text:      All the mission that doesn't include "Player Combat" in their description
True:      analysis
Predicted: opinion  (confidence: 0.33)

--- #15 ---
Text:      They do them regularly.  In fact Remix Rumble was one of the throwback sets last year
True:      analysis
Predicted: opinion  (confidence: 0.34)

ANALYSIS

The baseline had fairly good results on its own which leads me to believe the labels and data are alright. I believe something was wrong with the fine tuning setup, but whatever I did I could not find a way to resolve the issue. Turning down the learning rate caused every response to be guide. Adding epochs made no difference. I had no memory errors that would require changing the per_device_train_batch_size. It is difficult to provide strong analysis of my results because of this training error. The fine tuned model exclusively predicts opinion regardless of the text. I believe this may have been caused by the skew towards opinions in the data, but opinions made up only 47.8% of the data distribution.

---

# TakeMeter — TFT Post Classifier
### AI201 · Project 3

A text classifier that labels r/teamfighttactics Reddit posts into one of four categories: **analysis**, **guide**, **opinion**, and **complaint**. The project compares a zero-shot Groq baseline against a DistilBERT model fine-tuned on hand-labeled posts.

---

## Label Schema

| Label | Definition |
|---|---|
| **analysis** | Evidence-backed argument using stats, data, or systematic observation |
| **guide** | Instructional content for a comp, mechanic, or play pattern |
| **opinion** | Subjective take on balance/meta with no supporting evidence |
| **complaint** | Primarily venting frustration; minimal constructive content |

The hardest boundary to enforce in practice is **opinion vs. complaint**: both are subjective and negative, but complaint implies the post's primary purpose is venting, while opinion makes a claim (even if a weak one). Similarly, **guide vs. analysis** both involve TFT knowledge but differ in intent — guides teach, analyses argue.

---

## Dataset

- **Source:** r/teamfighttactics (posts and top comments scraped with `collect_tft.py`)
- **Total labeled examples:** 186 (after removing 34 "skip" entries)
- **Labeling method:** Human-in-the-loop with Groq (`llama-3.3-70b-versatile`) suggesting a label; human confirmed or overrode each one using `label_hil.py`

| Label | Count | % |
|---|---|---|
| opinion | 89 | 47.8% |
| complaint | 39 | 21.0% |
| analysis | 36 | 19.4% |
| guide | 22 | 11.8% |

The dataset is skewed toward opinion (~48%), reflecting the natural distribution on r/TFT — most posts are people sharing hot takes rather than writing guides.

**Train / val / test split:** 70% / 15% / 15% stratified by label (≈130 / 28 / 28 examples).

---

## Model Architecture and Training

**Base model:** `distilbert-base-uncased` with a 4-class classification head  
**Tokenizer:** DistilBERT tokenizer, max length 256 tokens  
**Training framework:** HuggingFace Trainer on Colab T4 GPU

**Hyperparameters used:**
- `num_train_epochs`: 3
- `learning_rate`: 2e-5
- `per_device_train_batch_size`: 16

No hyperparameters diverged from the starter defaults. Multiple adjustments were attempted to fix the opinion-collapse problem (lowering learning rate, adding epochs), but none resolved it within the scope of this project.

**Baseline:** Zero-shot `llama-3.3-70b-versatile` via Groq API. The baseline prompt defined all four labels with examples and instructed the model to return only the label string.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy | Test Examples |
|---|---|---|
| Groq zero-shot (baseline) | **0.607** | 28 |
| DistilBERT fine-tuned | **0.464** | 28 |

The fine-tuned model underperformed the zero-shot baseline by 14 percentage points. The cause is label collapse: the fine-tuned model predicted **opinion** for all 28 test examples.

---

### Per-Class Metrics

**Baseline (Groq zero-shot):**

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.33 | 0.17 | 0.22 | 6 |
| complaint | 0.80 | 0.67 | 0.73 | 6 |
| guide | 0.00 | 0.00 | 0.00 | 3 |
| opinion | 0.63 | 0.92 | 0.75 | 13 |
| **accuracy** | | | **0.61** | 28 |
| macro avg | 0.44 | 0.44 | 0.42 | 28 |
| weighted avg | 0.54 | 0.61 | 0.55 | 28 |

**Fine-Tuned DistilBERT:**

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.00 | 0.00 | 0.00 | 6 |
| complaint | 0.00 | 0.00 | 0.00 | 6 |
| guide | 0.00 | 0.00 | 0.00 | 3 |
| opinion | 0.46 | 1.00 | 0.63 | 13 |
| **accuracy** | | | **0.46** | 28 |
| macro avg | 0.12 | 0.25 | 0.16 | 28 |
| weighted avg | 0.21 | 0.46 | 0.29 | 28 |

---

### Confusion Matrix (Fine-Tuned Model)

Rows = true label, columns = predicted label.

| True \ Predicted | analysis | complaint | guide | opinion |
|---|---|---|---|---|
| **analysis** | 0 | 0 | 0 | 6 |
| **complaint** | 0 | 0 | 0 | 6 |
| **guide** | 0 | 0 | 0 | 3 |
| **opinion** | 0 | 0 | 0 | 13 |

Every prediction is "opinion." The model learned that predicting the majority class minimizes training loss on this imbalanced dataset and converged to that degenerate solution.

---

### AI-Assisted Pattern Analysis

To surface patterns in the 15 misclassified examples, I pasted them all into Groq and asked: *"What common themes do you see in these misclassified posts? Look at post length, tone, label pairs, and any structural features."*

Groq identified three patterns:

1. **Short / low-information posts** — Many misclassified posts are one or two sentences with no explicit structural markers. Without evidence, instructions, or explicit frustration language, the model defaults to "opinion."
2. **Subtle complaints** — Complaints that express frustration through implication or humor rather than direct venting language get mislabeled. The opinion/complaint boundary is tonally close.
3. **Questions labeled as guides** — Several "guide" examples are questions *seeking* advice rather than posts *giving* it.

After re-reading the examples myself, I confirmed patterns 1 and 2 but partially discarded pattern 3 — only two of the three guide misclassifications are question-format posts; the third (#7) is a short declarative tip that is guide-like but too brief to signal its category.

---

### In-Depth Error Analysis

#### Example 1: guide → opinion (confidence 0.36)

> *"Is there any cool/unique comps with bilge water in pengu? I already played something with pretty much all the traits but i can't seem to build something with bilge water to make it work. Has anyone o..."*

**Which labels are confused?** guide → opinion. This accounts for 2 of the 3 guide failures.

**Why is this boundary hard?** This post is a *question asking for* a guide, not a guide itself. The label definition ("instructional content") describes the producer's intent, but the post reads as a player seeking advice. The model has no signal that this belongs in "guide" — there are no instructions, no comp breakdowns, no tips.

**Labeling or data problem?** Labeling inconsistency. If "guide" means posts about how to play something, then a question about how to play something is borderline. The training data likely contains both question-format and instruction-format guide posts, which muddies the boundary.

**What would fix it?** Tighten the label definition: either explicitly include question-format posts and add more question examples to training, or restrict "guide" to instructional posts only and relabel questions as something else.

---

#### Example 2: complaint → opinion (confidence 0.36)

> *"Finally hit 6 ANIMA + perfect comp... but I just couldn't 3-star Fiora 😭"*

**Which labels are confused?** complaint → opinion. This is the most common error pair: 6 of 15 misclassifications are complaint posts predicted as opinion.

**Why is this boundary hard?** This post expresses disappointment but doesn't vent explicitly. The structure — "I achieved X but not Y" — looks like a neutral observation or opinion to a model trained on short examples. Complaints using humor or understatement are structurally almost identical to opinions.

**Labeling or data problem?** Primarily a data distribution problem. The model saw ~39 complaint examples in training versus ~62 opinions. It never learned a reliable signal for subtle complaint. The fix is more complaint examples covering the range of how TFT players vent — not just obvious "this is broken" posts, but understated and sarcastic ones too.

**What would fix it?** Add 20–30 more complaint examples that vary in expression style so the model sees the full distribution of how complaints are expressed, not just the most explicit cases.

---

#### Example 3: analysis → opinion (confidence 0.36)

> *"5 new units for next set revealed — Saw these during the state of unreal showcase. Thought i'd share."*

**Which labels are confused?** analysis → opinion. 6 of 15 errors are analysis posts predicted as opinion — the largest single error group.

**Why is this boundary hard?** This post is extremely short and contains no data, no argument, and no systematic observation. It's a factual share. The label "analysis" was applied because it conveys game-state information, but the content itself provides no analysis by the stated definition.

**Labeling or data problem?** This is a labeling problem. A post that says "here are new units" is not analysis by the stated definition — there's no argument, no evidence, no conclusion. If examples like this were labeled "analysis" during annotation, the model received contradictory training signal: some analysis posts are long and evidence-heavy, others are one-liners with no reasoning.

**What would fix it?** Audit analysis labels and relabel short informational posts that lack argument or evidence. Tighten the definition to require a minimum of reasoning ("X is true because Y"), not just factual statements.

---

### Sample Classifications

Five posts run through the fine-tuned model (all predicted as "opinion" due to label collapse):

| Post (truncated to 60 chars) | True Label | Predicted | Confidence |
|---|---|---|---|
| "Is there any cool/unique comps with bilge water in pengu?…" | guide | opinion | 0.36 |
| "TF 3 in a maxxed out The Crew team" | analysis | opinion | 0.33 |
| "Some subreddits are just an ego clown fest…" | complaint | opinion | 0.35 |
| "Finally hit 6 ANIMA + perfect comp... but I just couldn't 3-star Fiora 😭" | complaint | opinion | 0.36 |
| "Congrats challenge is insane" | complaint | opinion | 0.35 |

Confidence scores are uniformly low (~0.33–0.36), reflecting a nearly flat softmax distribution across four classes — the model has no strong learned boundary and opinion narrowly wins every time.

**Correctly predicted example:** All 13 correctly predicted test examples were true opinion posts. For instance, a short subjective take like "Congrats challenge is insane" gets predicted as opinion — which is wrong here (it's a complaint), but when a true opinion post has the same low-information structure, the prediction happens to be correct. The model's "success" on opinion is entirely explained by the majority-class bias, not by learning what makes a post an opinion.

---

## Model Reflection

The fine-tuned model learned to predict the majority class rather than the actual decision boundaries between labels. This is the gap between what I intended to capture — four structurally distinct ways TFT players communicate — and what the model actually captured: "almost half of all posts are opinions, so predict opinion and get 46% accuracy."

The boundaries I defined are real and meaningful to a human reader. The difference between an evidence-backed analysis and an unsupported hot take is clear in context. But the model didn't learn those boundaries because:

1. **The training signal was too noisy.** With ~130 training examples split four ways (~62 opinion, ~27 complaint, ~25 analysis, ~15 guide), minority classes don't have enough examples to generalize from — especially when some are inconsistently labeled at the opinion/complaint and guide/opinion edges.
2. **Short TFT posts don't carry enough signal.** Many posts are 1–3 sentences. When a complaint, an analysis, and an opinion post are all similarly short and casual in tone, the lexical features overlap heavily.
3. **The model overfit to class frequency.** Cross-entropy loss on an imbalanced dataset rewards predicting the majority class. Without class weighting or oversampling, the model converged to a degenerate solution.

What would be needed to capture the intended boundaries: class-weighted loss, at least 60–80 examples per class, tighter label definitions that eliminate ambiguous edge cases, and a review pass on the auto-labeled entries.

---

## Spec Reflection

**Where the spec helped:** The spec's requirement to define labels before collecting data was the right constraint. Having concrete definitions for all four labels before annotating forced me to think through the opinion/complaint boundary early — which is the boundary the model struggles with most. Without that upfront definition, annotation would have been more inconsistent from the start.

**Where my implementation diverged:** The spec assumed a roughly balanced dataset of 200+ examples with consistent label coverage. My dataset ended up with opinion at 48%, more than four times the guide count. This happened because the natural post distribution on r/TFT is skewed — most people share opinions, not guides. I didn't compensate by oversampling guides or applying class weights during training, which is the root cause of the label collapse. If I were doing this again, I would either stratify data collection (actively seek guide posts) or use class-weighted loss in the Trainer regardless of what the spec said about default hyperparameters.

---

## AI Usage

**Instance 1 — Human-in-the-loop annotation (`label_hil.py`):**
I directed `llama-3.3-70b-versatile` (Groq) to suggest a label and one-sentence rationale for each unlabeled post, using a system prompt with all four label definitions. For each of the ~100 posts labeled this way, I either accepted Groq's suggestion or overrode it. I overrode roughly 20–25% of suggestions — mostly cases where Groq predicted "opinion" for a post I read as "complaint." This confirmed that the opinion/complaint edge is genuinely hard, not just a model artifact.

**Instance 2 — Automated labeling for the remaining posts (`label_auto.py`):**
After the human-in-the-loop session, I used `label_auto.py` to automatically label the remaining posts without human review, accepting all of Groq's labels directly. In retrospect, this was a mistake: posts labeled this way received no human verification, which likely contributed to annotation noise — particularly on ambiguous opinion/complaint and guide/opinion cases. If I were doing this again I would review at least a sample of the auto-labeled entries before training.

**Instance 3 — Error pattern analysis:**
After collecting the 15 fine-tuned model misclassifications, I pasted them into Groq and asked it to identify common themes by post length, tone, and label pair. It surfaced three patterns (short posts, subtle complaints, question-format guides). I verified each manually and discarded the claim that all three guide errors were question-format posts — only two were; the third was a short declarative tip that was genuinely guide-like but too brief to signal its category.

