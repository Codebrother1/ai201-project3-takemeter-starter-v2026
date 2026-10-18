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

| Measure          | Baseline | Trained | Difference |
| ---------------- | -------- | ------- | ---------- |
| Overall accuracy |          |         |            |
| Macro F1         |          |         |            |
| F1 — `label_one` |          |         |            |
| F1 — `label_two` |          |         |            |

**What I predicted before I looked:**

<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**

<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
| --------- | ------ | ------- | ------ | --------- | ------- |
| 1.        |        |         |        |           |         |
| 2.        |        |         |        |           |         |
| 3.        |        |         |        |           |         |
| 4.        |        |         |        |           |         |
| 5.        |        |         |        |           |         |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |     |     |     |
| ---------------- | --- | --- | --- |
| \*\*\*\*         |     |     |     |
| \*\*\*\*         |     |     |     |
| \*\*\*\*         |     |     |     |

**My biggest off-diagonal number, and what it means:**

<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| #   | Criterion | Target | Verdict | How I decided |
| --- | --------- | ------ | ------- | ------------- |
| 1   |           |        |         |               |
| 2   |           |        |         |               |
| 3   |           |        |         |               |
| 4   |           |        |         |               |
| 5   |           |        |         |               |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** **_ / 30 = _**%

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

**1.**

> _The post:_
>
> _Staff said / I said:_
>
> _My call, and why:_
>
> _Which pile:_

**2.**

> _The post:_
>
> _Staff said / I said:_
>
> _My call, and why:_
>
> _Which pile:_

**What the pattern in my disagreements tells me:**

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
