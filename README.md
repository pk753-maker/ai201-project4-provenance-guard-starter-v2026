# Provenance Guard

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first.**
>
> Once `python test.py` passes, in one terminal:
>
> ```bash
> python app.py
> ```
>
> and in another:
>
> ```bash
> curl -X POST http://127.0.0.1:5000/ping \
>   -H "Content-Type: application/json" \
>   -d '{"message": "hello"}'
> ```
>
> That's a web service. Something sent it text and it sent something back.
> Everything this unit is filling in the middle.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     Unit 7 asks for the first five sections. Unit 8 adds the five below.

     Everything is pasted as TEXT. A pasted curl response gets full credit;
     a picture of your terminal gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 7 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- The path a submission takes, from arriving to the label a reader sees.
     Six or seven steps — this is the trace you wrote in Milestone 1. -->

1. A caller sends `POST /submit` with JSON containing `text` and `creator_id`. The Flask route receives the request and validates that the payload is a JSON object with the required fields.
2. The app generates a fresh `content_id` and stores the submission metadata in memory for the request flow. This gives every decision a stable identifier that can be audited later and later used by `/appeal`.
3. The text is passed to `detector.model_signal(text)`, which runs the local GPT-2 model and converts its predictability into a 0–1 AI-likelihood score. This adds the model-based signal that measures how predictable the writing looks.
4. The same text is passed to `stylometry.style_signal(text)`, which computes sentence-length spread, vocabulary repetition, and punctuation density and combines them into a second 0–1 score. This adds a text-shape signal that measures the writing style rather than the meaning.
5. `scoring.combine_signals(model_score, style_score)` combines the two signals with a disagreement-aware weighted average. This step decides how much to trust each source when they agree or disagree.
6. The combined score is passed to `scoring.score_to_label(score)`, which maps it to one of the three reader-facing labels: high-confidence human, unsure, or high-confidence AI. This adds the human-readable verdict the writer sees.
7. The app writes the decision to `audit.log_decision(...)` and returns the JSON response with the `content_id`, both signal scores, the combined score, the confidence, and the final label. This adds a permanent audit record so later appeals and attack diagnostics can be traced back to the original submission.

---

## Signals and Scoring

<!-- For each signal: what property it measures, why that might differ between
     human and AI writing, and WHAT IT CAN'T SEE.

     The blind spot is not optional. If you can't name it you don't understand
     the signal yet — and unit 8's attack set is built out of blind spots. -->

### Signal one — the model signal

**What it measures:**
The model signal measures how predictable the text looks to a local GPT-2 language model. Lower surprise means the text looks like something the model itself could plausibly generate, so it scores higher on the AI side.

**Why that might differ between human and AI writing:**
Human writing contains more occasional oddity, personal phrasing, and abrupt jumps in topic or style. AI writing tends to be more statistically smooth and conventional, which makes it easier for a language model to predict.

**What it can't see:**
It cannot tell whether a given sentence is actually authored by a person or a model; it can only tell how predictable the wording is. Plain, simple human prose can look AI-like, and ornate, rare-word prose can look human-like even when generated.

### Signal two — the style signal

**What it measures:**
The style signal measures textual shape: how varied the sentence lengths are, how much the vocabulary repeats, and how much punctuation the writing uses.

**Why that might differ between human and AI writing:**
Human writing tends to vary more in sentence rhythm and vocabulary, while generated prose often uses more even sentence lengths and a narrower range of words. Punctuation is also often flatter in AI output.

**What it can't see:**
It cannot read the content, so it is blind to meaning. Adding typos, breaking a sentence up, or rewriting the same idea in a different shape can move the score without changing the substance.

### The combining rule

**The rule:**
I combine the two signals with a weighted average, but if they disagree by more than 0.35 I nudge the score toward the model signal. That keeps the style signal meaningful without letting a single weak style reading dominate when the model signal is more confident.

**The numbers:**
The weights live in `config.py` and are currently:
- `WEIGHT_MODEL_SIGNAL = 0.65`
- `WEIGHT_STYLE_SIGNAL = 0.35`

The rule is:
- base = 0.65 * model_score + 0.35 * style_score
- if abs(model_score - style_score) > 0.35, shift the result another 0.10 toward the model signal
- clamp the result to 0.0–1.0

**Where it lives:** `scoring.py::combine_signals`

**A case where my two signals split, and what my rule does with it:**
A formal human paragraph is a good split case: the model signal can still think it is ordinary and predictable, while the style signal notices dense phrasing and more elaborate sentence structure. In a real run, a formal-human example scored around `model = 0.3993` and `style = 0.7038`, which are far apart. My rule keeps the combined score in the unsure zone instead of flipping to a hard AI verdict, because the disagreement is meaningful and the model signal is given more influence.

---

## Label Variants

<!-- The exact text of all three labels, as a reader would see them, and the
     score range each covers.

     Write them for someone who has never heard the word "threshold". -->

| Label | Score range | The exact text a reader sees |
|---|---|---|
| high-confidence human | 0.00 – 0.35 | "We think this was probably written by a person." |
| unsure | 0.35 – 0.65 | "This text could be either human or AI; we would want a second check before making a strong claim." |
| high-confidence AI | 0.65 – 1.00 | "We think this was probably written by AI." |

**Why I worded the "unsure" one this way:**
It admits uncertainty without sounding like an accusation. A person may well have written every word themselves, so the message must leave room for doubt and invite a second look instead of making a claim about authorship.

---

## Sample Run

<!-- One submission and one appeal, pasted as text: the request, the response,
     and the log entries. -->

**A submission**

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "I went to the coffee shop and sat by the window while the rain tapped on the glass.", "creator_id": "alice"}'
```

```json
{
  "combined_score": 0.696,
  "confidence": 0.392,
  "content_id": "6c2a9f1a0e2044edbbdd7b79e894b1df",
  "guess": "ai",
  "label": "We think this was probably written by AI.",
  "model_score": 0.6894,
  "style_score": 0.7083
}
```

**An appeal**

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "6c2a9f1a0e2044edbbdd7b79e894b1df", "reasoning": "I wrote this and the score feels too high."}'
```

```json
{
  "content_id": "6c2a9f1a0e2044edbbdd7b79e894b1df",
  "message": "Appeal recorded and marked under review.",
  "ok": true,
  "status": "under_review"
}
```

**What the log shows afterwards**

```bash
$ curl "http://127.0.0.1:5000/log?limit=10"
```

```json
{
  "count": 3,
  "entries": [
    {
      "combined_score": null,
      "content_id": "ecef8198b52f411aa53ccb881fe83ff7",
      "creator_id": "alice",
      "guess": "human",
      "label": "pending",
      "model_score": 0.3063,
      "status": "decided",
      "style_score": 0.0,
      "timestamp": "2026-11-03T13:25:43+00:00"
    },
    {
      "combined_score": 0.696,
      "content_id": "6c2a9f1a0e2044edbbdd7b79e894b1df",
      "creator_id": "alice",
      "guess": "ai",
      "label": "We think this was probably written by AI.",
      "model_score": 0.6894,
      "status": "decided",
      "style_score": 0.7083,
      "timestamp": "2026-11-03T13:39:51+00:00"
    },
    {
      "content_id": "6c2a9f1a0e2044edbbdd7b79e894b1df",
      "creator_id": "alice",
      "event": "appeal",
      "reasoning": "I wrote this and the score feels too high.",
      "status": "under_review",
      "timestamp": "2026-11-03T13:39:51+00:00"
    }
  ]
}
```

---

## How I Used AI

**Moment 1**

- *What I asked for:*
I asked AI to evaluate my five criteria and see if any of the criteria needed changes or are they able to pass and be checked by someone who did not know what I originally meant.  
- *What came back:*
It said my criteria were fine but some details were vague enough that it would not pass in the way I would expect and some were too strict that it wouldn't pass at all. 
- *What I changed:*
Utilizing the AI's help, I rewrote my criteria to better fit the standards and make sure that my 5 criteria would have a work better

**Moment 2**

- *What I asked for:*
I asked the AI to tell me for each of my labels what I would think it is saying and what I would do next and give me suggestions. 
- *What came back:*
It gave me suggestions on my label and advice on what to change within my labels and what to keep 
- *What I changed:*
I changed the labels to match the AI's suggestions and kept asking it the same question and changing until it gave me an answer that I wanted 

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** ___ per minute, ___ per day

**Why those numbers and not others:**
<!-- Think about two people: a real writer submitting their own work a few
     times an hour, and a script sending a thousand variations to map your
     thresholds. Your numbers have to be liveable for the first and hostile to
     the second. -->

**What a caller counts as:** <!-- per address, or per creator_id? They fail
differently — one script can look like a thousand callers, and one household
can look like one. -->

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```

```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->



**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json

```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:



---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Real output from one trial**, pasted as text, naming the file and function
that produced it:

```json

```

**Diagnoses**

<!-- For each miss AND each broken attack: the stage and the mechanism.

     Your service has five stages: the route, signal one, signal two, the
     combining rule, and the label mapping.

     Not a diagnosis:  "The evasion attacks got through."
     A diagnosis:      "Four evasion inputs got through. All four had typos
                        added to AI-generated text. My second signal measures
                        word variety, and typos raise word variety — so it read
                        them as more human. My combining rule weights that
                        signal at 0.6, so it dragged the whole score down."

     Look for a pattern. Five failures on one signal is one problem, not five. -->



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Attack Run — After

<!-- Same format. `run_attacks.py --label after` -->



**Did it help, and how do I know:**

<!-- Widening the unsure band usually fixes one problem and creates another.
     Reporting that trade honestly is worth full credit. -->



---

## What's Still Broken

<!-- For each attack still getting through and each criterion still missed:
     what you'd do, and why you stopped. -->



**The trade I'd make if this ran for real:**

<!-- Two sentences. Fewer wrong accusations means more AI text getting through.
     Say which way you'd go and WHO PAYS FOR IT. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 7

       [ ] criteria.md has five numbered criteria, each naming a target
       [ ] Each has a reason underneath
       [ ] POST /submit returns content_id, guess, confidence, label,
           model_score and style_score
       [ ] Two signals that measure DIFFERENT things, each with a named blind spot
       [ ] Three label variants, all reachable, written out in full
       [ ] POST /appeal changes status and writes to the log
       [ ] Signals and Scoring names scoring.py::combine_signals
       [ ] Sample Run: a submission AND an appeal, as pasted text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 8

       [ ] Rate limiting on /submit, with your numbers justified
       [ ] The pasted run of status codes showing 429
       [ ] Audit log with all nine fields, recording rejections too
       [ ] The attack set run IN FULL, every input with an outcome
       [ ] Ten audit log entries pasted as text
       [ ] Five criteria across three trials
       [ ] A diagnosis for every miss and every broken attack — stage AND mechanism
       [ ] One improvement, with Attack Run — After
       [ ] What's Still Broken, including the production trade
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**

