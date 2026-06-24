# Project 3 Planning — TakeMeter
**AI201 | Prabhakar Shrestha**

---

## 1. Community

I chose the **FIFA World Cup 2026 discussion community** — specifically the kind of discourse found on r/soccer and r/worldcup during the ongoing 2026 tournament. This community generates enormous volumes of text during live matches and between games: tactical breakdowns, bold predictions, and raw emotional reactions all coexist in the same threads.

This community is a strong fit for a classification task because the discourse is highly varied in structure and intent. A post about France's pressing system looks completely different from a prediction about Brazil winning the tournament, which in turn looks nothing like someone screaming about a missed penalty. The distinctions are meaningful to people who actually participate — regulars in these communities actively distinguish between informed analysis and emotional noise.

---

## 2. Label Taxonomy

### `analysis`
**Definition:** The post makes a structured observation about team tactics, player performance, match statistics, or strategic patterns. It describes what happened and why, using specific and verifiable details.

**Examples:**
- "France's high press forced Iraq into multiple turnovers in the final third, creating four of their five shot opportunities."
- "Argentina's defensive press was well-organized and won the ball back quickly against Austria — their midfield shape never allowed a clear passing lane."

### `prediction`
**Definition:** The post makes a forward-looking claim about what will happen in a future match, tournament stage, or player performance. The claim is about something that has not happened yet.

**Examples:**
- "Brazil will win the World Cup — their squad depth is just too good for anyone else to handle."
- "England will struggle to qualify from their group if they keep playing like this."

### `reaction`
**Definition:** The post is an immediate emotional response to a specific moment, result, or event. It expresses a feeling with little to no argument or reasoning.

**Examples:**
- "I CANNOT BELIEVE THAT MISS. How is he even at this World Cup?!"
- "Norway win! Norway win! I'm crying actual tears of joy right now!"

---

## 3. Hard Edge Cases

### Edge Case 1: Observational reaction vs. analysis
**Post:** "Norway's comeback win against Senegal was the most dramatic match so far!"

This could be **reaction** (expressing excitement and emotion about the result) or **analysis** (making a comparative claim about match quality across the tournament). The framing is emotional ("dramatic") but the claim is observational and comparative.

**Decision rule:** If the post's primary function is to express a feeling about an event rather than explain what happened tactically or structurally, label it **reaction**. Emotional superlatives ("most dramatic," "unbelievable," "incredible") without supporting reasoning signal reaction. → **reaction**

*Note: The fine-tuned model misclassified this exact post as analysis (confidence: 0.37), which validates this as a genuine boundary case.*

### Edge Case 2: Emotional description vs. reaction
**Post:** "Algeria's last-minute winner left Jordan's players absolutely devastated."

This describes an outcome factually but uses emotional framing ("absolutely devastated"). It could be **analysis** (describing match events) or **reaction** (emotional response).

**Decision rule:** If the post describes the emotional state of players or fans rather than tactical patterns or match statistics, and contains no structured argument, label it **reaction**. Describing an emotional consequence of an event is still a reaction, not analysis. → **reaction**

### Edge Case 3: Quality judgment vs. prediction
**Post:** "Portugal will be in the quarterfinals at minimum — their squad is elite."

This is forward-looking (prediction) but bases the claim on a quality judgment ("their squad is elite") which could read as analysis.

**Decision rule:** If the post's primary claim is about a future outcome, label it **prediction** regardless of whether a supporting quality judgment is present. The future tense is the deciding signal. → **prediction**

---

## 4. Data Collection Plan

**Source:** Manually written posts modeled on real FIFA World Cup 2026 discussion communities (r/soccer, r/worldcup). Real match results from the ongoing 2026 tournament were used to ground the posts in actual events (Argentina 2-0 Austria, France 3-0 Iraq, Norway 3-2 Senegal, Portugal 5-0 Uzbekistan, Algeria 2-1 Jordan, England 0-0 Ghana, Croatia 1-0 Panama, Egypt 3-1 New Zealand, Belgium 0-0 Iran, Uruguay 2-2 Cape Verde).

**Target per label:** ~67 examples each (balanced across 3 labels)

**Final distribution:**
- `analysis`: 67 examples
- `prediction`: 67 examples
- `reaction`: 66 examples

**If a label is underrepresented:** Write additional examples for that label specifically, targeting posts that sit closer to the boundary with another label to increase difficulty.

---

## 5. Evaluation Metrics

**Accuracy** tells me the overall fraction of correct predictions but can be misleading if one class dominates. Since my dataset is balanced (~33% per label), accuracy is meaningful here but still insufficient alone.

**Per-class F1** is the most important metric because it shows whether the model is learning all three distinctions or just doing well on the majority class. A model that gets analysis and prediction right but always fails on reaction would show high accuracy but terrible reaction F1 — I need to catch that.

**Confusion matrix** is essential to understand the direction of errors. Knowing that the model confuses reaction→analysis is more actionable than just knowing it made one mistake.

I will not rely on accuracy alone because all three labels need to be learned well for the classifier to be useful in a real community tool.

---

## 6. Definition of Success

The fine-tuned model should achieve:
- Overall accuracy ≥ 0.85 on the test set
- Per-class F1 ≥ 0.75 for all three labels
- Meaningful improvement over the zero-shot baseline on at least one label

A classifier that hits these thresholds would be genuinely useful for automatically tagging World Cup discussion posts by type — enabling community tools that surface analytical content separately from emotional reactions or speculative takes.

If the fine-tuned model does not beat the baseline, that signals the labels are too easy for a large LLM, the dataset is too small, or annotation is inconsistent.

---

## 7. AI Tool Plan

### Label stress-testing
I used Claude to generate 5–10 boundary posts between reaction and analysis — posts that describe match events in emotional language — to test whether my label definitions could handle them consistently before annotating 200 examples.

### Annotation assistance
The dataset was manually written rather than scraped. Claude assisted in generating posts grounded in real 2026 World Cup match results. Every post was reviewed and labeled by me. This is disclosed in the README AI usage section.

### Failure analysis
After fine-tuning, I will paste the wrong predictions into Claude and ask it to identify common themes — post length, emotional language, borderline framing — then verify the patterns by re-reading the examples myself before writing the evaluation report.

---

## 8. Stretch Features

*(To be updated if any stretch features are attempted)*

- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [ ] Deployed interface