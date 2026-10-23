# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter is a fine-tuned text classifier designed to categorize public posts from the r/ClaudeAI Reddit community. After browsing an initial set of recent posts and manually selecting roughly 19–20 examples, I identified three recurring types of discussions: people requesting assistance with Claude, users showcasing projects or tools they have built, and community members sharing opinions or feedback about Claude and Anthropic. These observations suggest three candidate categories: help requests, project showcases, and opinion or feedback discussions. The goal is to train a model that can distinguish these types of posts consistently using clearly defined labeling rules.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `help_request`

**Definition:** A post whose primary purpose is to request practical assistance, instructions, or an explanation for a specific problem or question involving Claude, its features, or its usage.

**Example 1:**

> **Post:** "How do you organize your Claude Projects?"
>
> **Source:** https://www.reddit.com/r/ClaudeAI/comments/1x6256u/how_do_you_organize_your_claude_projects/
>
> **Why this label fits:** The author asks whether to organize Claude Projects by repository or separate activities such as debugging, documentation, and planning. The primary purpose is to receive practical advice.

>

**Example 2:**

> **Post:** "So how does the hourly/weekly session limit work?"
>
> **Why this label fits:** The author is asking for clarification about how Claude's usage allowances work. The post seeks an explanation of a specific feature rather than sharing a project or expressing an opinion.

>

### `project_showcase`

**Definition:** A post whose primary purpose is to present a project, tool, application, automation, or creative work the author has built using Claude, including descriptions of its features, results, or implementation.

**Example 1:**

> **Post:** "Built a Claude Code plugin to cut tokens on big repos. The first version made it worse, so I measured everything"
>
> **Why this label fits:** The author presents a Claude Code plugin they developed to reduce token usage when working with large repositories. They describe their implementation, testing process, and measured results. The post's primary purpose is to showcase a tool they built.

>

**Example 2:**

> **Post:** "My Philips Hue lights turn blue while Claude Code works and flash green when it's done (open source)"
>
> **Why this label fits:** The author showcases an automation they built that connects Claude Code activity to Philips Hue smart lights. The post presents a completed integration and explains what it does, rather than asking for help or primarily expressing an opinion.

>

### `opinion_feedback`

**Definition:** A post whose primary purpose is to express an evaluation, complaint, praise, or opinion about Claude, Anthropic, its products, or the experience of using them, rather than requesting specific assistance or presenting a project the author built.

**Example 1:**

> **Post:** "Can Anthropic stop removing useful features from the Claude Code desktop UI?"
>
> **Source:** https://www.reddit.com/r/ClaudeAI/comments/1x61z0o/can_anthropic_stop_removing_useful_features_from/
>
> **Why this label fits:** The author criticizes changes to the Claude Code desktop interface, particularly the removal or relocation of useful features. The post's primary purpose is to express dissatisfaction and provide product feedback rather than request troubleshooting assistance.

**Example 2:**

> **Post:** "Opus 5.5 Reasoning is Unbelievable."
>
> **Why this label fits:** The author expresses strong praise for Claude Opus 5.5 after describing a successful technical troubleshooting experience. The post primarily evaluates the model's reasoning capabilities rather than requesting help or showcasing a project.

### The hardest boundary

**Which two labels:**

`help_request` and `opinion_feedback`

**Borderline example:** "So how does the hourly/weekly session limit work?" This post concerns a topic that frequently generates complaints about Claude, but its primary purpose is to obtain an explanation of the usage limits rather than evaluate them.

**The decision rule I used every time:**
If a post primarily asks for a specific explanation, instruction, solution, or practical recommendation about Claude, I label it `help_request`, even if it includes frustration or criticism. If a post primarily evaluates, praises, or criticizes Claude or Anthropic without seeking a specific answer or solution, I label it `opinion_feedback`, even if it contains a rhetorical question. When both appear, I look for the author's main requested outcome: receiving actionable assistance means `help_request`, while expressing a judgment or advocating a product change means `opinion_feedback`.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:**

All 200 posts are public posts from r/ClaudeAI, published in October 2026. I chose this community because I was already a member before the assignment. Each record is the post's full title and written body; author names, flair, images, ads and comments are not included.

- **My own selections.** I browsed Reddit manually, read posts, chose examples and took screenshots. I believe I was browsing the Hot feed, but I have not confirmed the sort order.
- **Larger batches.** After roughly 19–20 manually selected examples, I showed my selections to a Manus AI agent and asked it to collect batches of about 45 similar posts that met my criteria. Manus supplied those batches as PDFs.
- **Review.** I uploaded the PDFs to ChatGPT, where I reviewed the proposed classifications, approved decisions and discussed exceptions. ChatGPT also helped me write prompts for Cursor.
- **Extraction and integration.** Cursor extracted the reviewed posts from the PDFs, checked them and added them to CSV files. It did not collect the Manus batches. The final 11 AI-assisted posts were retrieved by Cursor from Reddit's public RSS feeds, because the PDF batches did not supply enough suitable posts.
- **Cold-labelling posts.** The 18 posts I labelled independently later in the project were collected separately from Reddit's public new-posts feed.

I excluded non-English posts, posts about AI in general rather than Claude, link-only news posts with no body, and duplicates, which were checked by Reddit post ID, title and full text. The PDFs had dropped dollar signs and spaces in some price amounts (for example "158to 0" instead of "$158 to $0"). I only corrected those passages with wording verified against the original Reddit post. Where I could not verify a passage, I excluded the post.

**How I labelled them:**

- **20 records are marked `cold`.** 18 of them are posts I labelled independently later in the project. They were collected from the new-posts feed in posting order and filtered only for objective reasons: already in the dataset, previously reviewed, or no written body. They were given to me as a blank CSV with no suggested labels, and I chose every label myself. I set aside three posts I could not confidently place in one label; they are not in `labels.csv`. The other two `cold` records (records 1 and 2) were among my first examples. They are marked `cold`, but I cannot remember with certainty whether AI discussion came before I chose their labels, so I do not claim all 20 as verified independent labels.
- **To keep the dataset at 200**, the 18 new cold records replaced 18 AI-assisted records. These were chosen with `random.Random(42).sample` from the 198 AI-assisted rows, without regard to label or content. The removed records are archived in `labels_removed_for_cold_integration.csv`.
- **180 records are AI-assisted.** An AI proposed each label and I personally reviewed and approved it, changing or excluding posts where I disagreed. Their notes read `AI-assisted; reviewed`, with a short reason added for borderline cases. I did not change labels to balance the dataset, and I did not add help requests just to reach 200.

The 20 cold records are 15 `project_showcase`, 4 `help_request` and 1 `opinion_feedback`. They were taken in posting order rather than chosen by label.

**Counts per label:**

| Label              | Count | Share |
| ------------------ | ----- | ----- |
| `help_request`     | 90    | 45%   |
| `project_showcase` | 58    | 29%   |
| `opinion_feedback` | 52    | 26%   |
| **Total**          | 200   | 100%  |

**Labelling consistency check (criterion 5):**

I labelled the 20 `cold` records a second time without looking at my first decisions. They were shuffled with seed 42 into a blank worksheet with no labels or notes. The two passes agreed on **18 of 20 (90%)**, above my 85% target. I set one post aside in the second pass, and I count it as a mismatch because the dataset has a label for it.

| Record | First pass (in dataset) | Second pass |
| ------ | ----------------------- | ----------- |
| 198, "I gave the Clawd Mochi desk toy a new brain…" | `project_showcase` | `help_request` |
| 1, "I let Claude hire its first employee. It chose a rock." | `project_showcase` | set aside |

This result should not be overstated:
- Criterion 5 describes a randomly selected sample of 20 posts, but I used the 20 `cold` records, not a random sample of the whole dataset.
- Some of the second-pass decisions were made the same night as the first, so I may have remembered my earlier choices.
- Three quarters of the sample is `project_showcase`, so it says little about my hardest boundary, `help_request` versus `opinion_feedback`.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**

> _The post:_ "I used Claude to design a stock investing strategy and then backtested it using a 2-year holdout. Here is what it got right and where it was biased." The author builds and tests an investing strategy with Claude and evaluates what Claude did well and where it showed bias.
>
> _Could have been:_ `project_showcase` or `opinion_feedback`. The AI suggested `project_showcase`.
>
> _I chose, because:_ `opinion_feedback`. The post's primary purpose is evaluating Claude's performance, including where it succeeded and where it showed bias, rather than showcasing the investing strategy.

**2.**

> _The post:_ "Skills list constantly repeating throughout context and triggering compaction." The author asks whether anyone else is seeing it, describes the behaviour, calls it a bug, and says it is "just chewing through context and triggering unnecessary automatic compactions."
>
> _Could have been:_ `help_request` or `opinion_feedback`
>
> _I chose, because:_ `opinion_feedback`. It is primarily a complaint about frustrating Claude behaviour, not a request for troubleshooting instructions.

**3.**

> _The post:_ "I gave the Clawd Mochi desk toy a new brain: it reacts to Claude Code, shows my usage limits, music, calendar and more." The author rewrote a desk toy's firmware with Claude Code, lists its features and links to the source code. This is one of my `cold` labels.
>
> _Could have been:_ `project_showcase` or `help_request`
>
> _I chose, because:_ `project_showcase`. The author primarily demonstrates what they built, describes its features and shares the source code rather than asking for help. In my second labelling pass I chose `help_request` for this post. I have kept that disagreement in my consistency result rather than hiding it.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`, fine-tuned for three-class classification of r/ClaudeAI posts.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, maximum length 128 tokens, seed 42. I trained on an Apple Silicon GPU (`mps`) with PyTorch 2.14.1.

**Anything I changed from the defaults, and why:** I replaced the starter notebook's three placeholder labels with my own: `help_request`, `project_showcase` and `opinion_feedback`. The notebook cannot read my `labels.csv` without that change. I kept the default model and all default hyperparameters.

**Split sizes:** The notebook split my 200 labelled posts itself (stratified, seed 42) into 139 training, 31 validation and 30 test posts.

| Label              | Train | Val | Test |
| ------------------ | ----- | --- | ---- |
| `help_request`     | 63    | 14  | 13   |
| `project_showcase` | 40    | 9   | 9    |
| `opinion_feedback` | 36    | 8   | 8    |

No label has fewer than 8 test posts, but 8 or 9 is still small. One more or one fewer correct post changes that label's F1 a lot.

**Results of this run (seed 42, 30 test posts):**

| Measure | Result |
| ------- | ------ |
| Overall accuracy | 0.4333 (13 / 30) |
| Macro F1 | 0.2016 |
| F1 — `help_request` | 0.605 |
| F1 — `project_showcase` | 0.000 |
| F1 — `opinion_feedback` | 0.000 |
| Accuracy on the most-confident third | 0.900 (10 posts) |

The model predicted `help_request` for all 30 test posts. Its 13 correct answers are exactly the 13 real `help_request` posts; all 9 `project_showcase` and all 8 `opinion_feedback` posts were called `help_request`. The classifier collapsed to my largest label and did not learn to distinguish the other two. Its confidence barely moved from an even guess: 0.437 on average when right and 0.408 when wrong, against 0.333 for a random three-way guess.

**Against my criteria (this run only):**

| # | Criterion | Target | This run | Status |
| - | --------- | ------ | -------- | ------ |
| 1 | Overall accuracy | ≥ 0.65 | 0.433 | Missed |
| 2 | Per-label F1 | `help_request` ≥ 0.55, `project_showcase` ≥ 0.65, `opinion_feedback` ≥ 0.55 | 0.605 / 0.000 / 0.000 | Missed (only `help_request` reached its target) |
| 3 | Dataset balance | No label above 50% | Largest is `help_request` at 45% | Met (a property of the data, not the model) |
| 4 | Most-confident-third accuracy | ≥ 0.70 | 0.900 | Met numerically, but not meaningful: every prediction was `help_request`, so the 9 of 10 "confident" correct answers are just `help_request` posts |
| 5 | Labelling consistency | ≥ 85% | 90% (18 / 20) | Met (measured on my labels in Milestone 4, not on the model) |

This was my first training run, with one fixed seed (42). It is not the three-seed robustness test from Unit 6, so these are not final verdicts. Unit 6 requires each target to hold across seeds 42, 7 and 2024.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

I used four AI tools, each for a different part of the work:
- **Manus** collected batches of about 45 posts similar to my own examples and supplied them as PDFs.
- **ChatGPT** reviewed the PDFs with me, proposed labels, discussed borderline cases and helped me write prompts for Cursor.
- **Cursor** extracted posts from the PDFs, checked text and duplicates, entered records into CSV files and validated them. For some PDF batches, and for the final batch of candidates it retrieved from Reddit's RSS feeds, Cursor also proposed labels and prepared review tables. Cursor did not collect the Manus PDF batches.
- **I** reviewed and approved every AI-proposed label, whichever tool suggested it.

AI helped with collecting and preparing data, but it did not choose any of the 18 later independent labels.

**Moment 1**

- _What I asked for:_ I asked Cursor to extract every post from a batch PDF with its full title and body, check for duplicates, and flag any damaged text instead of guessing.
- _What came back:_ Cursor found that the PDF had rendered price amounts as math, which dropped dollar signs and spaces. It flagged the affected posts and stopped before importing them.
- _What I changed:_ I checked a flagged post against a screenshot of the original Reddit post and supplied the exact sentence myself: "I run into my limit a lot with Pro surprisingly, but for my personal use, it doesn't make sense to go from $20/month to $100/month. 😵" Only that sentence was corrected.

**Moment 2**

- _What I asked for:_ A proposed label and a one-line justification for each post in a batch, with borderline cases flagged.
- _What came back:_ A review table for the batch, prepared by Cursor. It labelled the stock-investing backtest post `project_showcase` and flagged it as a possible evaluation of Claude.
- _What I changed:_ I changed it to `opinion_feedback`, because the post mainly evaluates Claude's performance (hard case 1). I also excluded posts that did not fit my taxonomy, such as non-English posts and general AI discussion. Later, when I was short of showcase and opinion posts, I picked 11 of 15 suggested candidates and dropped the borderline ones rather than relabelling posts to balance the counts.

**Pre-labelling disclosure:**

180 of my 200 labels were pre-labelled by AI and then individually reviewed by me, and changed where I disagreed. 20 records are marked `cold`. 18 of those I labelled myself with no AI suggestion: the AI only collected the posts and entered my decisions into the CSV. For the other two, records 1 and 2, I cannot remember with certainty whether AI discussion came before my decision. The `note` column shows which records are which.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure                 | Baseline | Trained | Difference |
| ----------------------- | -------- | ------- | ---------- |
| Overall accuracy        | 0.567    | 0.433   | -0.133     |
| Macro F1                | 0.449    | 0.202   | -0.248     |
| F1 — `help_request`     | 0.727    | 0.605   | -0.123     |
| F1 — `project_showcase` | 0.621    | 0.000   | -0.621     |
| F1 — `opinion_feedback` | 0.000    | 0.000   | +0.000     |

The baseline is `facebook/bart-large-mnli`, used zero-shot with my label definitions from `label_definitions.txt` as its only instructions. Both models were scored on the same 30 held-out posts in `test_split.csv`. **Difference = Trained − Baseline**, so a negative number means the baseline did better. On every measure, the baseline was better or tied.

**What I predicted before I looked:**

<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

I predict that the zero-shot baseline model will perform slightly better than my trained DistilBERT model, which achieved 43.3% accuracy.

My trained model kept predicting help_request for every test post. It correctly identified the 13 help requests but failed to recognize the other two categories. I think the zero-shot model will be less likely to repeat that pattern and will make predictions across all three labels instead of leaning toward just one.

I expect a small improvement, not necessarily a dramatic one.

**What the gap actually means:**

<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

My main prediction was correct: the zero-shot baseline beat my trained DistilBERT model. It scored 0.567 accuracy (17 of 30) against my model's 0.433 (13 of 30), about 13.3 percentage points higher. Its macro F1 was also higher, 0.449 against 0.202. On these posts, my fine-tuning made the model worse than using no training at all.

The biggest gap is on `project_showcase`. The baseline found all 9 project showcases (F1 0.621), while my trained model found none (F1 0.000). The baseline was also better on `help_request` (F1 0.727 against 0.605).

Neither model learned `opinion_feedback`. Both scored F1 = 0 on it. The baseline labelled 7 of the 8 opinion posts as `project_showcase` and the other 1 as `help_request`.

The second part of my prediction only partly held. The baseline did use more than one label, but it did not spread its predictions evenly. It leaned heavily toward `project_showcase`, which it chose for 20 of the 30 posts, and used `opinion_feedback` only once (and that one was wrong). So both models favoured one label; they just favoured different ones.

The baseline's 0.567 accuracy is still below my 0.65 target, so neither model met my overall accuracy criterion. The baseline beat a weak trained model, but it is not good enough on its own.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

All three runs use the same settings as my Milestone 5 run (3 epochs, learning rate 2e-5, batch size 16, max length 128) on `mps`, with 30 test posts per seed. The numbers come from `results_three_seeds_before.json`. In criterion 2, the three F1 scores are listed in the order `help_request` / `project_showcase` / `opinion_feedback`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
| --------- | ------ | ------- | ------ | --------- | ------- |
| 1. Overall accuracy | ≥ 0.65 | 0.433 | 0.467 | 0.500 | **MISSED** |
| 2. Per-label F1 | `help_request` ≥ 0.55, `project_showcase` ≥ 0.65, `opinion_feedback` ≥ 0.55 | 0.605 / 0.000 / 0.000 | 0.619 / 0.200 / 0.000 | 0.650 / 0.333 / 0.000 | **MISSED** |
| 3. Dataset balance | No label above 50% of `labels.csv` | 45% | 45% | 45% | **MET** |
| 4. Most-confident-third accuracy | ≥ 0.70 | 0.900 | 0.800 | 0.800 | **MET** (numerically; see below) |
| 5. Labelling consistency | ≥ 85% agreement on a repeat labelling of 20 posts | 90% (18 / 20) | 90% (18 / 20) | 90% (18 / 20) | **MET** (numerically; see below) |

**Notes on each verdict:**

1. **Overall accuracy missed on every seed.** Accuracy moved 0.067 across the three seeds, from 0.433 to 0.500. Even the best seed is 15 points below the 0.65 target, so the spread does not make this verdict uncertain.
2. **Per-label F1 missed.** `help_request` reached its target on all three seeds (0.605–0.650). `project_showcase` reached at most 0.333 against a 0.65 target, and `opinion_feedback` was 0.000 on every seed. The model never predicted `opinion_feedback` once in any of the three runs.
3. **Dataset balance met.** My largest label, `help_request`, is 90 of 200 posts (45%). This is a property of `labels.csv`, so it is the same for every seed and says nothing about the model.
4. **Most-confident-third accuracy met numerically on all three seeds, but it tells me little.** The model predicted `help_request` for 30, 29 and 27 of the 30 test posts on the three seeds. Its most confident guesses were therefore almost all `help_request`, and they were right mostly because the 13 real help requests are the largest group in the test split. The model's confidence stayed close to an even three-way guess (about 0.33). On average it was only 0.41–0.47 confident, whether right or wrong. A high score here does not show that the model knows when it is right.
5. **Labelling consistency met numerically.** This was measured on my labels in Milestone 4, not on the model, so it is the same for every seed. As I explain in The Dataset, I cannot be certain my first labels for records 1 and 2 were independent of AI discussion. The sample was also the 20 `cold` records, not a random sample, so the 90% should not be overstated.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

This matrix is from the **seed-42 run in `results_three_seeds_before.json`**. That run reproduced my Milestone 5 `results.json` exactly. Rows are the true labels; columns are the model's predictions.

| true \ predicted | help_request | project_showcase | opinion_feedback |
| ---------------- | ------------ | ---------------- | ---------------- |
| **help_request**     | 13 | 0 | 0 |
| **project_showcase** | 9  | 0 | 0 |
| **opinion_feedback** | 8  | 0 | 0 |

**My biggest off-diagonal number, and what it means:**

<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

The biggest error is 9: all 9 real `project_showcase` posts were predicted as `help_request`, and 0 real `help_request` posts went the other way. The model did not learn the boundary between showing off a finished project and asking for help; it pushed everything toward `help_request`. The same happened with `opinion_feedback`: all 8 real opinion posts were also called `help_request`. With seed 42, the model predicted only one label, so it learned no boundaries at all. It simply defaulted to my largest category.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| #   | Criterion | Target | Verdict | How I decided |
| --- | --------- | ------ | ------- | ------------- |
| 1   | Overall accuracy | ≥ 0.65 | **MISSED** | 0.433, 0.467 and 0.500 on seeds 42, 7 and 2024. All three are below the target, so the 0.067 spread does not change the verdict. |
| 2   | Per-label F1 | `help_request` ≥ 0.55, `project_showcase` ≥ 0.65, `opinion_feedback` ≥ 0.55 | **MISSED** | `help_request` passed on every seed (0.605, 0.619, 0.650). `project_showcase` failed on every seed (0.000, 0.200, 0.333), and so did `opinion_feedback` (0.000 on all three). |
| 3   | Dataset balance | No label above 50% | **MET** | My largest label, `help_request`, is 90 of 200 posts (45%). This is a property of `labels.csv`, not of the model. |
| 4   | Most-confident-third accuracy | ≥ 0.70 | **MET** (numerically) | 0.900, 0.800 and 0.800 on the three seeds, but the score is misleading (see below). |
| 5   | Labelling consistency | ≥ 85% | **MET** (numerically) | 18 of 20 (90%) on my repeat labelling of the 20 `cold` records in Milestone 4. The same caveats as in The Dataset apply. |

**Why criteria 4 and 5 are weaker than they look**

Criterion 4 passed only because the model almost always predicted `help_request`. When nearly every guess is `help_request`, the most confident guesses are almost all `help_request` too. They come out right mostly because the 13 real help requests are the largest group in the test split. The model's average confidence was only 0.41–0.47 on every seed, whether it was right or wrong, against 0.33 for an even three-way guess. So this pass does not show that the model knows when it is right.

Criterion 5 measures my labels, not the model. The sample was the 20 `cold` records, not a random sample. Three quarters of it was `project_showcase`, and only 1 record was `opinion_feedback`. I also cannot be sure my first labels for records 1 and 2 were independent of AI discussion. So the 90% tells me little about my hardest boundaries.

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

*What I observed (facts from my results files):*

- **My dataset was balanced, but the predictions were not.** `labels.csv` has 90 `help_request`, 58 `project_showcase` and 52 `opinion_feedback` posts. Yet the model predicted `help_request` for 30, 29 and 27 of the 30 test posts on seeds 42, 7 and 2024. A balanced dataset did not produce balanced predictions.
- **The biggest errors all point one way.** In the seed-42 confusion matrix, 9 real `project_showcase` posts and 8 real `opinion_feedback` posts were predicted as `help_request`. No real `help_request` post was predicted as anything else. Across all three seeds, the model never predicted `opinion_feedback` once.
- **The model's confidence and loss barely moved from guessing.** Its final training loss was 1.064, 1.041 and 1.029 on the three seeds. A model that gives every label an equal one-in-three chance scores about 1.099 on this measure. Validation loss was still falling at the end of epoch 3 on every seed. With 139 training posts and a batch size of 16, each run made only about 27 training updates in total.
- **The zero-shot baseline did better, but did not solve the task.** BART scored 0.567 accuracy against my trained model's 0.433 on the same 30 posts (seed 42). It found all 9 `project_showcase` posts (F1 0.621), but it also scored 0 on `opinion_feedback`. It labelled 7 of the 8 opinion posts as `project_showcase`, and it predicted `project_showcase` for 20 of the 30 posts. Neither model separated all three labels.

*What I think explains the misses (hypotheses, not proven causes):*

1. **The training was too short for the model to learn anything beyond a default.** This is my strongest hypothesis. The training loss stayed close to chance, validation loss was still falling when training stopped, and about 27 updates is very little. A model that has not learned the boundaries yet would fall back on the most common label it saw, and `help_request` is my largest label at 45%. This would explain why it leaned so heavily on `help_request` even though no label is above 50%. I have not tested this yet.
2. **`project_showcase` looks learnable, so its failure looks like a model problem rather than a labelling problem.** The baseline, given only my written definitions, found every `project_showcase` post in the test set. In my own repeat labelling, 15 of the 20 records were `project_showcase`, and I gave the same label both times on 13 of them. Both of my two disagreements were on this label, so my labelling of it was not perfect. Still, that suggests the label is described clearly enough to recognise. My fine-tuned model found at most 2 of the 9.
3. **`opinion_feedback` may be a genuinely hard boundary, possibly made worse by how I applied it.** Both models scored 0 on it. In my own taxonomy, many opinion posts also describe a problem or mention a project, so they overlap with the other two labels. My consistency check contained only one `opinion_feedback` record, so it cannot tell me whether I applied this label consistently. I cannot yet separate "hard boundary" from "inconsistent labels" for this class.

**What the staff agreement can and cannot tell me**

My 30 / 30 agreement with staff shows that I can apply someone else's written rules the way they intended on that set. It does not show that my own TakeMeter taxonomy is well defined. The staff set used different labels (`analysis`, `hot_take` and `reaction`) and different posts from a fictional running forum. The posts were also ordered in blocks of 10 by label. So I do not use it as evidence that my `help_request` / `project_showcase` / `opinion_feedback` labels are consistent. The closest evidence I have for that is my own 90% repeat-labelling result, with the limits above.

**Improvement candidate for Milestone 5 (not yet run)**

I plan to make one controlled change: increase `EPOCHS` from 3 to 10. Everything else stays the same: the dataset and labels, the splits and seeds (42, 7 and 2024), the learning rate, batch size, max length and evaluation method. This tests my strongest hypothesis, that the model was undertrained.

I predict that training for 10 epochs will lower training loss, reduce the model's tendency to predict `help_request` for nearly every post, and improve at least some of the per-label F1 scores across the three seeds. I will compare all three seeds, including validation loss, to see whether the gains are consistent or whether longer training overfits.

More epochs can also cause overfitting on 139 training posts, so I will look at validation loss as well as test scores.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** **30 / 30 = 100**%

I labelled all 30 staff posts myself under the staff taxonomy in `data/staff_taxonomy.md` before I saw any staff answers. My labels are in `my_staff_labels.csv`, committed in `c71d022` before I received `data/staff_labels.csv`. `python agreement.py` compared the 30 posts and found 30 matches and 0 disagreements (`agreement_results.json`, agreement rate 1.0).

| staff \ mine | analysis | hot_take | reaction |
| ------------ | -------- | -------- | -------- |
| **analysis** | 10 | 0 | 0 |
| **hot_take** | 0 | 10 | 0 |
| **reaction** | 0 | 0 | 10 |

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

There were no disagreements, so there are no cases to adjudicate and nothing to sort into the three piles.

**The seven posts staff marked as borderline**

Staff marked 7 of the 30 posts as hard cases in `data/staff_labels.csv`. I gave the same label as staff on all seven. I did not write a rationale for each post when I labelled them, so the reasons below are the staff's reasons, linked to the staff rules. They explain why each post is borderline, not how I decided.

| Post | Label | Why it is borderline |
| ---- | ----- | -------------------- |
| `sp_005`, "Carbon plates aren't the whole story…" | `analysis` | It opens like a hot take about shoes. Under rule 1 it is analysis, because the comparison that follows carries the argument: 2% and then another 2% in the same shoes, when only weekly mileage changed. |
| `sp_010`, "Three of us ran the same session…" | `analysis` | The conclusion is hedged and the sample is three people, but the claim rests on a checkable observation: three runners within 10 seconds over 5 miles. |
| `sp_014`, "Times are down about 3% since the shoe rules changed, so obviously…" | `hot_take` | Staff call this the canonical rule-1 case. The 3% is decoration on a conclusion already reached; "so obviously" does the work, not the number. |
| `sp_019`, "Nobody actually enjoys the long run…" | `hot_take` | It reads as reflective, almost like a reaction. Under rule 2 it is a general claim about everyone, not a report of what happened to the writer. |
| `sp_020`, "Every injury thread on this forum is the same injury…" | `hot_take` | It sounds observational and could in principle be checked, but no count is given. The writer is asserting, not showing, so it falls on the `hot_take` side of the `analysis` / `hot_take` boundary. |
| `sp_028`, "Third DNF this year…" | `reaction` | "Third this year" is countable, but nothing is argued from it. Rule 3 does not move it to `analysis`, because no fact is load-bearing. It is the writer reporting their own state. |
| `sp_029`, "Just realised the shoes I set every PB in…" | `reaction` | It could be read as a wry hot take about shoe discussions. Under rule 2 it is a reaction, because it reports the writer's own realisation, now. |

**What the pattern in my disagreements tells me:**

There is no pattern of disagreements to read, because there were none. That is a good sign that I can apply a written rule the way its authors did, but it does not prove my rule-following is flawless.

- **The set was ordered by label.** The first 10 posts were `analysis`, the next 10 were `hot_take`, and the last 10 were `reaction`. My labels followed those blocks. This arrangement may have made the task easier, so I would be cautious about treating 100% agreement as proof that I could achieve the same result on a shuffled set.

- **The agreement was higher than the taxonomy guidance suggests is typical.** The staff taxonomy says an agreement rate in the 70s is normal because some examples are ambiguous. My 100% agreement is encouraging, but this small, ordered sample cannot tell me how I would perform on new examples.
- **This measures a different taxonomy.** The result shows how well I applied the staff's `analysis` / `hot_take` / `reaction` rules to 30 fictional running-forum posts. It is not a measure of my own TakeMeter labels, and it says nothing directly about my model's accuracy. My classifier still scored only 0.433–0.500 accuracy on my own labels across the three seeds.

---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
| --------- | ------ | ------- | ------ | --------- | ------- |
| 1.        |        |         |        |           |         |
| 2.        |        |         |        |           |         |
| 3.        |        |         |        |           |         |
| 4.        |        |         |        |           |         |
| 5.        |        |         |        |           |         |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

**The gap between what I meant my labels to capture and what the model
learned:**

<!-- Two sentences. Your confusion matrix is the evidence. -->

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
