# Acceptance criteria — Provenance Guard

Five criteria that say what "working" means for this service, written in unit 7
**before** you built it.

**All five are yours.** None are given.

An acceptance criterion names a number or something plainly observable. *"The
detector is accurate"* is an opinion — and perfect AI detection isn't possible
anyway, so it's an opinion about something nobody can deliver. *"Of 10 pieces I
wrote myself, at most 1 is labelled high-confidence AI"* is a criterion.

Under each, write a sentence or two on **why that target**. *"I allow at most 1
false positive in 10 because on a writing site a wrong accusation costs more
than a missed detection"* is a real answer.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

**Cover at least three of these five areas.** They're prompts, not a form.

| Area | A question it could answer |
|---|---|
| False positives | How often may a human's writing get called AI? |
| Score spread | How far apart should clearly-AI and clearly-human text score? |
| Label coverage | Is every one of your three labels actually reachable? |
| The appeal path | Does an appeal always change status and always get recorded? |
| Bad input | What happens when `text` is missing, empty, or enormous? |

Two things worth knowing before you pick numbers:

- **The expensive mistake is one-sided.** Missing some AI text costs the site a
  little. Accusing a real writer costs that person a lot. A criterion that
  treats both errors as equally bad is describing a system nobody would ship.
- **Unit 8 tests across three trials, and the target has to hold across all
  three.** Your first signal runs a model, so identical input can score
  differently twice. A target of "at most 1 in 10" against trials of 2, 1, 2 is
  a **miss**.

---
## 1. False positives stay below 1 in 10 on human-authored text

Across 10 clearly human-written samples, at most 1 is labelled high-confidence AI or high-confidence human? no — the criterion is specifically that at most 1 of 10 human-authored samples receives a high-confidence AI label.

**Why this target:** I allow at most 1 false positive in 10 because on a writing site, accusing a real writer of using AI is a much more costly error than missing a few AI-generated submissions. The system should be conservative when it is making a potentially reputational accusation.

---

## 2. Clearly human and clearly AI texts must be separated by at least 0.35

On a set of 10 human-written and 10 AI-generated samples that I judge clearly one or the other, the average score gap between the two groups must be at least 0.35, and the lowest AI score must remain above the highest human score in at least 8 of 10 comparisons.

**Why this target:** A detector is not useful if the scores for obvious AI and obvious human writing blur together. A target of 0.35 gives a visible margin so the model can distinguish clear cases while still leaving room for uncertainty in borderline writing.

---

## 3. All three labels are reachable in the same trial run

In each of the three evaluation trials, the service must produce at least one high-confidence human label, at least one unsure label, and at least one high-confidence AI label across the test set.

**Why this target:** If one label is never reached, the system is hiding a class imbalance or a broken threshold. A three-way label set is only meaningful if all three outcomes are actually usable in practice, not just designed on paper.

---

## 4. Every appeal changes the decision and is recorded in the log

For every valid appeal request with a known content_id and non-empty reasoning, the response status changes from the original label to either a new label or an explicit unchanged-but-reviewed result, and the appeal is written to the audit log with the original decision, the appeal reason, and the updated decision.

**Why this target:** An appeal path that does not alter the state or does not leave evidence behind is not a real review process. Since a writer may challenge a mistaken classification, the system must show both an effect and a permanent record, otherwise people cannot meaningfully contest a result.

---

## 5. Bad input is rejected explicitly and never crashes the service

For text that is missing, empty, or larger than the configured maximum, the API must return a clear validation error (for example 400 or 422) rather than accepting the request or crashing. Specifically, I test 5 cases: missing text, empty string, whitespace-only string, a 1 MB text blob, and a 10 MB text blob; all must return a controlled failure with a response body explaining the issue.

**Why this target:** Real users do not always send good input, and a detector should degrade gracefully. A robust service needs to reject malformed or excessive submissions explicitly, because silent acceptance or a server crash is worse than a clean error when the system is being used on public-facing writing.

---
<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 8 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Uncertain cases land in the unsure band

         The system handles uncertain cases well.

         **Why this target:** ...

         > **Revised in unit 8:** Text I judged borderline scores between
         > 0.4 and 0.6 in at least 4 of 5 cases.
         >
         > **Why revised:** "well" gave me nothing to check. I couldn't
         > reach a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "At most 1 of 10 of my own pieces flagged as AI" → "at most 3
            of 10", because 1 of 10 turned out to be hard.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
