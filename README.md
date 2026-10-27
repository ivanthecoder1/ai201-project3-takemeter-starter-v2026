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

TakeMeter is a text classifier that analyzes public posts from the r/gaming subreddit. It sorts posts into two categories: commentary and content_share. The commentary label is for posts that include meaningful personal opinions, explanations, analysis, or experiences, while content_share is for posts that mainly share gaming-related news, trailers, announcements, images, or links with little additional commentary. The goal is to distinguish posts that add the author's perspective from posts that primarily share content.

---

## Label Taxonomy

### `commentary`

**Definition:** Posts where the author adds their own thoughts, opinions, analysis, or personal experience about a gaming-related topic, beyond simply sharing content.

**Example 1:**
> I want to get my wife into gaming but I don't know how
>
> My wife told me she wants to try my hobbies and I let her. Unfortunately, most of my games are either 1st or 3rd person. She had a hard time navigating, I tried teaching her but it's difficult since she's trying for the first time. Any recommendations on the best games to try first for those who had this small problem?

**Example 2:**
> Older gamers, what was the first video game you remember making your jaw drop, visually??
>
> We were a intellivision family when I was real young and they were nice, but you really had to stretch your imagination.
> For me, Castlevania on the NES was just, jaw dropping.
> (Same with Castlevania IV on the SNES which was the first SNES game I ever tried)
> Sure, it was so hard I didn't get too far but it looked amazing !

### `content_share`

**Definition:** Posts that primarily share gaming-related content, such as trailers, announcements, news, screenshots, or links, with little or no additional explanation or personal commentary from the author.

**Example 1:**
> Grand Theft Auto 6 Launch Trailer

**Example 2:**
> Crimson Desert Enhanced: Charting the Unknown features trailer
>
> Some of the bigger things in the trailer:
>
> They appear to have added black flag style ship combat and underwater exploration but with the ability to use various mounts underwater like a submersible or hitching a ride on a killer whale. You can also design the ship itself.
>
> There's a sims-like building editor where you can design your own place or business.
>
> You can now micromanage the businesses you set up at captured outposts. Design the shop layout and also what to sell and for how much.
>
> You can now marry various NPCs and this apparently has an effect on your greymane camp
>
> There's now an area where you can do a boss rush of bosses you've cleared already for various rewards.

### The hardest boundary

**Which two labels:** `commentary` vs. `content_share`

**The decision rule I used every time:** If the author adds a meaningful personal opinion, explanation, or analysis beyond the shared content, label the post `commentary`. If the post mainly shares information or media without adding a meaningful perspective, label it `content_share`. A short title or a basic description of the shared content does not count as commentary by itself.


## The Dataset

**Where the posts came from:** r/gaming, a public subreddit, with no login needed. The first 20 posts were copied by hand from the hot feed. The rest came from a CSV export of the new feed, a scrape tool, covering Oct 19–21, 2026. Each row is the post title plus body text. I skipped image-only posts whose titles meant nothing without the image, automated weekly mod threads, and a few very long article reposts.

**How I labelled them:** I labelled the first 20 posts cold, with no tools or help. They are marked `cold` in the note column. The remaining rows were pre-labelled by Claude from my definitions and decision rule, and are marked `pre-labeled`. I then read every row and corrected 10 of them. 

**Counts per label:** [update if you add rows or relabel]

| Label | Count | Share |
|---|---|---|
| commentary | 87 | 62.6% |
| content_share | 52 | 37.4% |
| **Total** | 139 | 100% |

I have 139 rows rather than 200 because of time constraints.

**Three hard cases**

**1.**
> *The post:* Crimson Desert Enhanced: Charting the Unknown features trailer. Some of the bigger things in the trailer: They appear to have added black flag style ship combat and underwater exploration […] There's now an area where you can do a boss rush of bosses you've cleared already for various rewards.
>
> *Could have been:* commentary
>
> *I chose, because:* content_share. It is long and written by the poster, which makes it look like commentary. But it only summarizes what the trailer shows, with no opinion, feeling, reason or experience from the author. Under my rule that is content_share.

**2.**
> *The post:* A new report, which interviewed 101 leaders from studios across North America and Europe, found that 94% said their studios use AI at some stage in development but more than half said it isn't deeply embedded into their production pipeline. Of note: This research was done by VAIA, who appear overtly pro-AI in their report.
>
> *Could have been:* content_share
>
> *I chose, because:* commentary. The title is a news statistic, but the poster's "Of note" line adds their own critical judgment of the source. 

**3.**
> *The post:* The Finals - Season 12 | Prime Time. The Finals just Updated to Season 12 Today! It is probably the only FPS I play these days.
>
> *Could have been:* content_share
>
> *I chose, because:* commentary. The post is a trailer link, but the poster states a personal experience in their own words ("the only FPS I play these days"). My rule says that makes it commentary even when a trailer is attached. 

---

## The Training Run

**Base model:** 2.14.1+cpu

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42. Trained on CPU (torch 2.14.1+cpu).

**Anything I changed from the defaults, and why:** Nothing. I used the defaults.

**Split sizes:** [update if you rerun] 97 train / 21 val / 21 test (70/15/15, stratified). Test split: 13 commentary, 8 content_share. `content_share` has 8 test examples, which is thin, so its metrics will move a lot between seeds.

---

## How I Used AI

**Moment 1: pre-labelling the scraped posts**

- *What I asked for:* I gave Claude my two label definitions and decision rule, plus a CSV export of r/gaming posts (title and body), and asked it to pre-label them in `text,label,note` format.
- *What came back:* 120 labelled rows, with 9 flagged as hard cases. Claude also flagged judgment calls, such as pure questions going to `commentary` and short opinion-bearing titles going to `commentary`.
- *What I changed:* I relabelled about 20 rows

**Moment 2: cleaning, formatting and consistency checks**

- *What I asked for:* Help turning pasted Reddit posts into a clean CSV, and a check on my criteria and labels.
- *What came back:* Cleaned rows with the page chrome removed, line breaks flattened and quotes escaped. Claude also pointed out that my labels didn't always follow my own decision rule (The Finals, Dragon Quest) and that my criteria had a weak baseline argument.
- *What I changed:* I relabelled The Finals, rewrote criteria 4 and 5 so a stranger could check them

**Pre-labelling disclosure:** About 130 rows were pre-labelled by Claude and are marked `pre-labeled` in the note column. I labelled the 20 `cold` rows myself with no help. I read and corrected every pre-labelled row.


**Pre-labelling disclosure:** About 119 rows were pre-labelled by Claude and are marked `pre-labeled` in the note column. I labelled the 20 `cold` rows myself with no help. I read and corrected every pre-labelled row.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.619 | 0.619 | +0.000 |
| Macro F1 | 0.382 | 0.382 | -0.000 |
| F1 — `commentary` | 0.765 | 0.765 | -0.000 |
| F1 — `content_share` | 0.000 | 0.000 | +0.000 |

**What I predicted before I looked: my expectation was that the trained model would beat the baseline on `content_share`. It did not.**


**What the gap actually means: There is no gap. Both models predicted `commentary` for all 21 test posts, so both score 13/21 = 0.619 accuracy and 0.000 F1 on `content_share`. Fine-tuning on my rows added nothing over a general model on this test set.**

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
