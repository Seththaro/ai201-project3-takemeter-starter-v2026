# TakeMeter

A small classifier that sorts Hacker News comments into four kinds: `analysis`,
`hot_take`, `reaction` and `question`. How to run it is in
[RUNNING.md](RUNNING.md).

## Stretch features I'm going for

Declared before I built anything beyond the taxonomy:

1. **A fourth label.** `question`. Hacker News threads have a lot of comments
   whose whole job is to ask something, and they don't fit `analysis`,
   `hot_take` or `reaction`. Details are in the Label Taxonomy below.
2. **A second training run with one hyperparameter changed.** Same data and
   seed as the main run, with epochs raised from 3 to 8 and nothing else changed.
3. **50 extra examples for the thinnest label.** The thinnest label in my first
   200 rows is `question` (39 rows), so I add 50 more `question` rows.

---

## Unit 6 stretch features I'm going for

Declared before I ran the baseline, the three seeds or any improvement:

1. **A confidence check.** Are the model's confident predictions right more often than its unsure ones? I report `accuracy_most_confident_third` against `accuracy_least_confident_third` for every seed and every run log.
2. **A per-label error write-up** for whichever label the confusion matrix says is worst: which label it gets confused with, and what those posts have in common.
3. **A second measured improvement.** A second change chosen from the diagnosis, run across the same three seeds and logged in the same table format as a third run log, with a statement of whether it helped.

---

## What This Does

TakeMeter sorts public Hacker News comments (news.ycombinator.com), the
discussion under tech, business and politics stories. Each comment gets one of
four labels: `analysis` if it backs its point with something specific you could
check, `hot_take` if it is a confident claim with nothing behind it, `reaction`
if it is a short emotional or social response such as thanks, a joke or
congratulations, and `question` if it is mainly asking for information. The
model is `distilbert-base-uncased` fine-tuned on 250 comments I labelled. The
point of the project is to find out what "working" means for a job where even
two readers could disagree on a comment, so criteria were written first, in
[criteria.md](criteria.md).

---

## Label Taxonomy

I read about 110 comments from several threads before settling on these. Four
differences kept showing up: some comments cite a number, date or link and
most don't; some are only a thank-you, a joke or a congratulation; some are
only a question; and a large middle group states an opinion and stops.

### `analysis`

**Definition:** A comment that supports its point with at least one specific, checkable item, such as a number, date, named source or link, direct quote, or concrete technical detail.

**Example 1:**
> If you make your Windows 11 USB with Rufus you can disable the TPM 2.0 and other hardware requirement checks. You totally should try a Linux install first though!

**Example 2:**
> If you want that you won't want this steam machine, HDMI 2.0 can do 4K60 HDR at 10-bit, but only with chroma subsampling (4:2:2 or 4:2:0) (not full 4:4:4).

### `hot_take`

**Definition:** A confident opinion, prediction or claim with nothing checkable behind it, including rhetorical questions and personal anecdotes that have no verifiable detail.

**Example 1:**
> WSJ is Fox News Platinum, I wouldn't overthink it

**Example 2:**
> 10 years is way too long for Google. It will be gone in 5 replaced by 3 other AI cloud services.

### `reaction`

**Definition:** A short emotional or social response (thanks, congratulations, a joke, sarcasm, a quip, or bare agreement) that doesn't make a claim of its own.

**Example 1:**
> what a sweet message. Congratulations!

**Example 2:**
> If intergalactic aliens landed on Earth tomorrow, HN would be flooded with alien news too ;-)

### `question` (stretch: fourth label)

**Definition:** A comment whose main purpose is to ask a real question and get information, an example or a clarification back.

**Example 1:**
> Has PianoVision been working for you to learn piano?

**Example 2:**
> Where are you finding a VPS in the EU for $10/year? Any I've seen are about 5-6 times that much.

**What `question` captures that the other three missed:** Without it, a
comment like the PianoVision one has nowhere to go. It makes no claim, so
it isn't `analysis` or `hot_take`, and it isn't an emotional response, so it isn't
`reaction`. Forcing it into one of those would have taught the model that
a question mark means `reaction`. In a 250-row sample it ended up the biggest
label (89 rows) once I added the extra 50, which is how common it is in threads.

### The hardest boundary

**Which two labels:** `analysis` vs `hot_take`.

**The decision rule I wrote down and used for the AI pre-labelling:** If the comment contains at least one specific
item a stranger could look up or verify without asking the author (a number, a
date, a named source or link, a direct quote, or a concrete technical detail)
*and uses it to back its point*, it is `analysis`, however calm or heated
the tone. If all it has is "I think", a prediction, an anecdote with no
verifiable detail, or a rhetorical question, it is `hot_take`, even if
it sounds confident and well argued.

Two smaller rules came out of testing the labels (see How I Used AI, Moment 1):

- **Rhetorical questions are `hot_take`, not `question`.** If the writer isn't waiting for an answer ("Isn't it obvious that...?"), it is a claim. (My own cold pass broke this once; see hard case 1.)
- **A short response that only reacts is `reaction`; one that asserts its own claim is `hot_take`.** "This is the correct take" is `reaction`. "Not true." is `hot_take`, because it asserts the opposite of something.

---

## The Dataset

**Where the posts came from:** Public Hacker News comments, pulled through the
public HN Search API (`hn.algolia.com/api/v1`, no login needed). I took the
comment trees of 45 stories from 2025 and 2026 with more than 80 comments each
(topics: AI, programming, privacy, startups, politics, hardware, Linux), which
gave about 34,000 comments. Then I filtered to comments of roughly 25 to 800
characters, dropped duplicates, and dropped any comment containing an `@` so no
email addresses or handles ended up in the file. I kept whole comments, nothing
trimmed. Labelling was done from a random sample, plus targeted pulls of short
comments and question marks to make sure the thin labels had enough rows. I also
skipped comments that are logistics rather than discussion (self-promotion,
"here's my blog", offers to email). That skipping matters: the dataset
doesn't cover those comments at all, so the model has never seen one.

**How I labelled them:** I wrote the definitions first, then labelled in
passes. Every row was first labelled by an AI assistant against the definitions
above (disclosed below), and the note column says `ai-prelabelled` on those rows.
I then ran a consistency check against my own decision rule: I listed every
`hot_take` row that contains a number or link and every `analysis` row that has
neither, and re-read them. That moved two rows (the "31 tanks" comment from
`hot_take` to `analysis`, and the tangled.org recommendation from `analysis`
to `hot_take`), and both are noted in the file. Then I did the cold pass: 20
rows (picked at random from the first 200) that I labelled myself, with no tool and no
label in view, marked `cold` in the note column. First pass was 200 rows, then I added
50 `question` rows (stretch), marked `extra-50 question set` in the note column.

**What the cold pass showed:** I agreed with the AI pre-label on only 10 of 20 rows
(50%), and I changed those 10 rows to my cold label. Five were the AI's `hot_take`
that I called `reaction` (short replies that assert something, like "I don't think I
would characterize this last era as one of global peace"). Three were the AI's
`analysis` that I called `hot_take` (the Qwen 3.7 comment, the "40-60 t/s" comment and
the "24 articles" comment, all of which contain specifics). One was the AI's `analysis`
that I called `reaction` (the "did not win a majority" comment), and one was the AI's
`hot_take` that I called `question` (the Turkey and Cyprus comment). So the written rules
and my gut disagree most on where a short assertive reply goes (`reaction` or `hot_take`)
and on whether a specific figure makes something `analysis`. That is the same
`analysis` vs `hot_take` boundary I picked as the hardest, and it means the 230 non-cold
labels follow my written rule more strictly than my own cold labels do. I kept my cold
labels as they are rather than rewrite them to match the rule, and I'm treating the
inconsistency as a finding to check against next unit's agreement report.

**Counts per label** (all 250 rows in `labels.csv`):

| Label | Count | Share |
|---|---|---|
| `question` | 89 | 35.6% |
| `hot_take` | 59 | 23.6% |
| `analysis` | 52 | 20.8% |
| `reaction` | 50 | 20.0% |
| **Total** | 250 | 100% |

The first 200 rows, before the extra 50, were `hot_take` 59 (29.5%), `analysis` 52
(26%), `reaction` 50 (25%) and `question` 39 (19.5%). No label is above 70% in
either version, and the biggest label is 35.6%.

**Three hard cases** (a fourth is included because it taught me the most)

**1.**
> *The post:* "Didn't NATO member Turkey invade NATO member Cyprus, with no consequences?"
>
> *Could have been:* `question` (it ends in a question mark) or `hot_take`.
>
> *I chose:* `question` in my cold pass, after the AI pre-label said `hot_take`. My written rule says `hot_take`, because the writer isn't asking for information: the "question" is the whole claim, and the facts in it are the claim rather than support for some other point (also, Cyprus is not a NATO member). I kept my cold label, and this row is the clearest example of my gut and my own rhetorical-question rule disagreeing. That rule came from comments like this one.

**2.**
> *The post:* "I think this works unironically. My mother is an avid gardener and can spend 8 hours a day gardening. When her life circumstances allowed for it, she hired a once a week gardener to do the tasks she didn't like [...] and still gardens the same amount."
>
> *Could have been:* `analysis` (it has numbers: 8 hours, once a week) or `hot_take`.
>
> *I chose:* `hot_take`, because nobody could check those numbers without asking the author. It's one person's experience used as proof. "Checkable" in my rule means a stranger could verify it.

**3.**
> *The post:* "Can't remember the source of this quote regarding software developers: "The best minds of my generation are thinking about how to make people click ads." We're all in the same boat."
>
> *Could have been:* `analysis` (a direct quote), `hot_take` or `reaction`.
>
> *I chose:* `analysis`, because my rule counts a direct quote as a checkable item and someone could search it. It's my weakest `analysis` row: the quote is decoration more than argument, and I expect the model to find this one hard.

**4.**
> *The post:* "> most of my fellow Americans did" / "Not true."
>
> *Could have been:* `reaction` (it's two words responding to someone else) or `hot_take`.
>
> *I chose:* `hot_take`, because "Not true." asserts the opposite of the quoted claim, so it makes a claim of its own. A reaction only expresses a feeling or agrees.

---

## The Training Run

**Base model:** `distilbert-base-uncased`, fine-tuned with the notebook's
Hugging Face `Trainer` setup (`takemeter.ipynb`, sections 1 to 5).

**Settings (main run, `results.json`):** 3 epochs, learning rate 2e-5, batch size 16,
128 tokens per post, seed 42, 250 rows in `labels.csv`. Trained on `cpu` with
torch 2.14.1+cpu on Windows, which `results.json` also records. Training took
about a minute or two.

**Anything I changed from the defaults, and why:** The main run uses the
defaults. The second run (stretch) changed one thing: **epochs from 3 to 8**,
with the data, seed and split exactly as in the main run. I chose epochs
because the main run was visibly under-trained: it predicted `question` for every `hot_take` and
`reaction` test post. I did not change learning rate, batch size or length.

**Split sizes:** 250 rows split 70/15/15 with seed 42 gives train 174 / val 38 / test 38.
Test posts per label: `analysis` 8, `hot_take` 9, `reaction` 8, `question` 13. Every label has
at least 8 test posts, but 8 is still small: one post moves a label's F1 by roughly
0.1 to 0.2, so expect jumpy numbers next unit. The first 200-row run had train
139 / val 31 / test 30, with 8 / 9 / 7 / 6 test posts, so `reaction` had 7 and `question` only 6.

**All three runs, held-out test split, seed 42, `cpu`:**

| | Run 1: 200 rows, 3 epochs | Run 2 (main): 250 rows, 3 epochs | Run 3: 250 rows, 8 epochs |
|---|---|---|---|
| File | `results_run1_200rows.json` | `results.json` | `results_run3_epochs8.json` |
| Test posts | 30 | 38 | 38 |
| Accuracy | 0.667 | 0.395 | 0.737 |
| Macro F1 | 0.604 | 0.233 | 0.681 |
| F1 `analysis` | 0.737 | 0.400 | 0.762 |
| F1 `hot_take` | 0.727 | 0.000 | 0.533 |
| F1 `reaction` | 0.667 | 0.000 | 0.500 |
| F1 `question` | 0.286 (6 test posts) | 0.531 (13 test posts) | 0.929 (13 test posts) |

I haven't judged these against my criteria yet; that is next unit's work.

**Stretch: second run with one hyperparameter changed (Run 2 → Run 3).** Raising
epochs from 3 to 8 on the same data, seed and test split moved accuracy
from 0.395 to 0.737 (+0.342) and macro F1 from 0.233 to 0.681 (+0.448). Every
label moved: `analysis` +0.362, `hot_take` +0.533, `reaction` +0.500,
`question` +0.398. At 3 epochs the model had collapsed to the biggest label:
all 9 `hot_take` and all 8 `reaction` test posts, and 6 of 8 `analysis` posts, were predicted as `question`,
which is why `hot_take` and `reaction` scored an F1 of 0. With 8 epochs it separates all four
labels, though `reaction` (F1 0.500, with 3 of 8 called `analysis` and 2 called `hot_take`) and `hot_take`
(F1 0.533, with 5 of 9 called something else) are still weakest.
Eight epochs took roughly three times as long as three (a few minutes on CPU).

**Stretch: 50 extra examples for the thinnest label (Run 1 → Run 2).** Adding
50 `question` rows (39 → 89) moved `question` F1 from 0.286 to 0.531 and made
`question` the majority class. The caveat is that this isn't like-for-like:
adding rows changes the split, so Run 2's test set is a different 38 posts
with 13 `question` posts instead of 6. Overall accuracy went down (0.667 →
0.395) because the under-trained 3-epoch model overcorrected: it called nearly every
test post `question` (precision for `question` was 0.36). So the extra rows
helped the label the model was missing, but only once the model
trained long enough (Run 3, `question` F1 0.929) did they stop costing the other
labels.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I gave an AI assistant my first-draft definitions (`analysis`: specific, checkable fact; `hot_take`: confident claim with no support; `reaction`: immediate response; `question`: asks something) and asked it to write eight comments that sit on the boundaries between labels, so I could test whether I could classify them cleanly before labelling 200 things.
- *What came back:* Eight boundary comments, such as "Wow, 2.3 million downloads in a week. Insane." (a number used as an exclamation), "Isn't it obvious Rust will replace C within a decade?" (a question mark with no real question), "I've run Postgres in production for 8 years and it has never lost data on me." (a number nobody can check), and "Thanks, that fixed it! Run `npm cache clean --force` then reinstall." (thanks plus a concrete command). I could place two of the eight from the first draft; the other six were genuinely unclear.
- *What I changed:* I tightened the definitions and added rules: a number only counts when it backs a claim; "checkable" means a stranger could verify it; rhetorical questions are `hot_take`, not `question`; a short response that only reacts is `reaction`, while one that asserts something is `hot_take`; and when a comment mixes a question with an opinion, its main purpose picks the label.

**Moment 2**

- *What I asked for:* After the first labelling pass, I asked the assistant to list every `hot_take` row that contains a digit or link and every `analysis` row that contains neither, so I could check them against my written decision rule instead of trusting memory.
- *What came back:* 24 rows. Most were fine (a rhetorical question with an unverifiable number, a prediction with a year in it). Two broke my own rule.
- *What I changed:* I relabelled "Heavy lifting like sending 31 tanks while putin had 10000?" from `hot_take` to `analysis`, because it cites two specific figures to make its point. I relabelled the tangled.org recommendation from `analysis` to `hot_take`, because it names a project but gives nothing checkable. I then re-ran all three training runs so `results*.json` match the final `labels.csv`.

**Pre-labelling disclosure:** All 250 rows in `labels.csv` were pre-labelled by an AI assistant against the definitions in this README; the note column says `ai-prelabelled` on those rows. The comments were collected by a script against the public Hacker News API, also written with AI help. I read the labels, changed two (above), and labelled 20 rows cold myself, with no tool and no label in view; those rows say `cold` in the note column. I disagreed with the AI on 10 of those 20 and changed
them to my label. The 50 extra `question` rows were pre-labelled the same way. The criteria in `criteria.md` were also drafted with AI help; they were filed, and committed, before any training run.

---

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

Written before I ran `baseline.py`, and after unit 5, so I already knew my main run (3 epochs) scored only 0.395 accuracy on its 38 test posts. Given that, my prediction is: the zero-shot baseline will beat my trained model on overall accuracy (I'd guess 0.45 to 0.55), and the trained model will only beat it on `question`, where a fine-tuned model can learn the "?" cue and a zero-shot model reading a long definition may not. I expect the baseline to be weakest on `reaction` and `hot_take`, because zero-shot reads my definitions literally and my `hot_take` definition is mostly defined by what is absent.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

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

**Agreement rate:** ___ / 30 = ___%

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
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

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
