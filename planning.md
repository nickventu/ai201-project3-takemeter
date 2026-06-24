# Project Planning: r/soccer Comment Classifier

## 1. Community

I chose **r/soccer** (8.7M members), the main hub for soccer news, results, and discussion on Reddit. I picked it because the same triggering event — a match moment, a transfer rumor, a refereeing decision — reliably produces an enormous spread of response quality in the same comment section: rigorous tactical/statistical breakdowns sit directly next to one-line dismissals, jokes, and pure reactionary takes. That spread is exactly what makes the discourse interesting to classify: there's a real, recognized distinction in this community between a comment that earns its conclusion and one that just states it loudly, and regulars clearly reward the former (it's visible in how threads escalate from assertion → rebuttal-with-evidence → counter-evidence once someone introduces a checkable fact). The subreddit easily clears the 200-post volume bar — any single high-traffic thread (a marquee match, a transfer megathread) routinely produces hundreds of comments.

## 2. Labels

Labels were not designed from memory — they were drawn from skimming real threads (a Ronaldo post-match reaction thread, an escalating team-ranking argument, and a Barcelona/Julián Álvarez transfer-rumor thread) and patched twice after finding cases the first draft couldn't cleanly classify (see Section 3).

**Analysis**: A comment that supports its claim with specific, checkable detail that actually bears on the claim being made — not just any fact, but one that would change whether the claim is true or false if it turned out otherwise.
- *"I believe he wasn't subbed out simply because he was on a hattrick and the game was already won by halftime... I do hope in a much closer game, he'd get subbed off when he's clearly out of it."*
- *"I mean, Alvarez's contract with Atletico ends until 2030, so I don't see why they should sell him if they don't like the amount Barca offers... at some point you have to make a point of showing players you are not just a stepping stone."*

**Hot take**: A strong evaluative claim about a player, team, or manager with no supporting evidence offered — the comment is the conclusion, not an argument for it.
- *"Pep is the most overrated manager in football history."*
- *"Arent really better options than Julian? I really like him and he's good but he's not worth more than 100$ million."*

**Banter / meme**: A comment whose primary function is humor, rivalry-jabbing, or in-group reference rather than making any claim about the sport.
- *"Imagine being a Man United fan in 2026 💀"*
- *"Now, for the next 10 games, Ronaldo is free to just blast free kicks onto the unsuspecting players in the wall, who will not be bracing themselves for the impact anymore..."*

**Transfer gossip / reaction**: A comment reacting to or speculating about a transfer rumor, with no tactical or performance analysis attached.
- *"Yes, this is barely newsworthy. We all know he wants this move, ofc he agreed to a deal."*
- *"Ah yes, the classic 'contract agreement' non-story without a transfer fee agreed between the two clubs. Now it's really summer."*

## 3. Hard edge cases

Three real ambiguity patterns showed up across the pre-annotation skim (~40 real comments across 3 threads), each of which forced a definition patch:

**Edge case 1 — True-but-non-probative detail (Analysis vs. Hot take).** A comment can cite a real, specific, checkable fact that still doesn't actually support its conclusion.
> *"I mean a 38 year old literally played the full 90 minutes just yesterday and all I heard is how much of a goat he is… so you're completely right"* — the fact (38-year-old, full 90, yesterday) is true and specific, but one anecdote can't establish the bias claim it's being used to prop up.

**Handling rule:** Before labeling something Analysis, apply the probative test: *if this detail were false or different, would the comment's conclusion have to change?* If no, it's a Hot take wearing a fact as a costume. This is why the Analysis definition explicitly requires the detail "actually bear on the claim," not just be present.

**Edge case 2 — Casual/joking tone masking real content (Banter vs. Analysis).** Hedged, lighthearted delivery is not a reliable signal of Banter.
> *"fair play to him still going for sprints and trying to win balls in minute 90 basically... if he is still putting in a shift like today at the end of the match you can hardly blame the coach"* — reads as casual/positive in tone but references a specific observable detail (still sprinting, not subbed) to support a real, if soft, claim.

**Handling rule:** Check for an actual claim-plus-evidence structure underneath the tone, regardless of how jokey or hedged the delivery is. Banter is reserved for comments with no real claim at all, not just comments with a light tone.

**Edge case 3 — Mixed-mode comments (one comment, two claim types).** A single comment can open with a probative, checkable claim and close with an unsupported evaluative one.
> *"It is interesting that he was irrelevant when at City... I mean its a good player, but man! This sounds like he is the best striker in the world or something. I guess this is the level of football after 2010s passed."* — opens with a true, relevant fact (was a backup at City) but the comment's actual destination is an unsupported generational complaint.

**Handling rule:** Label by the comment's real conclusion, not its best supporting fragment. When an unsupported evaluative claim is doing the actual work — i.e., the comment would have been written the same way without the factual aside — default to Hot take. The factual detail only earns Analysis when it's load-bearing for the conclusion being drawn.

**General annotation rule:** Labels are assigned **per-comment, independently of thread/reply context**. This trades some accuracy (a reply that's only legible as evidence *in response to* a specific prior claim may look like an unsupported Hot take in isolation) for speed and reproducibility — the same comment, read cold by a different annotator, should get the same label.

A **parking-lot issue** noted but not yet acted on: a small number of comments (a tangential anecdote about a different manager/league; a tangential claim about a different player's squad history) are specific and true but only loosely relevant to the claim at hand — neither clean Analysis nor any other existing label. Seen twice so far; if a third clear instance appears during full annotation, this becomes a candidate fifth label ("tangent") rather than being forced into Analysis or Hot take.

## 4. Data collection plan

- **Source:** r/soccer, pulling comments from a mix of thread types to avoid skewing toward one register: post-match reaction threads, transfer-rumor/megathreads, and tactical/match-discussion threads. Mixing thread types matters because a single thread type (e.g., only reaction threads) underrepresents Transfer gossip and skews Analysis toward a narrower kind of claim.
- **Target:** 200 comments minimum, aiming for roughly 50 per label as a starting balance, acknowledging real-world distribution won't be even (Hot take and Banter are likely to be naturally overrepresented relative to Analysis in most threads).
- **If a label is underrepresented after 200:** Rather than padding with marginal examples just to hit a quota, I'll deliberately mine thread types known to concentrate that label — e.g., if Analysis is short, pull from threads following a contentious refereeing decision or a tactical post-match breakdown, where the community's more rigorous posters tend to congregate, rather than reaction threads. If still short after a second targeted pass, I'll report the natural imbalance honestly in the writeup rather than force balance artificially, since an imbalanced label distribution is itself a real property of the community worth noting.

## 5. Evaluation metrics

Accuracy alone is insufficient because the labels are not balanced and not equally costly to confuse. I'll report:

- **Per-class precision and recall**, not just overall accuracy — this surfaces whether the model is systematically biased toward the majority class (likely Hot take/Banter) at the expense of rarer but more valuable labels (Analysis, which is arguably the most useful label for a real community tool to surface reliably).
- **Macro-averaged F1**, which weights all four classes equally regardless of how many examples each has — this is the right summary number *because* the classes are imbalanced; a model that nails Hot take and Banter but is mediocre at Analysis would still look good on accuracy alone but bad on macro-F1, and Analysis is the label most worth getting right.
- **A confusion matrix**, specifically to check whether errors cluster along the same seams identified in Section 3 (Analysis↔Hot take, Banter↔Analysis) rather than being randomly distributed — if the model's mistakes mirror my own human ambiguity, that's a meaningfully different (and more forgivable) failure mode than random noise.

## 6. Definition of success

**Genuinely useful threshold:** Macro-F1 ≥ 0.75, with no single class's recall below 0.65. This would mean the classifier reliably distinguishes the four categories even for the harder, less frequent ones — good enough to, say, power a filter that lets a user surface "Analysis"-tagged comments in a thread without burying them in false positives from confidently-worded Hot takes.

**"Good enough" for deployment in a real community tool:** Macro-F1 ≥ 0.65, AND the confusion matrix shows that the majority of remaining errors fall specifically on the three documented edge-case seams (Section 3) rather than being scattered/unexplainable. Errors on genuinely hard boundary cases (the same ones a human annotator would hesitate on) are acceptable for deployment; errors on cases a regular would consider obvious are not, regardless of the aggregate score.

**Reviewing for specificity:** These thresholds are objectively checkable at the end — macro-F1 and per-class recall are computed numbers, not impressions, and "errors cluster on documented seams" can be checked by manually reading the wrong predictions and confirming they overlap with the Section 3 cases (this is also the planned Failure Analysis step below). The one soft judgment call is "no single class's recall below 0.65" — I'm treating this as a hard cutoff specifically so an embarrassing single-class failure (e.g., the model never correctly identifies Transfer gossip) can't hide behind a decent macro average.

## 7. AI Tool Plan

**Label stress-testing.** Before annotating the full 200, I gave Claude my four label definitions plus the three documented edge cases and asked it to generate boundary-straddling example comments. This was done iteratively in practice: rather than one synthetic-generation pass, real comments pulled from live threads (Ronaldo reaction thread, ranking-argument thread, Álvarez transfer thread) served the same stress-testing function and produced three genuine definition patches (Sections 3.1–3.3) before any of the 200 were annotated. If, during full annotation, I hit a new ambiguity type not covered by the existing three patches, I'll go back to Claude with the new case before continuing, rather than pushing through with an unclear rule.

**Annotation assistance.** I will use an LLM (Claude) to pre-label a batch of comments before reviewing each one myself — not to replace my judgment, but to speed up the mechanical part of applying an already-fixed rule set. Tracking plan: pre-labeled examples will get a `pre_labeled_by_llm` boolean column in the annotation spreadsheet/CSV (alongside `final_label`, which is always my own call after review), so the AI usage section of the final writeup can report exactly what fraction of the 200 started from an LLM suggestion versus a cold human read, and so any systematic LLM bias (e.g., over-labeling Analysis) can be checked against that column later.

**Failure analysis.** Once the classifier is trained and evaluated, I'll compile the list of misclassified examples (predicted label vs. true label vs. comment text) and give that list to Claude to look for patterns — specifically, whether errors concentrate on the three known edge-case seams (Section 3), on a specific thread type (e.g., the model struggles more with transfer threads than match threads), or on comment length (e.g., short comments lacking enough signal either way). I will verify any pattern Claude proposes by manually pulling and re-reading a sample of the flagged errors myself before stating the pattern as a finding — an AI-identified pattern in the writeup is a hypothesis to check against the actual text, not a conclusion to report as-is.
