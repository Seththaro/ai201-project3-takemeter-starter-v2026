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

### Unit 6 additions

**Moment 3**

- *What I asked for:* After improvement 1, criterion 1 (accuracy at least 0.65 on every seed) came out MET on seeds 0.737, 0.763 and 0.658. I asked the AI assistant to argue the opposite verdict as strongly as it could, using only the criterion, the target and the three results.
- *What came back:* The best argument against MET was that 0.658 is 25 correct posts out of 38, so one fewer correct post gives 0.632 and a miss; that the notebook itself flags an accuracy spread over ten points (0.105 here) as a dataset too small for a stable measure; and that "every seed" was meant to protect against exactly a friendly draw.
- *What I changed:* I kept the verdict MET, because the criterion is written as a literal threshold on each of the three runs and all three clear it. But I added the fragility to the diagnosis and wrote it into What's Still Broken: this MET sits one test post above the line on seed 2024, and the three-seed spread is over ten points.

**Moment 4**

- *What I asked for:* For the per-label error write-up, I had the assistant retrain seed 42 with the final settings and save every held-out prediction with its confidence (`error_analysis_seed42.csv`), then asked what the nine true `hot_take` posts that the model got wrong or barely got right had in common.
- *What came back:* Five of the nine true `hot_take` posts were misclassified: two as `reaction` (both short and written in the first person), two as `question` (both rhetorical questions), one as `analysis` (a long post naming a specific project). It also pointed out that the model's two false `hot_take` calls were a joke and a post with a number in it.
- *What I changed:* I read each of the nine posts myself against the definitions before using the pattern, and wrote the pattern up as the error write-up below (surface cues, not meaning). It did not change any labels, because labels are fixed in this unit.

**Pre-labelling disclosure (unit 6):** The 30 extra `reaction` posts in improvement 2 were pre-labelled by an AI assistant against my definitions, exactly like the unit 5 rows, and are marked `ai-prelabelled; improvement-2 reaction set` in the note column. They were drawn from the same Hacker News pull as the original 250. The notebook's three-seed cells, the baseline and the error dump were run by an AI assistant from the repo's own scripts; the numbers in this README were copied from the files they wrote. My 30 staff-set labels were typed in by me with a script that shows each post without a label (see Agreement Report). The stand-in reference labels for that set (`data/ai_reference_labels_NOT_STAFF.csv`) were written by an AI assistant under the staff rules, in the same session after my labels were saved, and the disagreement adjudications were drafted with the assistant from the staff rules; they are not staff's labels.


---

## Baseline vs. Trained

All numbers below are scored on the **same 38 posts** (`test_split.csv`, the seed-42 split of the 250-row file). The baseline is `facebook/bart-large-mnli` run zero-shot on this machine with no training, using only the definitions in `label_definitions.txt`. The trained model is the unit 5 main run in `results.json` (3 epochs, seed 42). Device for every trained run in this README: **cpu** (torch 2.14.1+cpu, Windows), as `results.json` and the `results_three_seeds_*.json` files record. This is the table `python baseline.py --trained results.json` prints:

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.237 | 0.395 | +0.158 |
| Macro F1 | 0.189 | 0.233 | +0.043 |
| F1 — `analysis` | 0.316 | 0.400 | +0.084 |
| F1 — `hot_take` | 0.000 | 0.000 | +0.000 |
| F1 — `reaction` | 0.308 | 0.000 | -0.308 |
| F1 — `question` | 0.133 | 0.531 | +0.397 |

The same posts, but with the baseline run on bare label names (no definitions, `baseline_bare_names.json`), which turned out to be the stronger baseline:

| Measure | Baseline (bare names) | Trained (3 epochs) | Difference |
|---|---|---|---|
| Overall accuracy | 0.526 | 0.395 | -0.131 |
| Macro F1 | 0.402 | 0.233 | -0.169 |
| F1 — `analysis` | 0.000 | 0.400 | +0.400 |
| F1 — `hot_take` | 0.353 | 0.000 | -0.353 |
| F1 — `reaction` | 0.533 | 0.000 | -0.533 |
| F1 — `question` | 0.722 | 0.531 | -0.192 |

And against the trained model after improvement 1 (8 epochs, seed 42, the same 38 test posts, from `results_three_seeds_after.json`):

| Measure | Baseline (definitions) | Baseline (bare names) | Trained (8 epochs) | Difference vs. definitions | Difference vs. bare names |
|---|---|---|---|---|---|
| Overall accuracy | 0.237 | 0.526 | 0.737 | +0.500 | +0.211 |
| Macro F1 | 0.189 | 0.402 | 0.681 | +0.492 | +0.279 |
| F1 — `analysis` | 0.316 | 0.000 | 0.762 | +0.446 | +0.762 |
| F1 — `hot_take` | 0.000 | 0.353 | 0.533 | +0.533 | +0.180 |
| F1 — `reaction` | 0.308 | 0.533 | 0.500 | +0.192 | -0.033 |
| F1 — `question` | 0.133 | 0.722 | 0.929 | +0.796 | +0.207 |

`baseline_results.json` is the run that uses my definitions from `label_definitions.txt` (the default). `baseline_bare_names.json` is the same model and posts with the definitions removed, which I ran on purpose to measure what my definitions were worth. The run after improvement 2 is not in this table because adding 30 rows changed the split to a different 42 posts, so it can't be compared with the baseline like-for-like.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

Written before I ran `baseline.py`, and after unit 5, so I already knew my main run (3 epochs) scored only 0.395 accuracy on its 38 test posts. Given that, my prediction is: the zero-shot baseline will beat my trained model on overall accuracy (I'd guess 0.45 to 0.55), and the trained model will only beat it on `question`, where a fine-tuned model can learn the "?" cue and a zero-shot model reading a long definition may not. I expect the baseline to be weakest on `reaction` and `hot_take`, because zero-shot reads my definitions literally and my `hot_take` definition is mostly defined by what is absent.

My prediction was only half right. I expected the baseline to beat the 3-epoch model on accuracy and it didn't against the version that uses my definitions (0.237 vs 0.395), though it did against the bare-names version (0.526 vs 0.395). I was right that `hot_take` would be the baseline's weakest label (F1 0.000 with definitions), and I was wrong that the baseline would lose to the trained model on `question`: with bare names it got 0.722 against the 3-epoch model's 0.531.

**What the gap actually means:**

1. **My definitions made the baseline worse, not better.** With definitions the baseline scored 0.237 accuracy, below the 25% a coin flip gets across four labels. Without them it scored 0.526. So the definitions were worth **-0.289** accuracy. Looking at the predictions, the definition prompt sent 30 of 38 posts to `analysis` (its definition is the longest and most general, so a general-purpose entailment model finds it the best fit for almost any post), while bare names sent 23 of 38 to `question` and 8 to `hot_take` and never predicted `analysis` at all. So the baseline is only as fair as the definitions, and mine are long and abstract for a zero-shot model.
2. **The 3-epoch fine-tune was worse than doing nothing.** Against the stronger baseline (bare names, 0.526) my unit 5 main run (0.395) lost by 0.131. Fine-tuning only paid off after improvement 1: 8 epochs reached 0.737, which beats the best baseline by 0.211 accuracy and 0.279 macro F1 on these same 38 posts, and beats the with-definitions baseline by 0.500.
3. **Where the trained model clearly wins:** `analysis` (0.762 vs 0.316 or 0.000) and `question` (0.929 vs 0.133 or 0.722). **Where the baseline is still competitive:** `hot_take` and `reaction` are within 0.2 of each other (bare names 0.353 and 0.533 vs trained 0.533 and 0.500), so the training added little there. These are also the two labels my own cold labelling disagreed on, which points at the labels rather than the model.

---

## Run Log — Before

The system as it stood at the end of unit 5: `distilbert-base-uncased`, 3 epochs, learning rate 2e-5, batch size 16, 250 rows. Three trials means three different random splits (seeds 42, 7 and 2024) of the same file, retrained from scratch each time. Written by notebook section 6 into `results_three_seeds_before.json`. Device: **cpu**. Each split has 38 test posts (8 `analysis`, 9 `hot_take`, 8 `reaction`, 13 `question`).

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on every seed | 0.395 | 0.342 | 0.368 | MISSED |
| 2. Every label F1 | ≥ 0.55 on every seed | min 0.000 (`hot_take`, `reaction`) | min 0.000 (`analysis`, `hot_take`, `reaction`) | min 0.000 (`analysis`, `hot_take`) | MISSED |
| 3. Label balance | biggest ≤ 40%, smallest ≥ 15% of `labels.csv` | 35.6% / 20.0% | 35.6% / 20.0% | 35.6% / 20.0% | MET |
| 4. Agreement with staff | ≥ 21 of 30 | not a per-seed measure | — | — | NOT TESTED (no staff labels; see Agreement Report) |
| 5. Confidence | most-confident third ≥ 0.80 and ≥ 0.15 above least-confident third | 0.917 vs 0.167 | 0.833 vs 0.000 | 0.917 vs 0.167 | MET |

Per-label F1 on each seed (the data behind criterion 2):

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.395 | 0.342 | 0.368 | 0.053 |
| Macro F1 | 0.233 | 0.127 | 0.186 | 0.105 |
| F1 — `analysis` | 0.400 | 0.000 | 0.000 | 0.400 |
| F1 — `hot_take` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `reaction` | 0.000 | 0.000 | 0.222 | 0.222 |
| F1 — `question` | 0.531 | 0.510 | 0.520 | 0.021 |

Accuracy moved only 0.053 across seeds here, but that is stable for a bad reason: on every seed the model was stuck on one label.

### Confusion matrix

Seed 42, 3 epochs, device **cpu**. Rows are true labels, columns are the model's guesses.

| true \ predicted | analysis | hot_take | reaction | question |
|---|---|---|---|---|
| **analysis** | 2 | 0 | 0 | 6 |
| **hot_take** | 0 | 0 | 0 | 9 |
| **reaction** | 0 | 0 | 0 | 8 |
| **question** | 0 | 0 | 0 | 13 |

**My biggest off-diagonal number, and what it means:** 9 real `hot_take` posts were called `question` and 0 went the other way, and the `reaction` row is the same (8 of 8 called `question`). That's not one boundary the model failed to learn: it is the model answering `question` for 36 of 38 posts (every post except the 2 it got right as `analysis`), which is the majority label (35.6% of the file) and where the column precision of 0.36 matches the label's 34% share of the test split. Seeds 7 and 2024 show the same collapse (all 38 and 37 of 38 predicted `question`).

---

## Verdicts and Diagnoses

Verdicts are against the unit 5 targets in `criteria.md`, which are unchanged (no criterion was revised: all five were measurable). A target has to hold on all three seeds. The first verdict column is the system at the end of unit 5; the other two are after each improvement.

| # | Criterion | Target | Before | After 1 (epochs 8) | After 2 (+30 `reaction`) | How I decided |
|---|---|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 on every seed | MISSED | MET | MET | Before: 0.395, 0.342, 0.368, all under. After 1: 0.737, 0.763, 0.658, all at or over 0.65 (seed 2024 is 25 of 38). After 2: 0.810, 0.762, 0.714. |
| 2 | Every label F1 | ≥ 0.55 on every seed | MISSED | MISSED | MISSED | Needs 12 numbers (4 labels × 3 seeds) all at 0.55 or over. Before: 7 of 12 were 0.000. After 1: four are under (seed 42: `reaction` 0.500 and `hot_take` 0.533; seed 2024: `reaction` 0.400 and `analysis` 0.421). After 2: only `hot_take` on seed 42 (0.533) is under. |
| 3 | Label balance | biggest ≤ 40%, smallest ≥ 15% | MET | MET | MET | Counted in `labels.csv`: 35.6% / 20.0% at 250 rows, 31.8% / 18.6% at 280 rows. |
| 4 | Agreement with staff | ≥ 21 of 30 | NOT TESTED | NOT TESTED | NOT TESTED | `data/staff_labels.csv` never reached me, so there is no agreement rate against staff. A stand-in against an AI-written reference scores 24 of 30 (80%), but that is not the staff set and I'm not calling the criterion MET on it. |
| 5 | Confidence | most-confident third ≥ 0.80 and ≥ 0.15 above least | MET | MISSED | MET | Before: 0.917/0.167, 0.833/0.000, 0.917/0.167. After 1: seed 2024 is 0.750 vs 0.500, under 0.80. After 2: 0.929/0.571, 0.857/0.643, 0.929/0.500. |

**Diagnoses**

**Criterion 1 (before): MISSED. Cause: under-training, not the labels.** Accuracy was 0.395, 0.342 and 0.368, but the confusion matrix shows why: on seed 42 the model called 36 of 38 test posts `question`, and on seeds 7 and 2024 it called 38 and 37 of 38 `question`. `question` was the biggest label (35.6% of the file), so this is a model that collapsed to the majority class: its `question` precision of 0.36 equals that label's 34% share of the test split. The evidence that it was training and not labelling or data is that changing only `EPOCHS` from 3 to 8 took accuracy to 0.737, 0.763 and 0.658 on the same splits with no label touched. It is not the "task out of reach for 200 examples" cause: the same data reached 0.74 once trained longer.

**Criterion 2 (before): MISSED. Cause: the same collapse.** 0.000 F1 for `hot_take` on every seed and for `reaction` on two of three is what a model predicting `question` for everything scores, since a label never predicted has zero precision and recall.

**Criterion 2 (after 1 and after 2): MISSED. Cause: a boundary I applied inconsistently, and (after 1) too few `reaction` examples.** After 1, `reaction` F1 was 0.500 and 0.400 on two seeds, and the matrix sends its misses to `analysis` (3 of 8 on seed 42) and `hot_take` (2 of 8). Two pieces of evidence. First, `reaction` was my thinnest label at 50 rows, so only 8 test posts per split and one post moves its F1 by about 0.1 to 0.15. Adding 30 rows fixed that: `reaction` F1 went to 0.833, 0.750 and 0.720, all over 0.55. Second, my cold-labelling check found that I disagreed with the AI pre-label on 10 of 20 rows (50%), and 5 of those 10 were `reaction` vs `hot_take`, with 3 more where I called a post with a specific figure `hot_take` instead of `analysis`. That is direct evidence that the `reaction`/`hot_take` and `analysis`/`hot_take` boundaries were applied two different ways in the training data. After 2 the one remaining miss is `hot_take` on seed 42 (0.533, seeds 7 and 2024 are 0.588 and 0.556), a miss by 0.017 that sits on the same boundary: see the per-label error write-up, where all five `hot_take` errors follow surface cues rather than meaning. This is a labelling cause plus a genuinely hard label pair, not a model-size one: `hot_take` is defined only by what's absent.

**Criterion 5 (after 1): MISSED. Cause: measurement noise from a tiny third, not a failing model.** Seed 2024's most-confident third was 9 of 12 correct (0.750) against a target of 0.80, one post short, and the other two seeds passed (0.917 and 0.833). With 12 posts per third one post is worth 0.083, so this miss is smaller than the resolution of the measure. After improvement 2 (14 posts per third) all three seeds pass, but I wouldn't read the MET as more than "no evidence against".

**Criterion 3: MET**, nothing to diagnose. **Criterion 4:** NOT TESTED against staff, so there is nothing to diagnose; the stand-in comparison in the Agreement Report is evidence about my own labelling, not a verdict.

---

## Confidence check (stretch)

**Question:** are the model's confident predictions right more often than its unsure ones?

The notebook ranks the held-out posts by the model's top softmax probability and compares accuracy in the most-confident third against the least-confident third (12 posts per third at 38 test posts, 14 at 42).

| Run | Seed | Most-confident third | Least-confident third | Gap | Mean confidence when right | Mean confidence when wrong |
|---|---|---|---|---|---|---|
| Before (3 epochs) | 42 | 0.917 | 0.167 | 0.750 | 0.461 | 0.352 |
| Before (3 epochs) | 7 | 0.833 | 0.000 | 0.833 | 0.439 | 0.329 |
| Before (3 epochs) | 2024 | 0.917 | 0.167 | 0.750 | 0.437 | 0.344 |
| After 1 (8 epochs) | 42 | 0.917 | 0.417 | 0.500 | 0.607 | 0.401 |
| After 1 (8 epochs) | 7 | 0.833 | 0.500 | 0.333 | 0.551 | 0.466 |
| After 1 (8 epochs) | 2024 | 0.750 | 0.500 | 0.250 | 0.585 | 0.483 |
| After 2 (+30 `reaction`) | 42 | 0.929 | 0.571 | 0.358 | 0.675 | 0.462 |
| After 2 (+30 `reaction`) | 7 | 0.857 | 0.643 | 0.214 | 0.607 | 0.512 |
| After 2 (+30 `reaction`) | 2024 | 0.929 | 0.500 | 0.429 | 0.625 | 0.481 |

**Answer: yes, in all nine runs.** The most-confident third was more accurate than the least-confident third every time (gap from 0.214 to 0.833), and mean confidence was higher on right answers than wrong ones every time. Two cautions. First, a third is only 12 to 14 posts, so one post is worth about 0.07 to 0.08, and the 0.750 on seed 2024 after improvement 1 is nine posts out of twelve. Second, the before-run result flatters the model: it was collapsed onto `question`, so "confident and right" mostly means "confident that a post is a `question`, and it was one". The gap narrowed from before to after 1 (0.750/0.833/0.750 → 0.500/0.333/0.250) because the model stopped being uniformly sure about one label, and it stayed above 0.20 after improvement 2. So confidence is a usable signal for which posts to double-check, but a weak one.

---

## Per-label error write-up (stretch)

**Which label is worst:** `hot_take`. After improvement 2 it is the only label with an F1 under 0.55 on any seed (0.533, 0.588, 0.556), and it was the weakest or second weakest label on two of three seeds after improvement 1 too. Its confusion across the three seeds (counts of true `hot_take` posts, 9 per seed) goes to `reaction` 2, 1, 3; to `question` 2, 3, 0; and to `analysis` 1, 0, 1. The reverse is smaller: on seed 42 only 2 non-`hot_take` posts were called `hot_take`.

**What the misses have in common (seed 42, final settings, `error_analysis_seed42.csv`):** five of the nine true `hot_take` posts were wrong, and each one trips a surface cue:

1. **Short and written about the author, called `reaction` (2 posts).** "Of course i care that i lose money. I dont care that google loses money." (confidence 0.39) and "Sadly yes. Sam Altman wants online ID face scanning technology just like the administration does." (0.61). Both are claims about the world, but they are short and begin with a personal or feeling word, which is what most of my `reaction` rows look like.
2. **Rhetorical question, called `question` (2 posts).** "I'm surprised so many people would be impacted by this. Why bet your livelihood on a corporate sponsored, second class ecosystem?" (0.82, the most confident wrong call) and "> heavy brick like every Linux phone I read about / So you didn't read about Pinephone? Also, fighting against something requires efforts, you know." (0.36). My rule says rhetorical questions are `hot_take`, but the model just learned that a question mark means `question`.
3. **Names something specific, called `analysis` (1 post).** The tangled.org recommendation: a long post with a named project and technical-sounding detail but nothing checkable behind it (0.33, the least sure call in the group). It is the row I moved from `analysis` to `hot_take` in unit 5.

Four of the five misses were low confidence (0.33 to 0.61), and the one exception is the rhetorical question at 0.82. The two posts the model wrongly called `hot_take` were "lmao US needs China's input on how to build the GFW." (true `reaction`, a joke) and a long post about the US vote total (true `analysis`, it quotes "78 million children").

**What this says:** `hot_take` is defined by what is *missing* (no checkable support), so the model has nothing positive to latch onto and falls back on surface cues: length and first-person words mean `reaction`, a question mark means `question`, a named technical thing means `analysis`. The same cues show up in my cold pass, where 5 of my 10 disagreements with the AI were `reaction` vs `hot_take`, so some of this is labels that apply the rule loosely, not only the model.

---

## Agreement Report

**Status: this is a stand-in, not the staff comparison.** I labelled the 30 posts in `data/staff_posts.csv` myself under the staff taxonomy in `data/staff_taxonomy.md` (`my_staff_labels.csv`, typed in with a script that shows one post at a time with no label in view). The staff's own labels (`data/staff_labels.csv`) never reached me from my TF, so I can't report an agreement rate against staff. The Verdicts table therefore says NOT TESTED for criterion 4. To get *some* evidence on whether I can apply a written rule the way it was written, I had an AI assistant write its own labels for the same 30 posts from the staff definitions (`data/ai_reference_labels_NOT_STAFF.csv`) and ran `agreement.py` against that (`agreement_results.json`). Two limits: that reference is the assistant's reading of the staff rules, not staff's, and it was written in the same session after my labels were saved, so it isn't fully independent of mine. If the real file arrives, `python agreement.py` on it replaces everything below.

**Agreement rate (against the AI reference, not staff):** 24 / 30 = 80%

**Disagreements.** Every one is argued from the staff rules in `data/staff_taxonomy.md`. In all six the reference label is the better reading, and four of the six are posts where I picked `reaction` or `hot_take` for a post whose facts carry the argument.

**1.**
> *The post:* "I tracked sleep against session quality for eleven weeks. Under seven hours the next day's intervals were 6-8 seconds per mile slower, every time bar one. Small sample, one runner, but it's consistent enough that I've stopped scheduling hard days after late shifts."
>
> *Reference said / I said:* `analysis` / `reaction`
>
> *My call, and why:* The reference is right. Rule 3 says recency does not outrank evidence, and this post isn't even recent: it is an eleven-week log with a measured effect (6-8 seconds per mile, one exception) and a decision that follows from it. The definition of `reaction` is "an immediate response to something that just happened... in the last few hours", which this is not. I called it `reaction` because it's about my own running, and rule 2 says what separates `reaction` from `hot_take` is the writer's experience, but I skipped the part that it must be immediate and feeling-led.
>
> *Which pile:* (a) The rule covered it and I applied it loosely.

**2.**
> *The post:* "The club championship scoring changed in 2019 from top-6 to top-10. Since then our B team has finished above the A team twice. That's not depth improving, it's the scoring rewarding turnout."
>
> *Reference said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* The reference is right. Rule 1 asks whether the fact carries the argument, and its second example ("Times are down 3% since 2017, and the shoe rule changed in 2017. That's the most likely single cause.") has exactly this shape: a dated change, an outcome after it, and a conclusion that depends on both. Take out the 2019 change and the B-team result and there is nothing left of the post. I saw a confident conclusion in the last sentence and stopped there.
>
> *Which pile:* (a) The rule covered it and I applied it loosely.

**3.**
> *The post:* "Everyone blames the hill at 18 miles. I split the field into people who ran the first half under 1:45 and over, and the slowdown after 18 was almost identical for both groups. If the hill were the problem the fast starters would fall apart harder."
>
> *Reference said / I said:* `analysis` / `reaction`
>
> *My call, and why:* The reference is right, and this one is clearer than the last. The post runs a split, reports the result ("almost identical for both groups") and draws a conclusion from the result ("if the hill were the problem..."). The fact is the argument (rule 1), and there is no feeling in it to make it a `reaction` (rule 2). I have no good defence of `reaction` here; "Everyone blames..." reads like an aside, and I let the first sentence set the label.
>
> *Which pile:* (a) The rule covered it and I applied it loosely.

**4.**
> *The post:* "Ran the same loop in 4C and in 24C within a fortnight, same effort by heart rate. 90 seconds slower in the heat over six miles. People keep saying heat costs you 'a bit' and it's more than that."
>
> *Reference said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* The reference is the better reading, but this is the one where the rule really doesn't say. The staff `analysis` definition counts "a time" as a checkable fact, and 90 seconds over six miles at matched heart rate is the evidence for the closing claim, so rule 1 points to `analysis`. But the measurement is one runner, once, and nobody else can check it. The staff rules don't say whether a personal measurement counts as "checkable". I labelled it `hot_take` because that is what my own unit 5 rule says ("a stranger could verify it"), so I applied my taxonomy to a post I was supposed to read under theirs. That's the real lesson.
>
> *Which pile:* (b) The rule genuinely doesn't say (whether an n=1 personal measurement is "checkable"). It is also a bit of (a): I let my own rule leak in.

**5.**
> *The post:* "The club needs to stop pretending the Tuesday session is for everyone. It isn't and it never has been."
>
> *Reference said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* The reference is right. Rule 2 asks whether the post is about a claim or about the writer, and gives "A post can be heated and still be `hot_take` if it is making a general claim." This one asserts something about the club ("it never has been") and reports nothing that happened to the writer and no feeling. It reads like frustration, which is probably why I called it `reaction`, but the staff definition makes `reaction` about "the last few hours".
>
> *Which pile:* (a) The rule covered it and I applied it loosely.

**6.**
> *The post:* "Every injury thread on this forum is the same injury and the same three replies."
>
> *Reference said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* The reference is right on the rules, though this is the closest I came to a defensible argument for my side: it is a tired aside that sounds like venting. But it asserts something general about the forum ("every... thread") and there is no event and no feeling in it, so rule 2 sends it to `hot_take`. A sarcastic general claim is still a general claim.
>
> *Which pile:* (a) The rule covered it and I applied it loosely. I don't have a pile (c) case in this set.

**What the pattern in my disagreements tells me:** Five of the six disagreements are rule-covered misses (pile a) and one is a real gap in the staff rules (pile b), and none was a pile (c) win. Four of six are `reaction` or `hot_take` calls on posts whose facts do the arguing (posts 1 to 4), and two are `reaction` calls on general claims (posts 5 and 6), so I over-called `reaction` and under-called `analysis` on exactly the boundaries the staff rules were written for. That matters for my model: my unit 5 cold pass found the same habit (5 of 10 disagreements with the AI pre-label were `reaction` vs `hot_take`, 3 more were specifics I called `hot_take`), and my model's matrices show `reaction` posts going to `analysis` and `hot_take` posts going to `reaction`. So some of the model's confusion is training labels where I applied the boundary loosely, which supports the labelling cause I gave for criterion 2.

---

## The Improvement

Improvement 1 (required): epochs 3 → 8.

**What I changed:** `EPOCHS` from 3 to 8 in the notebook. Nothing else: same 250 rows, same labels, same learning rate (2e-5), batch size (16), length (128) and the same three seeds, so the splits are identical to the before run.

**Which diagnosis pointed at it:** The before-run confusion matrix shows the model collapsed onto `question` on every seed (see Run Log — Before and Verdicts and Diagnoses): it had not trained long enough to separate the labels, and the unit 5 stretch run had already shown that 8 epochs fixed the collapse on one seed. The evidence that this was training and not labels is that this one change takes accuracy from 0.395 to 0.737 on seed 42 without touching a single label.

### Run Log — After

Same table, same seeds, `results_three_seeds_after.json`, device **cpu**, 38 test posts per seed as before.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on every seed | 0.737 | 0.763 | 0.658 | MET |
| 2. Every label F1 | ≥ 0.55 on every seed | min 0.500 (`reaction`) | min 0.556 (`hot_take`) | min 0.400 (`reaction`) | MISSED |
| 3. Label balance | biggest ≤ 40%, smallest ≥ 15% | 35.6% / 20.0% | 35.6% / 20.0% | 35.6% / 20.0% | MET |
| 4. Agreement with staff | ≥ 21 of 30 | not a per-seed measure | — | — | NOT TESTED (no staff labels; see Agreement Report) |
| 5. Confidence | most-confident third ≥ 0.80 and ≥ 0.15 above least-confident third | 0.917 vs 0.417 | 0.833 vs 0.500 | 0.750 vs 0.500 | MISSED |


**Did it help, and how do I know:** Yes. Accuracy rose on all three seeds (0.395 → 0.737, 0.342 → 0.763, 0.368 → 0.658, which is +0.342, +0.421 and +0.290) and macro F1 rose from 0.233, 0.127, 0.186 to 0.681, 0.744, 0.607. Criterion 1 went from MISSED to MET, and the collapse onto `question` is gone: the model now puts `reaction` and `hot_take` posts into their own columns. It did not fix criterion 2 (reaction F1 0.500 and 0.400 on two seeds) and criterion 5 flipped from MET to MISSED, but only because seed 2024's most-confident third is 9 of 12 correct (0.750), one post short of the 0.80 target. The new weakness is that accuracy now moves 0.105 across seeds, which is over the ten points where the notebook says the dataset is too small for a stable measure, and the big swing is `analysis` (F1 0.857 on seed 7 vs 0.421 on seed 2024).

**Supporting numbers (after improvement 1):**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.737 | 0.763 | 0.658 | 0.105 |
| Macro F1 | 0.681 | 0.744 | 0.607 | 0.137 |
| F1 — `analysis` | 0.762 | 0.857 | 0.421 | 0.436 |
| F1 — `hot_take` | 0.533 | 0.556 | 0.727 | 0.194 |
| F1 — `reaction` | 0.500 | 0.667 | 0.400 | 0.267 |
| F1 — `question` | 0.929 | 0.897 | 0.880 | 0.049 |

Confusion matrix, seed 42, 8 epochs, device **cpu**:

| true \ predicted | analysis | hot_take | reaction | question |
|---|---|---|---|---|
| **analysis** | 8 | 0 | 0 | 0 |
| **hot_take** | 2 | 4 | 1 | 2 |
| **reaction** | 3 | 2 | 3 | 0 |
| **question** | 0 | 0 | 0 | 13 |

### Improvement 2 (stretch): 30 more `reaction` posts

**What I changed:** Added 30 posts labelled `reaction` to `labels.csv` (250 → 280 rows), on top of improvement 1 (still 8 epochs). Epochs, learning rate, batch size and the three seeds are unchanged. Same labelling process as before: the posts came from the same Hacker News pull, were pre-labelled by an AI assistant against my definitions, and are marked `ai-prelabelled; improvement-2 reaction set` in the note column. `reaction` is now 80 of 280 (28.6%), `question` 89 (31.8%), `hot_take` 59 (21.1%), `analysis` 52 (18.6%): still inside criterion 3.

**Which diagnosis pointed at it:** After improvement 1, `reaction` had the worst and most unstable F1 (0.500, 0.667, 0.400) and it was the thinnest label in the file at 50 rows (8 test posts per split). Its misses go to `analysis` (3 of 8 on seed 42, 4 of 8 on seed 2024) and my agreement evidence says why this boundary is fragile: in my cold pass I disagreed with the AI pre-label on 10 of 20 rows, and 5 of the 10 were `reaction` vs `hot_take`. More examples is the change that addresses "too few examples", the one cause I can fix without editing labels.

### Run Log — After 2

Same table, same seeds, `results_three_seeds_after2.json`, device **cpu**. **Caveat:** adding rows changes the stratified split, so these are different test sets from the first two logs: **42 test posts per seed** (8 `analysis`, 9 `hot_take`, **12** `reaction`, 13 `question`) instead of 38. Before and after 2 are the same three seeds but not the same posts.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on every seed | 0.810 | 0.762 | 0.714 | MET |
| 2. Every label F1 | ≥ 0.55 on every seed | min 0.533 (`hot_take`) | min 0.588 (`hot_take`) | min 0.556 (`hot_take`) | MISSED |
| 3. Label balance | biggest ≤ 40%, smallest ≥ 15% | 31.8% / 18.6% | 31.8% / 18.6% | 31.8% / 18.6% | MET |
| 4. Agreement with staff | ≥ 21 of 30 | not a per-seed measure | — | — | NOT TESTED (no staff labels; see Agreement Report) |
| 5. Confidence | most-confident third ≥ 0.80 and ≥ 0.15 above least-confident third | 0.929 vs 0.571 | 0.857 vs 0.643 | 0.929 vs 0.500 | MET |


**Did it help, and how do I know:** Mostly yes, with a caveat. `reaction` F1 rose on every seed (0.500 → 0.833, 0.667 → 0.750, 0.400 → 0.720), which is the label I targeted, and the `reaction` → `analysis` confusion shrank from 3 (seed 42) to 1. Accuracy rose on seeds 42 and 2024 (0.737 → 0.810, 0.658 → 0.714) and was flat on seed 7 (0.763 → 0.762), macro F1 rose on two of three seeds (0.681 → 0.780, 0.607 → 0.695, and 0.744 → 0.737 on seed 7), and criterion 5 went back from MISSED to MET. It did not fix criterion 2, because `hot_take` is now the label under 0.55, by 0.017 on seed 42. The caveat is that the test sets changed (38 → 42 posts, 8 → 12 `reaction` posts), so part of the `reaction` gain could be an easier draw, not only more training data, and I can't separate the two with this design. Seed 7 not moving at all is a sign the gain isn't uniform.

**Supporting numbers (after improvement 2):**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.810 | 0.762 | 0.714 | 0.095 |
| Macro F1 | 0.780 | 0.737 | 0.695 | 0.085 |
| F1 — `analysis` | 0.824 | 0.714 | 0.625 | 0.199 |
| F1 — `hot_take` | 0.533 | 0.588 | 0.556 | 0.055 |
| F1 — `reaction` | 0.833 | 0.750 | 0.720 | 0.113 |
| F1 — `question` | 0.929 | 0.897 | 0.880 | 0.049 |

Confusion matrix, seed 42, 8 epochs, 280 rows, device **cpu**:

| true \ predicted | analysis | hot_take | reaction | question |
|---|---|---|---|---|
| **analysis** | 7 | 1 | 0 | 0 |
| **hot_take** | 1 | 4 | 2 | 2 |
| **reaction** | 1 | 1 | 10 | 0 |
| **question** | 0 | 0 | 0 | 13 |

---

## What's Still Broken

**Criterion 2 (every label F1 ≥ 0.55 on every seed): still MISSED.** After improvement 2 the only number under target is `hot_take` on seed 42 (0.533), and it is 0.017 short, but `hot_take` is also the label with the clearest pattern of errors (short or first-person posts called `reaction`, rhetorical questions called `question`). What I'd do next: relabel the `hot_take`/`reaction` and `hot_take`/`question` boundary once, applying the written rule to every row, then add `hot_take` examples (it's now the second-thinnest label at 59 rows). I stopped because this unit fixes labels and taxonomy in place, and the cold-pass disagreements (10 of 20) mean a relabelling pass would also have to decide whether my cold labels or my written rule is the one I believe.

**Criterion 4 (agreement ≥ 21/30): NOT TESTED.** The staff labels never reached me, so I can't call it. What I'd do: get `data/staff_labels.csv` from my TF and run `python agreement.py` on it; the stand-in against an AI-written reference (24 of 30, 80%) says I'd probably clear 21 but also shows the habit that matters, over-calling `reaction`. I stopped because the file is something only staff can send.

**Criterion 1 is MET but fragile.** After improvement 1 seed 2024 is 0.658, which is 25 correct of 38 posts: one fewer correct post is a miss. Accuracy moves 0.105 across the three seeds after improvement 1 and 0.095 after improvement 2, which is the size of the effect the notebook flags as too unstable to call a number near the line. With 38 to 42 test posts per split this dataset can't give a tighter measure. More labelled data, not more tuning, is the fix, and I stopped because 280 rows is what the project asks for.

**The gap between what I meant my labels to capture and what the model learned:** I meant `hot_take` to be a confident claim with nothing checkable behind it, and `reaction` to be a short feeling-or-social response that makes no claim, but the model learned surface cues instead: a question mark means `question`, a short first-person opening means `reaction`, and a named technical thing means `analysis`. The confusion matrices show it: `hot_take` is the only label whose true posts go to all three other labels (on seed 42 after improvement 2: 1 to `analysis`, 2 to `reaction`, 2 to `question`), while `question` and `analysis` stay almost clean, and my own cold labels disagreed with the written rule on exactly these boundaries.

---

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
