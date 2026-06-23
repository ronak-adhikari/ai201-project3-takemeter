TakeMeter — Project Planning


## Community
Chosen community: r/IPL (Indian Premier League subreddit)
The IPL subreddit is an active, text-heavy fan community centered on the Indian Premier League cricket tournament. Discourse ranges from immediate emotional reactions to match events, to bold player opinions, to structured arguments backed by statistics and cricket history. The "Discussion" flair filters for argumentative, opinion-driven posts, making it a reliable source of varied discourse quality. These distinctions — between evidence-backed reasoning, unsupported opinion, and pure emotional reaction — are ones that regular participants in this community actively recognize and respond to differently.


## Label Taxonomy

**analysis**

Definition: The post makes a structured argument supported by stats, historical examples, rules/policies, or logical reasoning. The claim could be evaluated or challenged based on the evidence provided.

Clear examples:
A post explaining BCCI's bone testing policy, citing specific player cases (Rasikh Dar, Manjot Kalra), margin of error (±2 years), and a specific claimed date of birth — the argument is detailed enough to be fact-checked.
A comment explaining why IPL jerseys have too many sponsor logos by laying out the business case: low merch sales, short season, multiple sponsor commitments making smaller apparel deals more viable.

Uncertain example:
"too many sponsors and front sponsor not adjusting their logos to fit the jersey. currently, only rcb one fits well and csk tho eithad looks clean everywhere."
This names specific jerseys and makes an aesthetic judgment with mild observation — but it doesn't provide reasoning for why those fit well. → Labeled hot_take because the observations are asserted rather than argued.


**hot_take**

Definition: A confident opinion or claim stated without meaningful supporting evidence. The post asserts a position but doesn't back it up with stats, history, or structured reasoning.

Clear examples:
"I don't think he is faking his age!" — bold dismissal of the age fraud claim with no reasoning provided.
"The IPL jerseys colour + branding combination is amongst the worst I've seen in sports." — strong opinion, no evidence or comparison to support it.


Uncertain example:
"the way he has been performing is what matters, not the commentators' obsession with attaching his age with him."
This could be a reaction (emotional response to the discourse) or a hot_take (dismissing an argument). → Decision rule: if the post is dismissing another argument rather than responding to a specific event or moment, label it hot_take. Reactions are about events, not about other people's opinions.


**reaction**

Definition: An immediate emotional or nostalgic response to a specific event, player moment, or memory. Little to no argument — the post is expressing a feeling rather than making a case.

Clear examples:
"That Aircel logo meant domination back in the days. IPL was the tournament where 7 teams fight to get a chance to meet CSK in playoffs. Chennai was so deadly 2010-2013..." — pure nostalgia, no argument being made.
"Players?? a team won two trophies due to impact player" — one-liner emotional exclamation reacting to the topic framing.


Uncertain example:
"Well I was skeptical of him as well, but the way he has been performing is what matters" — starts with a personal feeling but pivots to a mild claim. → Labeled hot_take because the pivot makes it an assertion, not a pure reaction.



### Edge Case Decision Rules
| Situation | Rule |
|---|---|
| Post has one stat but is mostly assertive in tone | If the stat feels decorative rather than central to a logical argument → hot_take |
| Post is dismissing someone else's opinion | If no event/moment is being reacted to → hot_take, not reaction |
| Post is nostalgic but makes a comparative claim | If the claim is supported by specific examples → analysis; if it's just feeling → reaction |
| Post is in Hinglish (mixed Hindi/English) | Include only if the argument is followable in English; skip if majority non-English |

---

### Label Distribution Target

| Label | Target % | Target Count (of 200) |
|---|---|---|
| analysis | ~30% | ~60 |
| hot_take | ~40% | ~80 |
| reaction | ~30% | ~60 |

Note: hot_take is expected to be the most common label in online discourse. Will adjust targets after first 50 annotations if distribution diverges significantly.

### Data Collection Plan

**Source:** r/IPL on Reddit, filtered by the "Discussion" flair.

**Method:** PRAW (Python Reddit API Wrapper) — a Python library that lets you pull post titles, comment text, and metadata from Reddit programmatically. Will collect comments across 10–15 different discussion posts covering a variety of topics (player form, match reactions, team strategy, rule debates) to ensure topical variety in the dataset.

**Per-label targets:**
- analysis: ~60 examples
- hot_take: ~80 examples
- reaction: ~60 examples

**If a label is underrepresented after 200 examples:**
- Seek out post types likely to produce that label — e.g., post-match threads for more reactions, player debate threads for more hot_takes, stat-heavy discussion threads for more analysis
- Do not artificially duplicate examples — collect new ones instead
- Acceptable final distribution: no label below 20% of total dataset (40 examples minimum for each label)

**Annotation approach:** Manual labeling by the primary annotator (me), with edge cases resolved by referring to the decision rules table above. If a comment is genuinely unclassifiable after applying all decision rules, it will be skipped rather than forced into a label.

---

### Evaluation Metrics

**Primary metric: Overall accuracy**
Measures what percentage of test examples the model labels correctly. This is the baseline signal for whether fine-tuning worked at all.

**Why accuracy alone is not enough:**
If the model predicts hot_take for every single example, it would achieve ~40% accuracy on a balanced dataset — without learning anything useful. Accuracy doesn't reveal whether the model is actually distinguishing between labels or just exploiting class imbalance.

**Additional metrics:**

- **Per-class F1 score:** The harmonic mean of precision and recall for each label. This tells us whether the model is performing well across all three labels or just getting good at one. A model with 85% accuracy but 20% F1 on analysis has not learned to identify analysis.

- **Confusion matrix:** Shows exactly which labels the model confuses with which. For this task, the most likely confusion is hot_take vs. analysis (both involve claims) and hot_take vs. reaction (both can be short and assertive). The confusion matrix will reveal which boundary the model struggles with most.

- **Per-class recall for analysis:** In a community tool context, missing an analysis post (labeling it as hot_take) is worse than the reverse — analysis posts are the ones worth surfacing. High recall on analysis is therefore a priority metric.

---

### Definition of Success

**Minimum acceptable performance (good enough for a real tool):**
- Overall accuracy >= 70% on the test set
- F1 score >= 0.60 on all three labels individually
- Analysis recall >= 0.65 (the model correctly identifies at least 65% of analysis posts)

**Strong performance:**
- Overall accuracy >= 80%
- F1 score >= 0.70 on all three labels
- Confusion matrix shows no label being systematically collapsed into another

**Why these thresholds:**
This is a subjective classification task on informal text. Human annotators would likely disagree on 15–20% of examples themselves. A model hitting 70–75% accuracy on genuinely ambiguous text is capturing real signal. Expecting 90%+ would suggest either label leakage or labels that are too easy.

---

### AI Tool Plan

**Label stress-testing:**
Before annotating 200 examples, I will give Claude my three label definitions and edge case descriptions and ask it to generate 10 boundary posts — comments that could plausibly belong to two labels. If any generated post is unclassifiable using my current decision rules, I will tighten the relevant definition before starting annotation. This is a one-time check done before data collection.

**Annotation assistance:**
All 200 examples will be labeled manually. I will not use an LLM to pre-label examples. For genuinely ambiguous cases encountered during annotation, I will paste the comment to Claude and work through it against my decision rules — but the final label decision is always mine. These assisted decisions will be noted in the difficult examples section of the README.

**Failure analysis:**
After generating model predictions on the test set, I will compile the list of wrong predictions and share them with Claude, asking it to identify systematic patterns (e.g., "the model consistently misclassifies short hot_takes as reactions" or "analysis posts with emotional framing are mislabeled"). I will then verify any identified pattern by manually reviewing the relevant subset of errors before writing it up in my evaluation report.

---
