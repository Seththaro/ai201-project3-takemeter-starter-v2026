# Acceptance criteria — TakeMeter

Five criteria for a classifier that sorts Hacker News comments into `analysis`,
`hot_take`, `reaction` and `question`. Written in unit 5, **before anything was
trained**. Every criterion names a number and says where to read it from, so
someone who didn't write it can check it.

Context for the numbers: four labels, so a coin-flip guesser gets about 25%,
and a model that always answers `hot_take` (my biggest label, 29.5% of the first
200 rows) gets about 30%. The held-out test split is 15% of the file, so about
30 posts for the first 200 rows. One post in that split is worth roughly 3
accuracy points, and the smallest label (`question`, 39 rows) will have about 6
test posts.

---

## 1. Overall accuracy is at least 0.65 on every seed

`accuracy` in the test-split results is at least **0.65** for each of seeds 42,
7 and 2024. All three seeds have to clear it, not the average.

**Why this target:** 0.65 is more than double the 25% coin flip and about 2x
the 30% an always-`hot_take` model scores, so it is a long way from guessing.
It is not higher because I only have 200 labelled comments, and two of my
labels (`analysis` and `hot_take`) differ by one subtle rule: whether a
checkable fact is present. At roughly 30 test posts, 0.65 is about 20 correct,
and I'd call a tool that is right two times in three useful for sorting a
comment thread but not for anything automatic.

---

## 2. Every label has an F1 of at least 0.55 on every seed

For each of `analysis`, `hot_take`, `reaction` and `question`, `per_label.<label>.f1`
is at least **0.55** on each of the three seeds.

**Why this target:** accuracy can hide one label failing. `question` is my
smallest label (39 of the first 200 rows), so only about 6 test posts. An F1
on 6 posts swings by a lot per post, which is why this is 0.55 and not 0.70.
I expect `question` and `reaction` to be the easy ones (they have surface
cues like "?" and "thanks") and `analysis` versus `hot_take` to be where a
label drops, so this criterion is aimed at that boundary.

---

## 3. No label is above 40% or below 15% of `labels.csv`

Count the rows per label in `labels.csv`. The biggest label is at most **40%**
of the rows and the smallest is at least **15%**.

**Why this target:** the brief caps a label at 70%, but with four labels 70%
would mean the model could score well by mostly guessing one. 40% is
well under that. The 15% floor is there because of the test-split problem
above: below about 30 rows a label gets under 5 test posts and its F1 is mostly
luck. My first 200 rows are 29.5% `hot_take`, 26% `analysis`, 25% `reaction` and
19.5% `question`, which sits inside both limits.

---

## 4. I agree with the staff labels on at least 21 of the 30 staff posts

Label the 30 staff posts under the staff taxonomy, run `python agreement.py`,
and the agreement rate is at least **21 / 30 (70%)**.

**Why this target:** this is the only outside check on whether my own 200
labels are consistent. 70% is the point where I'd trust that a stranger
reading my definitions would label most posts the way I did. I'm not asking
for 90% because the staff set was picked to include ambiguous posts, and my own
hardest boundary (`analysis` vs `hot_take`, which depends on whether a "specific,
checkable" item is there) is a judgment call even with a written rule. Below
70% would mean I'm applying written rules inconsistently, which would
undermine every other number here.

---

## 5. Confident predictions are right more often than unsure ones

In the test-split results, `confidence.accuracy_most_confident_third` is at
least **0.80**, and it is at least **0.15** higher than
`confidence.accuracy_least_confident_third`. Both conditions hold on all three
seeds.

**Why this target:** if the model's confidence means anything, the third of
test posts it is surest about should be right far more than the third it is
least sure about. With about 10 posts per third, a 0.15 gap is about 1.5
posts, so it is a modest bar, and 0.80 means at most 2 wrong in the 10 it is
most sure of. If this fails, the model is unreliable in a way accuracy alone
would not show: a confident `hot_take` call would be no better than a coin flip,
and I could not use confidence to decide which comments need a human look.

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
