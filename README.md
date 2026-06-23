# TakeMeter — IPL Discourse Quality Classifier

A fine-tuned text classifier that evaluates the discourse quality of comments from the r/IPL (Indian Premier League) subreddit.

---

## Community Choice

**Community:** r/IPL (Indian Premier League subreddit)

The IPL subreddit is an active, text-heavy fan community centered on the Indian Premier League cricket tournament. It was chosen because discourse quality varies enormously within a single thread — the same post can attract structured statistical arguments, bold unsupported opinions, and emotional reactions. The "Discussion" flair filters for argumentative, opinion-driven posts, making it a reliable source of varied discourse quality across a single consistent source.

This community is a strong fit for a classification task because the distinctions between discourse types are ones that regular participants actively recognize. An IPL fan can immediately tell the difference between someone citing Narine's 2024 MVP stats and someone declaring "Kohli is finished" with no evidence — these are different kinds of contributions to a conversation, and that difference is worth noting.

---

## Label Taxonomy

### `analysis`
The comment makes a structured argument supported by stats, historical examples, rules/policies, or logical reasoning. The claim could be evaluated or challenged based on the evidence provided.

**Examples:**
- *"BCCI strictly does bone testing of every player to determine their age. Due to bone testing we have a margin of +-2 years. They literally banned Rasikh Dar for the same. And also Manjot Kalra (U19 final MoM was banned)."* — cites specific policy, specific banned players, and specific margin of error.
- *"Most IPL fans don't buy original jerseys, opting instead for cheaper replicas. That makes it difficult for brands like Nike, Adidas, Puma, or Reebok to generate enough sales to justify large sponsorship deals. Add to that the pressure of accommodating multiple sponsor logos on the jersey and the relatively short IPL season, and the business case becomes even less attractive."* — explains jersey sponsor problem through market reasoning.

### `hot_take`
A confident opinion or claim stated without meaningful supporting evidence. The comment asserts a position but doesn't back it up with stats, history, or structured reasoning.

**Examples:**
- *"I don't think he is faking his age!"* — bold dismissal with no reasoning.
- *"The IPL jerseys colour + branding combination is amongst the worst I've seen in sports."* — strong opinion with no comparative evidence.

### `reaction`
An immediate emotional or nostalgic response to a specific event, player moment, or memory. Little to no argument — the comment is expressing a feeling rather than making a case.

**Examples:**
- *"That Aircel logo meant domination back in the days. IPL was the tournament where 7 teams fight to get a chance to meet CSK in playoffs. Chennai was so deadly 2010-2013..."* — pure nostalgia, no argument.
- *"LETS GO RCB WON THE FINALS"* — immediate emotional reaction to a result.

---

## Data Collection

**Source:** r/IPL subreddit, filtered by "Discussion" flair

**Method:** Manual collection across 21 different discussion posts covering a variety of topics (age fraud allegations, player form debates, rule changes, team selection, match reactions, jersey design). Comments were collected across a range of upvote levels — not just top comments — to ensure variety in discourse quality.

**Total examples:** 230

**Label distribution:**

| Label | Count | Percentage |
|---|---|---|
| analysis | 123 | 53.5% |
| hot_take | 66 | 28.7% |
| reaction | 41 | 17.8% |

**Note on distribution:** `reaction` comments are naturally underrepresented in discussion-flair posts, which attract more argumentative content by design. Despite targeted collection from match threads and fan reaction posts, reactions remained below the 20% target. This imbalance contributed to the model's majority-class collapse during fine-tuning (see Evaluation Report).

**Labeling process:** All 230 examples were labeled manually by the primary annotator (me). An AI tool (Claude) was used for posts that were difficult for me to label. I then reviewed and verified Claude's label and overrode if needed. The final label decision was always mine (the annotator's).

---

## Difficult-to-Label Examples

**Example 1:**
> *"I agree RCB is simply the better team"*

Could be **reaction** (expressing fan emotion) or **hot_take** (asserting a comparative claim about team quality). Decision: **hot_take** — it's making a standing claim rather than reacting to a specific moment or event. Reactions are tied to something that just happened; this is a general opinion.

**Example 2:**
> *"You are same kind of low life person. There is no need to support or non support age fraud. He did not commit age fraud. I know he is 1.5 years older. But i don't care."*

Could be **analysis** (contains a factual concession — acknowledges the 1.5-year age difference) or **hot_take** (emotional, dismissive throughout). Decision: **hot_take** — the factual acknowledgment is used to dismiss the argument rather than engage with it. Per decision rule: a fact used decoratively rather than as part of a logical argument → hot_take.

**Example 3:**
> *"IPL hastened it, but Kohli truly led the WWE-Ification of cricket."*

Could be **analysis** (structured metaphorical claim) or **hot_take** (bold assertion, no evidence). Decision: **hot_take** — despite the creative framing, no evidence or reasoning is provided to support the claim. A well-constructed sentence is not the same as a well-supported argument.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased`
**Training setup:**
- Dataset split: 70% train (161), 15% validation (34), 15% test (35) — stratified
- GPU: Google Colab T4 (free tier)


**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)

**Prompt used:**

```
You are classifying comments from the r/IPL subreddit (Indian Premier League cricket community).
Assign each post to exactly one of the following categories.

analysis: The comment makes a structured argument supported by stats, historical examples,
rules/policies, or logical reasoning. The claim could be evaluated or challenged based on
the evidence provided.
Example: "BCCI strictly does bone testing of every player to determine their age..."

hot_take: A confident opinion or claim stated without meaningful supporting evidence.
Example: "Virat is finished, he hasn't looked the same since 2021, change my mind."

reaction: An immediate emotional or nostalgic response to a specific event, player moment,
or memory. Little to no argument — the comment is expressing a feeling rather than making a case.
Example: "LETS GO RCB WON THE FINALS"

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
analysis
hot_take
reaction
```

**Collection method:** Run via Groq API on the test set (35 examples) in Section 5 of the Colab notebook. All 35 responses were parseable — no unparseable outputs.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | 51.4% |
| Fine-tuned DistilBERT | 54.3% |
| Improvement | +2.9% |

### Per-Class Metrics — Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.64 | 0.47 | 0.55 | 19 |
| hot_take | 0.31 | 0.50 | 0.38 | 10 |
| reaction | 0.80 | 0.67 | 0.73 | 6 |
| **macro avg** | 0.59 | 0.55 | 0.55 | 35 |

### Per-Class Metrics — Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.54 | 1.00 | 0.70 | 19 |
| hot_take | 0.00 | 0.00 | 0.00 | 10 |
| reaction | 0.00 | 0.00 | 0.00 | 6 |
| **macro avg** | 0.18 | 0.33 | 0.23 | 35 |

### Confusion Matrix — Fine-Tuned Model

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 19 | 0 | 0 |
| **True: hot_take** | 10 | 0 | 0 |
| **True: reaction** | 6 | 0 | 0 |

The model predicted `analysis` for every single test example. It correctly identified all 19 analysis examples but missed all 10 hot_takes and all 6 reactions.

### Wrong Predictions Analysis

**Wrong prediction #1:**
> *"I honestly don't care about it being religious. It's part of our culture. People don't bat an eye when every other Bollywood song is in urdu..."*
- **True label:** hot_take | **Predicted:** analysis | **Confidence:** 0.44

**Why it failed:** This comment has the structural markers of analysis — it's multi-sentence, uses a cultural comparison as a rhetorical device, and engages with a broader argument. The model appears to have learned that length and sentence complexity = analysis, missing that the argument is asserted rather than supported with evidence. The Bollywood comparison is rhetorical flourish, not genuine reasoning.

**Wrong prediction #2:**
> *"Awesome or not, halftime show is an American concept that I hate seeping into every sport I watch"*
- **True label:** reaction | **Predicted:** analysis | **Confidence:** 0.36

**Why it failed:** This is a pure emotional reaction to the halftime show, but it contains a structured-sounding claim ("American concept seeping into every sport"). The model likely picked up on the analytical framing ("American concept") rather than the emotional content. This reveals a systematic pattern: the model conflates *structured-sounding language* with *analysis*, missing that reactions can be expressed in complete sentences.

**Wrong prediction #3:**
> *"GAYLE HAD 8 WICKETS AND 600 ODD RUNS AT 50 RPI AND @ 180 SR"*
- **True label:** reaction | **Predicted:** analysis | **Confidence:** 0.43

**Why it failed:** This contains genuine statistics — exactly the signal the model associates with analysis. However, the context is pure fan excitement (all caps, no argument being made). The stats are being cited as an exclamation, not as evidence for a claim. The model has learned "stats = analysis" without learning that the *function* of the stats matters. This is the hardest boundary in the task.


---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn a distinction based on *argumentative function* — whether a comment reasons from evidence or merely asserts. What it actually learned was a proxy for that distinction: **length and surface complexity**.

The model collapsed to predicting `analysis` for everything because analysis comments in the training set were disproportionately longer and more syntactically complex than hot_takes or reactions. DistilBERT learned that longer, multi-clause sentences with specific nouns (player names, statistics, policy terms) signal `analysis` — which is often true, but misses the cases where short comments make genuine arguments and long comments make no real case.

The most revealing failure is comment #3 above — "GAYLE HAD 8 WICKETS AND 600 ODD RUNS AT 50 RPI AND @ 180 SR." This contains real statistics but is clearly a reaction. The model saw stats and predicted analysis. What it failed to capture is the *purpose* of the stats — are they evidence in an argument, or are they being shouted in excitement? That distinction requires understanding conversational context, not just surface features, and 161 training examples is nowhere near enough for DistilBERT to learn it.

To fix this, I would need: (1) more training examples — at least 500, (2) more balanced classes especially for reaction, and (3) explicit hard cases in the training set that show stats appearing in reactions and analysis appearing in short posts, to break the length/complexity proxy.

---

## Spec Reflection

**One way the spec helped:** The requirement to define edge cases and decision rules before annotating 200 examples was genuinely useful. Writing the decision rule "if a stat is used decoratively rather than as part of a logical argument → hot_take" forced a precision that made annotation much faster and more consistent. Without that rule, the BCCI bone testing comments and the Gayle stat-shouting comments would have been genuinely confusing to distinguish.

**One way implementation diverged:** The spec suggests aiming for at least 20% per label. Despite targeted collection efforts, `reaction` only reached 17.8%. Discussion-flair posts on r/IPL systematically attract analytical and opinion-based content — the flair itself filters out the pure reaction posts. In hindsight, I should have collected from match day threads (which have no flair filtering) alongside discussion posts to get a more balanced distribution. The imbalance directly contributed to the model's majority-class collapse.

---

## AI Usage

**Instance 1 — Label design and stress-testing:**
During Milestone 1, I shared my label definitions and real r/IPL comments with Claude and asked it to classify them. This surfaced edge cases I hadn't anticipated — specifically the boundary between `hot_take` and `reaction` for comments that dismiss an argument emotionally. Claude proposed the decision rule "reactions are about events, not about other people's opinions," which I adopted into my planning.md after verifying it held up across multiple examples.

**Instance 2 — Pre-labeling the dataset:**
After collecting 230 comments, I uploaded the CSV to this conversation and asked Claude to label all examples using my taxonomy definitions. Claude labeled all 230 comments with brief reasoning for borderline cases. I then reviewed every label, overriding approximately 15-20% based on my own judgment — particularly cases where Claude labeled long hot_takes as analysis due to their length and surface complexity. The final labels are mine; Claude's pre-labeling served as a first pass that I verified and corrected.

**Instance 3 — Failure pattern analysis:**
After reviewing the wrong predictions from Section 4, I shared the 16 misclassified examples with Claude and asked it to identify systematic patterns. Claude identified the "stats = analysis" false proxy and the "structured language = analysis" conflation — patterns that were hard to see reviewing cases one at a time. I verified both patterns by re-reading all 16 wrong predictions and confirmed they accounted for the majority of errors. These observations directly informed the reflection section above.

**Instance 4 — Planning.md and README.md text formating for GitHub:**
I used Claude to help me neatly format the text contents in this file and the planning.md file.
