# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

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

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

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

## 1. Overall accuracy

The classifier must achieve at least 0.75 accuracy on the held-out test set across all three random seeds.

**Why this target:**

Because TakeMeter has two labels, a random guesser would achieve around 0.50 accuracy on a balanced dataset. A target of 0.75 requires the classifier to learn meaningful differences between commentary and content-sharing posts while allowing for ambiguous Reddit posts.

---

## 2. Per-label performance

The classifier must achieve an F1 score of at least 0.65 for each label (commentary and content_share) across all three seeds.

**Why this target:**

Both labels are important to the purpose of TakeMeter. A classifier that performs well on content-sharing posts but frequently misses commentary would not reliably distinguish the two types of posts. The 0.65 threshold allows for some ambiguity in posts that contain both shared content and personal opinions.

---

## 3. Balance

The number of examples in the larger label must be no more than twice the number of examples in the smaller label in the training dataset.


**Why this target:**

If one label dominates the dataset, the classifier may learn to favor that label instead of recognizing meaningful differences between the two categories. Keeping the ratio at or below 2:1 helps ensure that the smaller category has enough examples to learn from. If the initial dataset exceeds this ratio, I will review the sampling and collect additional examples for the smaller label.

---

## 4. Labeling consistency

A second person, working only from my label definitions and decision rule (not my labels), independently labels 30 posts. Their labels must match mine on at least 24 of the 30 (80%).

The 30 posts are chosen at random from labels.csv, excluding the 20 cold-labeled posts, so the check is a separate sample. The second person labels from the post text alone and does not see my label or my note column.

**Why this target:**

The hardest boundary in my taxonomy is between commentary and content_share. Posts that include a trailer or announcement alongside a brief opinion may be read differently by different people. 80% agreement shows whether my definitions and decision rule are clear enough to apply consistently. With only 30 posts, one disagreement moves the score by about 3 points, so I treat the result as a rough signal rather than a precise measurement.


---

## 5. Prediction confidence

For each seed, among test-set predictions with a confidence score of at least 0.80, at least 85% must be correct. This applies only if the seed has at least 20 such predictions. If a seed has fewer than 20, [OPTION A / OPTION B below].

**Why this target:**

Posts that clearly express an opinion or simply share a link should be easier to classify than posts that mix both behaviors, so high-confidence predictions should be more reliable. This criterion checks whether the model's confidence is useful, rather than allowing confident mistakes on ambiguous posts. The 20-prediction minimum exists because with a test set of roughly 30 posts, a handful of errors could swing the percentage too much to mean anything.


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
