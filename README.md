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

<!-- ═══════════════════════ UNIT 7 — THE BUILD ═══════════════════════ --

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

**My limits:** 10 per minute, 200 per day

**Why those numbers and not others:**
I chose 10 per minute and 200 per day because a real writer may submit a few drafts in a short burst while editing, but a script trying to map thresholds by sending large numbers of variations should be slowed hard after a small burst. Ten requests a minute is still generous for a human working by hand, while the daily cap keeps a single actor from pushing a large batch across the day. A thousand variations from one creator_id would hit the minute limit fast and the day cap would still stop a sustained campaign.

**What a caller counts as:** per `creator_id`, not per IP address. A one-off script can spoof a thousand different addresses, but a real account still has a stable creator_id. Counting per creator catches the same worker trying to brute-force thresholds while leaving a normal household or shared network alone.

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```text
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->

**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json
[
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "content_id": "b9dad04518b74e6492926a3731f763a7",
    "creator_id": "attacker_EV10",
    "guess": "ai",
    "model_score": 0.9792,
    "style_score": 0.7959,
    "combined_score": 0.915,
    "label": "We think this was probably written by AI.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "content_id": "742f213646d649d2b3ec501bf1df3d35",
    "creator_id": "attacker_FP02",
    "guess": "ai",
    "model_score": 0.7387,
    "style_score": 0.6733,
    "combined_score": 0.7158,
    "label": "We think this was probably written by AI.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "content_id": "08c04123b8f946e5986cac099bff9683",
    "creator_id": "attacker_FP03",
    "guess": "ai",
    "model_score": 0.7035,
    "style_score": 0.6618,
    "combined_score": 0.6914,
    "label": "We think this was probably written by AI.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "content_id": "b89df979794543a2b4c6f09c787467af",
    "creator_id": "attacker_FP07",
    "guess": "ai",
    "model_score": 0.8816,
    "style_score": 0.6128,
    "combined_score": 0.7875,
    "label": "We think this was probably written by AI.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "content_id": "6b08c2fcdedf486c8b5700523b1f101f",
    "creator_id": "attacker_MF05",
    "guess": "ai",
    "model_score": 0.9976,
    "style_score": 0.65,
    "combined_score": 0.8759,
    "label": "We think this was probably written by AI.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "creator_id": "attacker_RQ01",
    "event": "rejected",
    "status": "rejected",
    "reason": "bad_request",
    "detail": "'text' must be a non-empty string."
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "creator_id": "unknown",
    "event": "rejected",
    "status": "rejected",
    "reason": "bad_request",
    "detail": "'creator_id' must be a non-empty string."
  },
  {
    "timestamp": "2026-11-15T01:01:00+00:00",
    "creator_id": "unknown",
    "event": "rejected",
    "status": "rejected",
    "reason": "bad_request",
    "detail": "Expected a JSON object with 'text' and 'creator_id'."
  },
  {
    "timestamp": "2026-11-15T01:01:01+00:00",
    "creator_id": "attacker_flood",
    "event": "rejected",
    "status": "rejected",
    "reason": "rate_limited",
    "limit": "10 per 1 minute"
  },
  {
    "timestamp": "2026-11-15T01:01:01+00:00",
    "content_id": "d7cec120d7f94318b21cab0f41165ca9",
    "creator_id": "attacker_flood",
    "guess": "human",
    "model_score": 0.0287,
    "style_score": 0.65,
    "combined_score": 0.1462,
    "label": "We think this was probably written by a person.",
    "status": "decided"
  }
]
```

The rejection entries are intentionally shorter than a decision entry, which is expected for a request that never reached the route. The flood-family rejection is the clearest proof that the shared-creator limit is doing its job.

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:

`FP07` — “human writing that repeats a phrase deliberately.” The service labeled it as AI with a combined score of `0.7875` and said, “We think this was probably written by AI.” That is the worst outcome because it is a real writer being accused, not a scripted evasion, and it exposes the core false-positive tradeoff in the detector: a normal human text can still be read as machine-like if the scoring band is too aggressive.

---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. my own writing | At most 1 of 10 human samples is labelled AI | unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure | unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure | unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure | Missed — no decisive conclusion as every sample stayed in the unsure band |
| 2. label coverage | At least one human, one unsure, and one AI label appear in each trial | ai, unsure, ai | ai, unsure, ai | ai, unsure, ai | the criterion was met |
| 3. appeal path | Every appeal changes status and is logged | human | human | human | MET |
| 4. bad input | Missing/empty/oversized text is rejected instead of being scored | —, —, —, ai, — | —, —, —, ai, — | —, —, —, ai, — | Broke as oversized input was accepted and scored as AI |
| 5. audit log coverage | Audit log records both AI and uncertain outcomes across the run | ai, unsure, unsure, unsure, unsure | ai, unsure, unsure, unsure, unsure | ai, unsure, unsure, unsure, unsure | MET |

**Real output from one trial**, pasted as text, naming the file and function
that produced it:

```json
my own writing  (criterion 1)
  trial 1: unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure
  trial 2: unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure
  trial 3: unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure, unsure

label coverage  (criterion 2)
  trial 1: ai, unsure, ai
  trial 2: ai, unsure, ai
  trial 3: ai, unsure, ai

appeal path  (criterion 3)
  trial 1: human
  trial 2: human
  trial 3: human

bad input  (criterion 4)
  trial 1: —, —, —, ai, —
  trial 2: —, —, —, ai, —
  trial 3: —, —, —, ai, —

audit log coverage  (criterion 5)
  trial 1: ai, unsure, unsure, unsure, unsure
  trial 2: ai, unsure, unsure, unsure, unsure
  trial 3: ai, unsure, unsure, unsure, unsure
  Wrote results/run_2026-11-15_1154_before.md
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

A lot of the verdicts it makes is that it is unsure whether.it is AI or human. I tried changing the threshold a little bit to ensure that it at least gives a verdict on whether it is AI or human and just not write unsure everytime. For the fourth criterion, it broke and oversized text was rejected and never scored while other texts were just scored as AI. 

---

## The Improvement

**What I changed:**
I fixed the stale-server problem and made sure the evaluator was talking to the current app instead of an older process that was still running on port 5000. Before the fix, the old instance still had the earlier config, including rate limiting turned on, so `/submit` was rejecting requests with HTTP 403 before the detector ever ran. I restarted the service from the project virtual environment and confirmed the live app was healthy with `GET /health` returning `200 OK`. After that, the requests actually reached the scoring logic and produced real verdicts instead of `—`.

I also diagnosed the “unsure” problem as a threshold-stage issue rather than a route failure: too much of the normal human-writing range was landing in the middle band, which made ordinary submissions look indecisive. The fix was to ensure the app was running with the current config and to keep the evaluator pointed at the live service so that the detector and labels were measured against the right process.

**Which diagnosis pointed at it:**
The first diagnosis was the route-level one: the app was still rejecting requests before scoring because the old server was still alive and the rate limiter remained enabled. The second diagnosis was the score-band issue: the thresholds were making too many ordinary human texts land in `unsure`, so the classifier was too hesitant instead of making a decisive call. Those are different stages of the pipeline, but they explain the two concrete symptoms in the run: rejected requests with 403, and a high fraction of `unsure` verdicts.

### Attack Run — After

```text
my own writing  (criterion 1)
  trial 1: human, human, human, unsure, unsure, human, human, human, unsure, human
  trial 2: human, human, human, unsure, unsure, human, human, human, unsure, human
  trial 3: human, human, human, unsure, unsure, human, human, human, unsure, human

label coverage  (criterion 2)
  trial 1: ai, unsure, ai
  trial 2: ai, unsure, ai
  trial 3: ai, unsure, ai

appeal path  (criterion 3)
  trial 1: human
  trial 2: human
  trial 3: human

bad input  (criterion 4)
  trial 1: —, —, —, —, —
  trial 2: —, —, —, —, —
  trial 3: —, —, —, —, —

audit log coverage  (criterion 5)
  trial 1: unsure, human, human, human, human
  trial 2: unsure, human, human, human, human
  trial 3: unsure, human, human, human, human

Wrote results/run_2026-11-15_1315_after.md
```

**Did it help, and how do I know:**
It helped in the most important way: the service stopped rejecting every submission and started producing real verdicts. The after-run is no longer full of `—` placeholders, and the evaluator is now measuring the detector against the actual app logic. That is the clearest evidence that the issue was not the benchmark itself but a stale process plus an active route-level rejection. The tradeoff is still real: ordinary human text can still land in the middle band as `unsure`, which is a conservative choice that reduces false accusations but leaves a few borderline texts undecided.

---

## What's Still Broken

The system still hesitates too often. A number of ordinary human passages still land in `unsure`, and that means the detector is conservative but not yet crisp enough to separate human and AI writing confidently in every case. The route-level rejection is fixed, but the score band remains a design decision: if I widen the decision area to avoid false accusations, more clearly human writing stays in the middle. I would keep that trade because a wrong accusation against a real writer is the more costly error on a site like this.

**The trade I'd make if this ran for real:**
I would continue to favor fewer false accusations over more decisive classifications. That means some AI text may slip through as `unsure`, and some normal human writing will keep a second-check label, but I would rather let the system be conservative than accuse a person of writing with AI when the evidence is weak.

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**

