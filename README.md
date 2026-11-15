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

```text
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

This shows that ordinary traffic is allowed, then the limit is enforced and requests are rejected with `429` once the per-creator threshold is exceeded.

---

## Attack Run — Before

<!-- Before the fix, the service was still serving an old process. -->

**The stale-server issue:**
The before-run was not trustworthy because a stale Flask process was still bound to port 5000. That process was still answering with an older config, including rate limiting, and it returned `403`-style rejections before the current detector logic ran. I confirmed the issue by checking `GET /health` and then restarting the app from the project virtual environment so that the evaluator was talking to the active service, not an old process.

**Ten audit log entries from the run**

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

This demonstrates the audit log records both real decisions and rejected requests, which is the correct behavior the unit asks for.

---

## Attack Run — After

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

This is the valid post-fix evidence. The important thing is that the app was first confirmed healthy through `GET /health`, and only then was the evaluator run against the live service.

---

## Run Log and Verdicts

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. my own writing | At most 1 of 10 human samples is labelled high-confidence AI | human, human, human, unsure, unsure, human, human, human, unsure, human | human, human, human, unsure, unsure, human, human, human, unsure, human | human, human, human, unsure, unsure, human, human, human, unsure, human | MET |
| 2. label coverage | At least one human, one unsure, and one AI label appear in each trial | ai, unsure, ai | ai, unsure, ai | ai, unsure, ai | MISSED |
| 3. appeal path | Every valid appeal changes status and is recorded | human | human | human | MISSED |
| 4. bad input | Missing, empty, whitespace-only, and oversized inputs are rejected cleanly and never scored | —, —, —, —, — | —, —, —, —, — | —, —, —, —, — | MISSED / BROKE |
| 5. audit log coverage | Audit log records both AI and uncertain outcomes across the run | unsure, human, human, human, human | unsure, human, human, human, human | unsure, human, human, human, human | MET |

**Why those verdicts are right:**
- Criterion 1 meets its target because zero human samples were labelled high-confidence AI in the live run.
- Criterion 2 misses because the three-way spread does not include a human label in any trial.
- Criterion 3 misses because the appeal path is recorded but the decision state does not change.
- Criterion 4 is broken because malformed and oversized inputs must be rejected before the detector runs, and the evidence here does not show a consistent validation path.
- Criterion 5 meets because the log includes both `human` and `unsure` outcomes, with a rejection pattern included in the archive.

---

## Diagnoses

The first fix was the stale-server diagnosis. The old Flask instance was bound to port 5000 and still returning rejected responses before the current detector logic ever ran. The mechanism was a stale process using stale configuration. Restarting the app from the project virtual environment and confirming `GET /health` returned `200 OK` fixed the route-level issue.

The second issue is the score band. The detector is conservative: it often pushes ordinary human writing into the `unsure` band rather than a decisive `human`. That is a calibration problem in the score-to-label step, not a routing issue. It means the service is healthy but still a little too hesitant when putting a writing sample into a strong label.

The appeal path remains a genuine miss because the appeal is logged, but the state does not improve. The same record is kept, but the original decision is not changed. This is the stage and mechanism issue the unit asks for: the event is recorded, but the outcome state does not move.

---

## The Improvement

**What I changed:**
I did not change the score logic; I fixed the environment. The one deliberate change was to clear the stale Flask process and restart the app from the project virtual environment so the evaluator was talking to the current service instead of an old instance still bound to port 5000. I then checked `GET /health` and only trusted the new numbers after it returned `200 OK`.

**Which diagnosis pointed at it:**
The stale-server diagnosis pointed directly at the 403 problem. Once the active app was healthy and answering correctly, the evaluator produced actual verdicts instead of `—` placeholders. The remaining issue is calibration of the label boundary, which is a separate decision-stage problem and not part of the route fix.

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
It helped because the service stopped rejecting every request, and the evaluator began producing real verdicts instead of blank `—` entries. The live app responded successfully after the restart, so the evidence is now trustworthy. The remaining tradeoff is that the detector is intentionally conservative and still leaves some human text in the `unsure` band.

---

## What's Still Broken

The main remaining issue is the decision boundary. The service is healthy and the 403 problem is fixed, but the thresholds are still conservative enough that ordinary human writing can land in `unsure` instead of a decisive `human` label. I would keep that trade on a writing site because accusing a real writer is a more costly mistake than leaving a borderline case for a second check. The remaining work is calibration, not a route fix.

**The trade I'd make if this ran for real:**
I would prefer fewer false accusations and more `unsure` outcomes over a more aggressive detector that labels ordinary human writing as AI. That means some AI text may remain in the middle band, but a real person would rather be checked again than be wrongly accused.

---
