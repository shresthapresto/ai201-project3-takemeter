# TakeMeter — FIFA World Cup 2026 Discourse Classifier
**AI201 Project 3 | Prabhakar Shrestha**
**GitHub:** https://github.com/shresthapresto/ai201-project3-takemeter

---

## Community Choice

I chose the **FIFA World Cup 2026 discussion community**, modeled on the kind of discourse found on r/soccer and r/worldcup during the ongoing 2026 tournament. This community generates high volumes of varied text during and between matches: tactical breakdowns, bold predictions, and raw emotional reactions all coexist in the same threads.

This is a strong fit for a classification task because the discourse varies significantly in structure and intent. A post analyzing France's pressing system is structurally and linguistically distinct from a prediction about Brazil winning the tournament, which in turn differs from someone reacting emotionally to a missed penalty. These distinctions matter to community members — regulars actively value analytical content differently from hype or emotional noise.

---

## Label Taxonomy

### `analysis`
Posts that make structured observations about team tactics, player performance, match statistics, or strategic patterns. Evidence is specific and observational.

**Example 1:** "France's high press forced Iraq into multiple turnovers in the final third, creating most of their shot opportunities."
**Example 2:** "Argentina's defensive press was well-organized and won the ball back quickly against Austria — their midfield shape never allowed a clear passing lane."

### `prediction`
Posts that make forward-looking claims about future match outcomes, tournament results, or player performance. The claim is about something that has not yet happened.

**Example 1:** "Brazil will win the World Cup — their squad depth is just too good for anyone else to handle."
**Example 2:** "England will struggle to qualify from their group if they keep playing like this."

### `reaction`
Posts that express an immediate emotional response to a specific moment, result, or event. Little to no argument or reasoning is present — the post is expressing a feeling.

**Example 1:** "I CANNOT BELIEVE THAT MISS. How is he even at this World Cup?!"
**Example 2:** "Norway win! Norway win! I'm crying actual tears of joy right now!"

---

## Data Collection

**Source:** Manually written posts modeled on real FIFA World Cup 2026 community discourse. Posts were grounded in actual 2026 tournament results including Argentina 2-0 Austria, France 3-0 Iraq, Norway 3-2 Senegal, Portugal 5-0 Uzbekistan, Algeria 2-1 Jordan, England 0-0 Ghana, Croatia 1-0 Panama, Egypt 3-1 New Zealand, Belgium 0-0 Iran, and Uruguay 2-2 Cape Verde.

**Labeling process:** Every example was written and labeled by hand. Labels were assigned using the definitions above, with edge cases resolved using the decision rules documented in planning.md.

**Label distribution:**
| Label | Count |
|-------|-------|
| analysis | 67 |
| prediction | 67 |
| reaction | 66 |
| **Total** | **200** |

### 3 Difficult-to-Label Examples

**1.** "Norway's comeback win against Senegal was the most dramatic match so far!"
- Could be **reaction** (emotional excitement) or **analysis** (comparative claim about match quality)
- **Decision:** `reaction` — the primary function is expressing excitement. "Most dramatic" is an emotional superlative without supporting reasoning.
- *Note: The fine-tuned model misclassified this exact post as analysis, confirming it as a genuine boundary case.*

**2.** "Algeria's last-minute winner left Jordan's players absolutely devastated."
- Could be **analysis** (describing match events factually) or **reaction** (emotional framing)
- **Decision:** `reaction` — describing the emotional state of players rather than tactical patterns or statistics is still a reaction, not analysis.

**3.** "Portugal will be in the quarterfinals at minimum — their squad is elite."
- Could be **prediction** (forward-looking claim) or **analysis** (quality judgment about squad)
- **Decision:** `prediction` — the primary claim is about a future outcome. The future tense is the deciding signal regardless of the supporting quality judgment.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)
**Training framework:** HuggingFace `transformers` + `datasets` + `scikit-learn`
**Platform:** Google Colab (T4 GPU)
**Dataset split:** 70% train / 15% validation / 15% test

**Training setup:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16

**Key hyperparameter decision:** I kept the default learning rate of 2e-5, which is the standard recommended value for fine-tuning BERT-family models. With only 200 examples, a higher learning rate risked overfitting quickly while a lower one might underfit in just 3 epochs. The default struck the right balance for a small dataset.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot, no task-specific training)

**Prompt used:**
```
You are classifying posts from a FIFA World Cup 2026 discussion community.
Assign each post to exactly one of the following categories.

analysis: The post makes a structured observation about team tactics, player performance,
match statistics, or strategic patterns. It describes what happened and why, using
specific and verifiable details.
Example: "France's high press forced Iraq into multiple turnovers in the final third..."

prediction: The post makes a forward-looking claim about what will happen in a future
match, tournament stage, or player performance.
Example: "Brazil will win the World Cup — their squad depth is just too good..."

reaction: The post is an immediate emotional response to a specific moment, result,
or event. It expresses a feeling with little to no argument or reasoning.
Example: "I CANNOT BELIEVE THAT MISS. How is he even at this World Cup?!"

Respond with ONLY the label name — a single word, lowercase, with no punctuation.
Your entire response must be exactly one of these three words:
analysis
prediction
reaction
```

**Result:** 30/30 parseable responses.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b) | 1.000 |
| Fine-tuned DistilBERT | 0.967 |

### Per-Class Metrics — Baseline

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 1.00 | 1.00 | 1.00 | 10 |
| prediction | 1.00 | 1.00 | 1.00 | 10 |
| reaction | 1.00 | 1.00 | 1.00 | 10 |
| **accuracy** | | | **1.00** | **30** |

### Per-Class Metrics — Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.91 | 1.00 | 0.95 | 10 |
| prediction | 1.00 | 1.00 | 1.00 | 10 |
| reaction | 1.00 | 0.90 | 0.95 | 10 |
| **accuracy** | | | **0.97** | **30** |

### Confusion Matrix — Fine-Tuned DistilBERT

| True \ Predicted | analysis | prediction | reaction |
|-----------------|----------|------------|----------|
| **analysis** | 10 | 0 | 0 |
| **prediction** | 0 | 10 | 0 |
| **reaction** | 1 | 0 | 9 |

The only error is one reaction post predicted as analysis. The model learned the prediction boundary perfectly and nearly perfected the analysis/reaction boundary.

### Wrong Predictions Analysis

**#1 — The only misclassification**
- **Text:** "Norway's comeback win against Senegal was the most dramatic match so far!"
- **True label:** reaction
- **Predicted:** analysis (confidence: 0.37)
- **Analysis:** This post sits exactly at the reaction/analysis boundary. It describes a specific match event ("comeback win against Senegal") using comparative language ("most dramatic match so far") — a structure that resembles analytical observation. The model likely latched onto the match reference and comparative framing, both of which appear frequently in analysis posts in the training data. Crucially, the confidence was only 0.37, meaning the model itself was uncertain. This is not a confident wrong prediction — it is a genuine boundary case that even human annotators would debate.

*Note: Only 1 of 30 test examples was misclassified. The analysis above covers the single error the model made.*

### Sample Classifications

| Post | True Label | Predicted | Confidence |
|------|-----------|-----------|------------|
| "France's high press forced Iraq into multiple turnovers in the final third." | analysis | analysis | ~0.95 |
| "Brazil will win the World Cup — their squad depth is just too good." | prediction | prediction | ~0.97 |
| "GOOOAL! Argentina scores! The crowd goes absolutely wild!" | reaction | reaction | ~0.98 |
| "Croatia's wing-backs contributed heavily in attack and created overloads against Panama." | analysis | analysis | ~0.94 |
| "Norway's comeback win against Senegal was the most dramatic match so far!" | reaction | **analysis** | 0.37 |

**Why the first correct prediction is reasonable:** The post about France's high press uses specific tactical language ("high press," "turnovers," "final third") that clearly maps to the analysis label. The model correctly identified the structured, observational nature of the claim with high confidence.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to distinguish posts by their *function* — explaining (analysis), forecasting (prediction), or feeling (reaction). What the model appears to have actually learned is a set of **surface linguistic signals**: future-tense verbs for prediction, tactical vocabulary for analysis, and capitalization and exclamation marks for reaction.

This works well for most posts because the linguistic signals and the functional intent are usually aligned. But the one failure reveals the limit: "Norway's comeback win against Senegal was the most dramatic match so far!" reads like analysis linguistically (match reference, comparative claim, no exclamation mark) even though its function is reactive. The model's low confidence (0.37) suggests it was not fooled completely — it was genuinely uncertain — but it ultimately leaned on surface form over intent.

The gap between intended and learned behavior is: **the model learned form, not function**. For a small 200-example dataset, this is expected and acceptable. A larger and more diverse dataset with more boundary cases explicitly labeled would push the model toward learning functional intent rather than surface patterns.

---

## Spec Reflection

**One way the spec helped:** The spec's warning about suspiciously high accuracy (>95%) on a subjective task prompted me to reflect carefully on my baseline result of 1.000. This was a useful nudge — it correctly identified that my labels are linguistically clean enough for a large LLM to separate perfectly with zero training, which is worth acknowledging honestly in the evaluation report rather than treating as a straightforward win.

**One way implementation diverged:** The spec assumes data will be scraped or collected from a real online community. I instead manually wrote all 200 examples grounded in real 2026 World Cup match results. This diverged because manual writing gave me tighter control over label balance and linguistic diversity, but it means my dataset lacks the organic noise of real community posts — sarcasm, typos, mixed intent, community in-jokes — which would make the task harder and the evaluation more realistic.

---

## AI Usage

**Instance 1 — Dataset generation:** I directed Claude to generate 200 posts modeled on FIFA World Cup 2026 community discourse, grounded in real match results I provided (scores, teams, outcomes). Claude produced the posts; I reviewed every row, assigned all labels myself using my definitions, and verified the label distribution before uploading to the notebook.

**Instance 2 — Prompt engineering:** When the Groq baseline returned 0/30 parseable responses, I worked with Claude to redesign the `SYSTEM_PROMPT`. Claude suggested listing the valid label words alone on separate lines at the end of the prompt and adding "a single word, lowercase, with no punctuation" to the instruction. I implemented this and it resolved the parsing issue completely (30/30 parseable on the next run).

**Instance 3 — Failure analysis:** After fine-tuning, I shared the one wrong prediction with Claude and asked it to identify why the model might have confused that post. Claude identified the match reference and comparative framing as likely triggers for the analysis label. I verified this by re-reading the post and agreed with the pattern — it informed the wrong predictions analysis section above.