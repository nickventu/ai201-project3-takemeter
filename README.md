# r/soccer Comment Classifier

## Community

I chose **r/soccer** (8.7M members), the main hub for soccer news, results, and discussion on Reddit. I picked it because the same triggering event — a match moment, a transfer rumor, a refereeing decision — reliably produces an enormous spread of response quality in the same comment section: rigorous tactical/statistical breakdowns sit directly next to one-line dismissals, jokes, and pure reactionary takes. That spread is exactly what makes the discourse interesting to classify: there's a real, recognized distinction in this community between a comment that earns its conclusion and one that just states it loudly, and regulars clearly reward the former (it's visible in how threads escalate from assertion → rebuttal-with-evidence → counter-evidence once someone introduces a checkable fact). The subreddit easily clears the 200-post volume bar — any single high-traffic thread (a marquee match, a transfer megathread) routinely produces hundreds of comments.

## Label Taxonomy

**Analysis**
A comment that supports its claim with specific, checkable detail that actually bears on the claim being made — not just any fact, but one that would change whether the claim is true or false if it turned out otherwise.

- *"I believe he wasn't subbed out simply because he was on a hattrick and the game was already won by halftime... I do hope in a much closer game, he'd get subbed off when he's clearly out of it."*
- *"I mean, Alvarez's contract with Atletico ends until 2030, so I don't see why they should sell him if they don't like the amount Barca offers... at some point you have to make a point of showing players you are not just a stepping stone."*

**Hot take**
A strong evaluative claim about a player, team, or manager with no supporting evidence offered — the comment is the conclusion, not an argument for it.

- *"Pep is the most overrated manager in football history."*
- *"Arent really better options than Julian? I really like him and he's good but he's not worth more than 100$ million."*

**Banter**
A comment whose primary function is humor, rivalry-jabbing, or in-group reference rather than making any claim about the sport.

- *"Imagine being a Man United fan in 2026 💀"*
- *"Now, for the next 10 games, Ronaldo is free to just blast free kicks onto the unsuspecting players in the wall, who will not be bracing themselves for the impact anymore..."*

**Transfer gossip**
A comment reacting to or speculating about a transfer rumor, with no tactical or performance analysis attached.

- *"Yes, this is barely newsworthy. We all know he wants this move, ofc he agreed to a deal."*
- *"Ah yes, the classic 'contract agreement' non-story without a transfer fee agreed between the two clubs. Now it's really summer."*

---

## Data Collection

**Source:** r/soccer

**Labeling process:** LLM pre-label followed by human review

**Final label distribution:**

| Label | Count | % |
|---|---|---|
| Hot take | 48 | 34% |
| Analysis | 35 | 25% |
| Banter | 29 | 21% |
| Transfer gossip | 28 | 20% |

**Difficult-to-label examples:**

1. *"I mean a 38 year old literally played the full 90 minutes just yesterday and all I heard is how much of a goat he is… so you're completely right"*
   - **Decision:** [Hot take]
   - **Reasoning:** Edge case 1 — True-but-non-probative detail (Analysis vs. Hot take). A comment can cite a real, specific, checkable fact that still doesn't actually support its conclusion.
     - The fact (38-year-old, full 90, yesterday) is true and specific, but one anecdote can't establish the bias claim it's being used to prop up.
     -  **Handling rule:** Before labeling something Analysis, apply the probative test: *if this detail were false or different, would the comment's conclusion have to change?* If no, it's a Hot take wearing a fact as a costume. This is why the Analysis definition explicitly requires the detail "actually bear on the claim," not just be present.

2. *"fair play to him still going for sprints and trying to win balls in minute 90 basically... if he is still putting in a shift like today at the end of the match you can hardly blame the coach"*
   - **Decision:** [Analysis]
   - **Reasoning:** Edge case 2 — Casual/joking tone masking real content (Banter vs. Analysis). Hedged, lighthearted delivery is not a reliable signal of Banter.
       - reads as casual/positive in tone but references a specific observable detail (still sprinting, not subbed) to support a real, if soft, claim.
       - **Handling rule:** Check for an actual claim-plus-evidence structure underneath the tone, regardless of how jokey or hedged the delivery is. Banter is reserved for comments with no real claim at all, not just comments with a light tone.

3. *"It is interesting that he was irrelevant when at City... I mean its a good player, but man! This sounds like he is the best striker in the world or something. I guess this is the level of football after 2010s passed."*
   - **Decision:** [Hot take]
   - **Reasoning:** Edge case 3 — Mixed-mode comments (one comment, two claim types). A single comment can open with a probative, checkable claim and close with an unsupported evaluative one.
       - opens with a true, relevant fact (was a backup at City) but the comment's actual destination is an unsupported generational complaint.
       - **Handling rule:** Label by the comment's real conclusion, not its best supporting fragment. When an unsupported evaluative claim is doing the actual work — i.e., the comment would have been written the same way without the factual aside — default to Hot take. The factual detail only earns Analysis when it's load-bearing for the conclusion being drawn.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` via HuggingFace `AutoModelForSequenceClassification`

**Training setup:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Train/val/test split: 70/15/15, stratified by label

**Hyperparameter decisions:** Experimented with 5 epochs and class weighting but found the results inconsistent and returned to defaults for the final reported run.

---

## Baseline

**Model:** Groq hosted llama-3.3-70b-versatile

**Prompt used:**

...
You are classifying comments from r/soccer, a soccer discussion community on Reddit.
Assign each comment to exactly one of the following categories.

Analysis: A comment that supports its claim with specific, checkable detail that actually bears on the claim being made — not just any fact, but one that would change whether the claim is true or false if it turned out otherwise.
Example: "I mean, Alvarez's contract with Atletico ends until 2030, so I don't see why they should sell him if they don't like the amount Barca offers... at some point you have to make a point of showing players you are not just a stepping stone."

Hot take: A strong evaluative claim about a player, team, or manager with no supporting evidence offered — the comment is the conclusion, not an argument for it.
Example: "Arent really better options than Julian? I really like him and he's good but he's not worth more than 100$ million."

Banter: A comment whose primary function is humor, rivalry-jabbing, or in-group reference rather than making any claim about the sport.
Example: "Now, for the next 10 games, Ronaldo is free to just blast free kicks onto the unsuspecting players in the wall, who will not be bracing themselves for the impact anymore..."

Transfer gossip: A comment reacting to or speculating about a transfer rumor, with no tactical or performance analysis attached.
Example: "Ah yes, the classic 'contract agreement' non-story without a transfer fee agreed between the two clubs. Now it's really summer."

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
Analysis
Hot take
Banter
Transfer gossip
...

**How results were collected:** Each of the 30 test comments was passed individually to Llama 3.3 70B via the Groq API using the system prompt above. The model's response was stripped, lowercased, and matched against the four valid label names (checked longest-first to avoid substring collisions). Any response that didn't match a known label was marked unparseable and excluded. All 30 responses parsed successfully. Predicted labels were then compared against ground truth to compute accuracy and per-class metrics.

---

## Evaluation Report

### Overall accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq) | 0.633 |
| Fine-tuned DistilBERT | 0.367 |

### Per-class metrics — Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Analysis | 0.86 | 0.75 | 0.80 | 8 |
| Hot take | 0.62 | 0.50 | 0.56 | 10 |
| Banter | 0.50 | 0.83 | 0.62 | 6 |
| Transfer gossip | 0.60 | 0.50 | 0.55 | 6 |
| **Macro avg** | 0.65 | 0.65 | 0.63 | 30 |

### Per-class metrics — Fine-tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Analysis | 0.00 | 0.00 | 0.00 | 8 |
| Hot take | 0.36 | 1.00 | 0.53 | 10 |
| Banter | 0.00 | 0.00 | 0.00 | 6 |
| Transfer gossip | 0.00 | 0.00 | 0.00 | 6 |
| **Macro avg** | 0.25 | 0.36 | 0.28 | 30 |

### Confusion matrix (fine-tuned model)

|  | Pred: Analysis | Pred: Hot take | Pred: Banter | Pred: Transfer gossip |
|---|---|---|---|---|
| **True: Analysis** | 0 | 8 | 0 | 0 |
| **True: Hot take** | 0 | 10 | 0 | 0 |
| **True: Banter** | 0 | 6 | 0 | 0 |
| **True: Transfer gossip** | 0 | 5 | 0 | 1 |

### Wrong predictions analyzed

**1.** *"England vs Mexico in Mexico City is about to be absolute cinema."* — True: [Banter], Predicted: [Hot take]
The comment contains no humor marker, rivalry jab, or in-group reference that a surface-level model could latch onto. "Cinema" is informal hyperbole, which likely pattern-matches to the assertive, evaluative language the model associated with Hot take. With only 29 Banter training examples, the model had insufficient signal to learn that Banter can be expressed through anticipatory excitement rather than an explicit joke or meme format.

**2.** *"Because it was only a 1-0 game. If Austria scores to tie it 1-1 and Messi is off the field they would be in trouble. He was subbed off when they were up 3-0."* — True: [Analysis], Predicted: [Hot take]
This is a reply-dependent comment and its probative detail (the scoreline, the substitution timing) only reads as evidence-backed Analysis if you know what claim it's responding to. In isolation it looks like a string of assertions, which the model likely read as Hot take. This is a known limitation of the per-comment, context-free labeling approach documented in planning.md.

**3.** *"Its not a proper Villa transfer window without a loan signing of a Manchester United outcast, is it? 🤣 I do hope we see the best of Sancho as there is a great player in there somewhere!! Harvey Elliot"* — True: [Transfer gossip], Predicted: [Analysis]
This comment mixes modes by opening with a banter-style joke about Villa's transfer habits, then shifting to genuine speculation about Sancho. The model predicted Analysis, likely because the second half names specific players and expresses a reasoned hope, which superficially resembles evidenced claims. This sits on the Banter/Analysis seam from edge case 2, but with a transfer context layered on top (a combination the model had almost no training examples to learn from.)

### Sample classifications

| Comment (truncated) | True label | Predicted label | Confidence |
|---|---|---|---|
| *"Will Palace lose out long term on a fee for Guehi in future? Didn't want to sell him last season for £65 mil and just pulled the plug on a move to Liv"* | Analysis | Analysis | 0.27 |
| *"Scored two goals against the worst team at the World Cup, IM BACK!"* | Hot take | Hot take | 0.28 |
| *"Someone give pardew a hug"* | Banter | Hot take | 0.26 |
| *"Apparently Solanke's ankle is worst than was thought. Might need surgery. Anyone got a Nico Jackson to go on loan?"* | Transfer gossip | Hot take | 0.28 |

**Correct prediction explained:**
- *"Will Palace lose out long term on a fee for Guehi in future? Didn't want to sell him last season for £65 mil and just pulled the plug on a move to Liv"*
- The model correctly identified this as Analysis with 0.27 confidence. The comment cites a specific transfer fee (£65M) and a named player (Guehi) to support a forward-looking claim about sell-on value which is exactly the probative detail the Analysis definition requires. The low confidence (0.27) reflects how thin the margin is between Analysis and Hot take for the model, consistent with its overall difficulty separating these two classes.

---

## Reflection

The model was intended to learn four linguistically distinct discourse modes: whether a comment earns its conclusion with checkable evidence (Analysis), asserts without evidence (Hot take), functions as humor or rivalry (Banter), or reacts to transfer news without analysis (Transfer gossip). In practice, the model learned a single coarse distinction: comments that pattern-match to assertive, evaluative language get predicted as Hot take, and everything else collapses into that same bucket. The intended boundaries — particularly Analysis vs. Hot take (does the evidence actually bear on the claim?) and Banter vs. Hot take (is the primary function humor rather than assertion?) — require understanding pragmatic intent and discourse structure, not surface lexical patterns. With 140 training examples and no access to the label definitions at inference time, DistilBERT had no mechanism to learn those distinctions and instead latched onto the path of least resistance: Hot take is the largest class and the most lexically similar to the others, so predicting it minimizes training loss without learning anything meaningful about the actual boundaries.


---

## Spec Reflection

**One way the spec helped:** The probative test in edge case 1 forced a concrete annotation rule before full labeling began, which meant the Analysis/Hot take boundary was applied consistently across all 200 examples rather than drifting mid-collection. Without that rule, comments like the 38-year-old anecdote would likely have been inconsistently labeled.

**One way implementation diverged from the spec and why:** Initially planned a pre_labeled_by_llm tracking column but this was not implemented; instead all annotation assistance is disclosed in aggregate in the AI usage section

---

## AI Usage

**Label stress-testing.** Before and throughout annotating the full 200, I gave Claude my label definitions plus the running set of edge-case rules and asked it to flag boundary-straddling comments as they came up in real batches. This was done iteratively in practice rather than as one synthetic-generation pass: real comments pulled from live threads served the stress-testing function directly, and produced four numbered definition patches (Section 3) plus several recurring sub-patterns (tone traps, format mimicry, the intent-vs-content tiebreaker) over the course of annotation — not all identified up front, but added as soon as a new ambiguity type appeared, before continuing to label with an unclear rule.

**Annotation assistance.** All 200 examples went through an LLM pre-labeling and human review workflow: I pasted batches of real comments (collected directly from r/soccer threads) to Claude along with the current label definitions and edge-case rules, Claude assigned a label and flagged any genuinely ambiguous cases, and I reviewed and either confirmed or corrected every label before it was added to the dataset. No label was accepted on a skim — multiple labels were corrected on review during this process (e.g., a "good defense" comment was initially mislabeled Analysis and reclassified to Hot take after patch 4 was introduced), which is itself evidence the review step was substantive rather than rubber-stamping. Since this process was applied uniformly across the entire dataset, no per-row tracking column was added to the CSV; this paragraph serves as the disclosure instead.
