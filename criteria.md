# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. _"The model is accurate"_ is an
opinion. _"Every label has an F1 of at least 0.60 on the held-out set"_ is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — _"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"_. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area                  | A question it could answer                                            |
| --------------------- | --------------------------------------------------------------------- |
| Overall accuracy      | How often does it need to be right to be worth using?                 |
| Per-label performance | Is one label allowed to be much worse than the others?                |
| Balance               | How lopsided can your label counts get before it's a problem?         |
| Consistency           | If someone else labelled the same posts, how often should you agree?  |
| Confidence            | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall Classification Accuracy

The TakeMeter classifier must achieve an overall accuracy of at least **0.65 (65%)** on the held-out test set. Accuracy will be calculated by dividing the number of correctly classified posts by the total number of posts in the test set.

**Why this target:**

I chose 65% because it is meaningfully above the approximately 33% accuracy expected from uniform random guessing across three labels. The r/ClaudeAI community includes overlapping discussions, especially posts that combine requests for help with opinions or complaints. This target requires the model to perform substantially better than chance while accounting for those challenging boundaries. The actual label distribution will also be examined because chance performance depends on class balance.

---

## 2. Per-Label F1 Performance

The TakeMeter classifier must achieve the following minimum F1 scores on the held-out test set:

- `help_request`: at least **0.55**
- `project_showcase`: at least **0.65**
- `opinion_feedback`: at least **0.55**

Each label will be evaluated separately using its F1 score, which combines precision and recall. This criterion is met only if all three labels reach their individual thresholds.

**Why this target:**

I chose a higher target for `project_showcase` because posts presenting completed projects, tools, or automations tend to contain recognizable descriptions of what the author built. I chose slightly lower targets for `help_request` and `opinion_feedback` because posts about Claude's problems or limitations may combine requests for assistance with complaints or personal evaluations. These thresholds recognize that some categories are more ambiguous while still requiring meaningful performance on every label.

---

## 3. Dataset Balance

No single label may represent more than **50% of the total labeled posts** in `labels.csv`. This will be tested by counting the number of posts assigned to each label, dividing each count by the total number of labeled posts, and verifying that every label has a share of 0.50 or less.

**Why this target:**

I chose 50% because the r/ClaudeAI community contains several different types of discussions, and I want the dataset to represent help requests, project showcases, and opinion or feedback posts rather than being dominated by just one category. This is stricter than the assignment's 70% maximum and should reduce the risk of a classifier achieving high overall accuracy simply by favoring the most common label.

---

## 4. High-Confidence Prediction Accuracy

The TakeMeter classifier must achieve at least **0.70 (70%) accuracy** on the most-confident one-third of predictions in the held-out test set. Predictions will be ranked by their highest predicted class probability, and accuracy will be calculated on the top one-third of examples.

**Why this target:**

I chose 70% because I want the classifier's strongest predictions to be reasonably reliable before using its classifications to interpret discussions in the r/ClaudeAI community. Some posts combine questions, complaints, and opinions, so even a confident prediction may be wrong. This target tests whether the model's confidence is supported by correct classifications rather than assuming that high confidence always means high accuracy.

---

## 5. Human Labeling Consistency

At least **85% (17 out of 20)** of a randomly selected sample of 20 r/ClaudeAI posts must receive the same label when I independently classify them twice without looking at my original decisions. Agreement will be calculated as the number of matching labels divided by 20.

**Why this target:**

I chose 85% because consistent human labeling is important for training a reliable classifier. The `help_request` and `opinion_feedback` categories sometimes overlap when people complain about Claude while also asking questions. A target of 85% allows for a few genuinely ambiguous posts while still requiring clear labeling rules that I can apply consistently. Comparing two independent labeling passes will help identify whether the taxonomy needs clarification.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
