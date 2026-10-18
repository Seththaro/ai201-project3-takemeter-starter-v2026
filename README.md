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
