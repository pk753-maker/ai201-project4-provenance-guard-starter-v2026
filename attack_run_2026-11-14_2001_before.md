# Attack run — before

- Produced by: `run_attacks.py::main`
- Service: `app.py::submit`
- 56 attacks · 2026-11-14 20:01
- Auto-marked BROKE (5xx, crash, no response): **1**
- Left for you to judge: **55**

The Verdict column is blank where a script can't honestly decide. Whether
an evasion worked depends on where you put your thresholds — read the
`targets` column, look at what came back, and write HELD or BROKE yourself.

Each attack submits as creator_id `attacker_<id>`, and the content_id it
came back with is in the table — either one will match this run against your
audit log when you go looking for the ten entries to paste.

| ID | Family | Targeting | Status | Content ID | Guess | Score | Verdict |
|---|---|---|---|---|---|---|---|
| EV01 | evasion | signal two: typos raise word variety | 200 | `0dbff26d31454fbf862ed820ade6f581` | human | 0.582 |  |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 200 | `42034fb284ab4e22b7b37a82f4ab6356` | unsure | 0.081 |  |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | `8b135172457c4f889c8294bedf0cec50` | unsure | 0.222 |  |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | `423540c751cd44c7919d2cc290888f38` | unsure | 0.185 |  |
| EV05 | evasion | both signals: heavy human editing pass | 200 | `95760144e97e481d9aa8cfbcfafaa1cf` | unsure | 0.281 |  |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | `55730d81e2214c7b9f9f1c50646f1ca2` | unsure | 0.029 |  |
| EV07 | evasion | signal two: padding with punctuation | 200 | `f605c37fa6fd409b8276177ba78046f8` | unsure | 0.063 |  |
| EV08 | evasion | signal one: text below the token window | 200 | `2d44ece028a04cf98f825ddac7464f85` | unsure | 0.272 |  |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | `d38002325e28455a9431810c7d43741f` | unsure | 0.269 |  |
| EV10 | evasion | signal two: repeated stanza structure | 200 | `b9dad04518b74e6492926a3731f763a7` | ai | 0.830 |  |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | `e1445d4af4274987ad16057239ba805d` | unsure | 0.232 |  |
| FP02 | false_positive | human writing in a formal register | 200 | `742f213646d649d2b3ec501bf1df3d35` | ai | 0.432 |  |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | `08c04123b8f946e5986cac099bff9683` | ai | 0.516 |  |
| FP04 | false_positive | a student essay written to a rubric | 200 | `9d23a88c73b34ebc9a07cfb9a8fd19a9` | ai | 0.508 |  |
| FP05 | false_positive | technical documentation written by a person | 200 | `50cb3c14e9fd4284ad16c8c057b96815` | ai | 0.519 |  |
| FP06 | false_positive | a person imitating a formal template | 200 | `92d2a940ab5c446a9976803777647499` | ai | 0.563 |  |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | `b89df979794543a2b4c6f09c787467af` | ai | 0.575 |  |
| FP08 | false_positive | very short genuine human submission | 200 | `9979c999e0264780934950792948b462` | unsure | 0.197 |  |
| PI01 | prompt_injection | instruction to ignore the check | 200 | `1127c3cddef54c058faddb17eb77387a` | unsure | 0.036 |  |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | `b92cba7e9e394b69b1ead3f58adf4fe7` | human | 0.427 |  |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | `c225e8750a884cd09869d948fa84df23` | unsure | 0.173 |  |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | `932dd892a84a4ba0b1a7a64740ae8e90` | unsure | 0.028 |  |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | `2b5561ddec4d4ae7aeab56c0f0fe6a23` | unsure | 0.208 |  |
| PI06 | prompt_injection | instruction plus real human text | 200 | `0fa71099849040398db22e4ca9174b5d` | unsure | 0.159 |  |
| MF01 | malformed | text is only whitespace | 400 | `—` | — | — |  |
| MF02 | malformed | text is a single character | 500 | `—` | — | — | BROKE |
| MF03 | malformed | text is control characters | 200 | `cd8e1c1341d64e5f930a363008478575` | human | 0.745 |  |
| MF04 | malformed | text is only punctuation | 200 | `f179bb49f97849f5b670e11e54d4375b` | human | 0.485 |  |
| MF05 | malformed | text is a very long single word | 200 | `6b08c2fcdedf486c8b5700523b1f101f` | ai | 0.752 |  |
| MF06 | malformed | text is emoji only | 200 | `dea7c88c3ce3419a86adf97bc97fdb97` | ai | 0.996 |  |
| MF07 | malformed | text is right-to-left script | 200 | `37a10d43b11942de8175f09dfb99776e` | ai | 0.878 |  |
| MF08 | malformed | text is 60,000 characters | 200 | `8b076d73d34f4e728c7cd794e7e4baa2` | ai | 0.890 |  |
| RQ01 | malformed | no text field at all | 400 | `—` | — | — |  |
| RQ02 | malformed | no creator_id | 400 | `—` | — | — |  |
| RQ03 | malformed | text is a number | 400 | `—` | — | — |  |
| RQ04 | malformed | text is a list | 400 | `—` | — | — |  |
| RQ05 | malformed | text is null | 400 | `—` | — | — |  |
| RQ06 | malformed | creator_id is an object | 400 | `—` | — | — |  |
| RQ07 | malformed | body is a JSON array, not an object | 400 | `—` | — | — |  |
| RQ08 | malformed | body is unparseable JSON | 400 | `—` | — | — |  |
| RQ09 | malformed | body is empty | 400 | `—` | — | — |  |
| RQ10 | malformed | body is form-encoded, not JSON | 400 | `—` | — | — |  |
| RQ11 | malformed | body is JSON claiming to be plain text | 400 | `—` | — | — |  |
| RQ12 | malformed | deeply nested JSON | 400 | `—` | — | — |  |
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 200 | `a5c6c1748b0e4fa1bb85b91ff416346c` | human | 0.708 |  |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 200 | `8c70df920a624a6495388914ad52b61a` | human | 0.708 |  |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 200 | `d16c302e39004774a3bfa24cf6f459b6` | human | 0.708 |  |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 200 | `dedd9d10dae0450fadf20ac097e29b63` | human | 0.708 |  |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 200 | `f2cf4c4b297341be821a1c4ffd5816c4` | human | 0.708 |  |
| FL01_06 | flood_same_creator | 20 rapid submissions from one creator | 200 | `16dc0746975c4981930c7a3e9c9f09b6` | human | 0.708 |  |
| FL01_07 | flood_same_creator | 20 rapid submissions from one creator | 200 | `8a83f4a475734f28bd2400be82225cc5` | human | 0.708 |  |
| FL01_08 | flood_same_creator | 20 rapid submissions from one creator | 200 | `94d7764fa67e467495c1d41b91cca515` | human | 0.708 |  |
| FL01_09 | flood_same_creator | 20 rapid submissions from one creator | 200 | `0a068d234d034341a7d59f6e2f80ad75` | human | 0.708 |  |
| FL01_10 | flood_same_creator | 20 rapid submissions from one creator | 200 | `d7cec120d7f94318b21cab0f41165ca9` | human | 0.708 |  |
| FL01_11 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — |  |
| FL01_12 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — |  |

---

## By family

| Family | Attacks | Auto-BROKE | Left to judge |
|---|---|---|---|
| `evasion` | 10 | 0 | 10 |
| `false_positive` | 8 | 0 | 8 |
| `flood_same_creator` | 12 | 0 | 12 |
| `malformed` | 20 | 1 | 19 |
| `prompt_injection` | 6 | 0 | 6 |

> The **false_positive** family is the one to look at hardest. Those are
> written to be genuine human writing that reads as machine-like. Every one
> your service calls AI is a real writer it would have accused.

---

## What came back

The detail behind the table. Paste the interesting failures into your
README — the ones that show something, not the ten easiest.

### EV01 — `evasion`

*Targeting:* signal two: typos raise word variety

```json
{
  "id": "EV01",
  "status": 200,
  "content_id": "0dbff26d31454fbf862ed820ade6f581",
  "creator_id": "attacker_EV01",
  "guess": "human",
  "confidence": 0.582,
  "model_score": 0.1635,
  "style_score": 0.5793,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### EV02 — `evasion`

*Targeting:* signal two: broken-up sentences raise length spread

```json
{
  "id": "EV02",
  "status": 200,
  "content_id": "42034fb284ab4e22b7b37a82f4ab6356",
  "creator_id": "attacker_EV02",
  "guess": "unsure",
  "confidence": 0.0806,
  "model_score": 0.5943,
  "style_score": 0.44,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV03 — `evasion`

*Targeting:* signal two: a pasted quotation changes the shape

```json
{
  "id": "EV03",
  "status": 200,
  "content_id": "8b135172457c4f889c8294bedf0cec50",
  "creator_id": "attacker_EV03",
  "guess": "unsure",
  "confidence": 0.2218,
  "model_score": 0.6662,
  "style_score": 0.5083,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV04 — `evasion`

*Targeting:* signal one: unusual words raise perplexity

```json
{
  "id": "EV04",
  "status": 200,
  "content_id": "423540c751cd44c7919d2cc290888f38",
  "creator_id": "attacker_EV04",
  "guess": "unsure",
  "confidence": 0.185,
  "model_score": 0.6114,
  "style_score": 0.5573,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV05 — `evasion`

*Targeting:* both signals: heavy human editing pass

```json
{
  "id": "EV05",
  "status": 200,
  "content_id": "95760144e97e481d9aa8cfbcfafaa1cf",
  "creator_id": "attacker_EV05",
  "guess": "unsure",
  "confidence": 0.2806,
  "model_score": 0.3212,
  "style_score": 0.4312,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV06 — `evasion`

*Targeting:* signal two: one very long sentence and one very short

```json
{
  "id": "EV06",
  "status": 200,
  "content_id": "55730d81e2214c7b9f9f1c50646f1ca2",
  "creator_id": "attacker_EV06",
  "guess": "unsure",
  "confidence": 0.029,
  "model_score": 0.5727,
  "style_score": 0.4065,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV07 — `evasion`

*Targeting:* signal two: padding with punctuation

```json
{
  "id": "EV07",
  "status": 200,
  "content_id": "f605c37fa6fd409b8276177ba78046f8",
  "creator_id": "attacker_EV07",
  "guess": "unsure",
  "confidence": 0.0626,
  "model_score": 0.5862,
  "style_score": 0.4294,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV08 — `evasion`

*Targeting:* signal one: text below the token window

```json
{
  "id": "EV08",
  "status": 200,
  "content_id": "2d44ece028a04cf98f825ddac7464f85",
  "creator_id": "attacker_EV08",
  "guess": "unsure",
  "confidence": 0.2724,
  "model_score": 0.2452,
  "style_score": 0.584,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV09 — `evasion`

*Targeting:* both: AI text with a human opening paragraph bolted on

```json
{
  "id": "EV09",
  "status": 200,
  "content_id": "d38002325e28455a9431810c7d43741f",
  "creator_id": "attacker_EV09",
  "guess": "unsure",
  "confidence": 0.2686,
  "model_score": 0.7042,
  "style_score": 0.5044,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### EV10 — `evasion`

*Targeting:* signal two: repeated stanza structure

```json
{
  "id": "EV10",
  "status": 200,
  "content_id": "b9dad04518b74e6492926a3731f763a7",
  "creator_id": "attacker_EV10",
  "guess": "ai",
  "confidence": 0.83,
  "model_score": 0.9792,
  "style_score": 0.7959,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP01 — `false_positive`

*Targeting:* plain, clear human writing with no flourishes

```json
{
  "id": "FP01",
  "status": 200,
  "content_id": "e1445d4af4274987ad16057239ba805d",
  "creator_id": "attacker_FP01",
  "guess": "unsure",
  "confidence": 0.2318,
  "model_score": 0.6175,
  "style_score": 0.6129,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### FP02 — `false_positive`

*Targeting:* human writing in a formal register

```json
{
  "id": "FP02",
  "status": 200,
  "content_id": "742f213646d649d2b3ec501bf1df3d35",
  "creator_id": "attacker_FP02",
  "guess": "ai",
  "confidence": 0.4316,
  "model_score": 0.7387,
  "style_score": 0.6733,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP03 — `false_positive`

*Targeting:* a non-native English speaker writing carefully

```json
{
  "id": "FP03",
  "status": 200,
  "content_id": "08c04123b8f946e5986cac099bff9683",
  "creator_id": "attacker_FP03",
  "guess": "ai",
  "confidence": 0.516,
  "model_score": 0.8051,
  "style_score": 0.6704,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP04 — `false_positive`

*Targeting:* a student essay written to a rubric

```json
{
  "id": "FP04",
  "status": 200,
  "content_id": "9d23a88c73b34ebc9a07cfb9a8fd19a9",
  "creator_id": "attacker_FP04",
  "guess": "ai",
  "confidence": 0.5076,
  "model_score": 0.8329,
  "style_score": 0.6069,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP05 — `false_positive`

*Targeting:* technical documentation written by a person

```json
{
  "id": "FP05",
  "status": 200,
  "content_id": "50cb3c14e9fd4284ad16c8c057b96815",
  "creator_id": "attacker_FP05",
  "guess": "ai",
  "confidence": 0.5186,
  "model_score": 0.8005,
  "style_score": 0.6827,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP06 — `false_positive`

*Targeting:* a person imitating a formal template

```json
{
  "id": "FP06",
  "status": 200,
  "content_id": "92d2a940ab5c446a9976803777647499",
  "creator_id": "attacker_FP06",
  "guess": "ai",
  "confidence": 0.5628,
  "model_score": 0.8814,
  "style_score": 0.5958,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP07 — `false_positive`

*Targeting:* human writing that repeats a phrase deliberately

```json
{
  "id": "FP07",
  "status": 200,
  "content_id": "b89df979794543a2b4c6f09c787467af",
  "creator_id": "attacker_FP07",
  "guess": "ai",
  "confidence": 0.575,
  "model_score": 0.8816,
  "style_score": 0.6128,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### FP08 — `false_positive`

*Targeting:* very short genuine human submission

```json
{
  "id": "FP08",
  "status": 200,
  "content_id": "9979c999e0264780934950792948b462",
  "creator_id": "attacker_FP08",
  "guess": "unsure",
  "confidence": 0.1968,
  "model_score": 0.3223,
  "style_score": 0.549,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### PI01 — `prompt_injection`

*Targeting:* instruction to ignore the check

```json
{
  "id": "PI01",
  "status": 200,
  "content_id": "1127c3cddef54c058faddb17eb77387a",
  "creator_id": "attacker_PI01",
  "guess": "unsure",
  "confidence": 0.0356,
  "model_score": 0.4625,
  "style_score": 0.5187,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### PI02 — `prompt_injection`

*Targeting:* instruction disguised as a system message

```json
{
  "id": "PI02",
  "status": 200,
  "content_id": "b92cba7e9e394b69b1ead3f58adf4fe7",
  "creator_id": "attacker_PI02",
  "guess": "human",
  "confidence": 0.4266,
  "model_score": 0.2044,
  "style_score": 0.4395,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### PI03 — `prompt_injection`

*Targeting:* instruction wrapped in the writer's voice

```json
{
  "id": "PI03",
  "status": 200,
  "content_id": "c225e8750a884cd09869d948fa84df23",
  "creator_id": "attacker_PI03",
  "guess": "unsure",
  "confidence": 0.1726,
  "model_score": 0.5943,
  "style_score": 0.5714,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### PI04 — `prompt_injection`

*Targeting:* attempt to make the label text hostile

```json
{
  "id": "PI04",
  "status": 200,
  "content_id": "932dd892a84a4ba0b1a7a64740ae8e90",
  "creator_id": "attacker_PI04",
  "guess": "unsure",
  "confidence": 0.0278,
  "model_score": 0.4998,
  "style_score": 0.5402,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### PI05 — `prompt_injection`

*Targeting:* attempt to inject into the audit log

```json
{
  "id": "PI05",
  "status": 200,
  "content_id": "2b5561ddec4d4ae7aeab56c0f0fe6a23",
  "creator_id": "attacker_PI05",
  "guess": "unsure",
  "confidence": 0.2084,
  "model_score": 0.6401,
  "style_score": 0.2517,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### PI06 — `prompt_injection`

*Targeting:* instruction plus real human text

```json
{
  "id": "PI06",
  "status": 200,
  "content_id": "0fa71099849040398db22e4ca9174b5d",
  "creator_id": "attacker_PI06",
  "guess": "unsure",
  "confidence": 0.159,
  "model_score": 0.5583,
  "style_score": 0.6188,
  "label": "This text could have been written by either a person or AI; we would want a second check before making a strong claim.",
  "error": null,
  "verdict": ""
}
```

### MF01 — `malformed`

*Targeting:* text is only whitespace

```json
{
  "id": "MF01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF01",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF02 — `malformed`

*Targeting:* text is a single character

```json
{
  "id": "MF02",
  "status": 500,
  "content_id": null,
  "creator_id": "attacker_MF02",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": "response wasn't JSON: <!doctype html>\n<html lang=en>\n<title>500 Internal Server Error</title>\n<h1>Internal Server Error</h1>\n<p>The server encountered an internal error and was unable to complete your request. Either the s",
  "verdict": "BROKE"
}
```

### MF03 — `malformed`

*Targeting:* text is control characters

```json
{
  "id": "MF03",
  "status": 200,
  "content_id": "cd8e1c1341d64e5f930a363008478575",
  "creator_id": "attacker_MF03",
  "guess": "human",
  "confidence": 0.745,
  "model_score": 0.0,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### MF04 — `malformed`

*Targeting:* text is only punctuation

```json
{
  "id": "MF04",
  "status": 200,
  "content_id": "f179bb49f97849f5b670e11e54d4375b",
  "creator_id": "attacker_MF04",
  "guess": "human",
  "confidence": 0.4854,
  "model_score": 0.0112,
  "style_score": 1.0,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### MF05 — `malformed`

*Targeting:* text is a very long single word

```json
{
  "id": "MF05",
  "status": 200,
  "content_id": "6b08c2fcdedf486c8b5700523b1f101f",
  "creator_id": "attacker_MF05",
  "guess": "ai",
  "confidence": 0.7518,
  "model_score": 0.9976,
  "style_score": 0.65,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### MF06 — `malformed`

*Targeting:* text is emoji only

```json
{
  "id": "MF06",
  "status": 200,
  "content_id": "dea7c88c3ce3419a86adf97bc97fdb97",
  "creator_id": "attacker_MF06",
  "guess": "ai",
  "confidence": 0.996,
  "model_score": 0.9969,
  "style_score": 1.0,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### MF07 — `malformed`

*Targeting:* text is right-to-left script

```json
{
  "id": "MF07",
  "status": 200,
  "content_id": "37a10d43b11942de8175f09dfb99776e",
  "creator_id": "attacker_MF07",
  "guess": "ai",
  "confidence": 0.8784,
  "model_score": 0.9064,
  "style_score": 1.0,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### MF08 — `malformed`

*Targeting:* text is 60,000 characters

```json
{
  "id": "MF08",
  "status": 200,
  "content_id": "8b076d73d34f4e728c7cd794e7e4baa2",
  "creator_id": "attacker_MF08",
  "guess": "ai",
  "confidence": 0.8898,
  "model_score": 0.9876,
  "style_score": 0.8655,
  "label": "We think this was probably written by AI.",
  "error": null,
  "verdict": ""
}
```

### RQ01 — `malformed`

*Targeting:* no text field at all

```json
{
  "id": "RQ01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ01",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ02 — `malformed`

*Targeting:* no creator_id

```json
{
  "id": "RQ02",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ03 — `malformed`

*Targeting:* text is a number

```json
{
  "id": "RQ03",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ03",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ04 — `malformed`

*Targeting:* text is a list

```json
{
  "id": "RQ04",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ04",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ05 — `malformed`

*Targeting:* text is null

```json
{
  "id": "RQ05",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ05",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ06 — `malformed`

*Targeting:* creator_id is an object

```json
{
  "id": "RQ06",
  "status": 400,
  "content_id": null,
  "creator_id": {
    "id": "x"
  },
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ07 — `malformed`

*Targeting:* body is a JSON array, not an object

```json
{
  "id": "RQ07",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ08 — `malformed`

*Targeting:* body is unparseable JSON

```json
{
  "id": "RQ08",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ09 — `malformed`

*Targeting:* body is empty

```json
{
  "id": "RQ09",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ10 — `malformed`

*Targeting:* body is form-encoded, not JSON

```json
{
  "id": "RQ10",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ11 — `malformed`

*Targeting:* body is JSON claiming to be plain text

```json
{
  "id": "RQ11",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ12 — `malformed`

*Targeting:* deeply nested JSON

```json
{
  "id": "RQ12",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ12",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_01 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_01",
  "status": 200,
  "content_id": "a5c6c1748b0e4fa1bb85b91ff416346c",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_02 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_02",
  "status": 200,
  "content_id": "8c70df920a624a6495388914ad52b61a",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_03 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_03",
  "status": 200,
  "content_id": "d16c302e39004774a3bfa24cf6f459b6",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_04 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_04",
  "status": 200,
  "content_id": "dedd9d10dae0450fadf20ac097e29b63",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_05 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_05",
  "status": 200,
  "content_id": "f2cf4c4b297341be821a1c4ffd5816c4",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_06 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_06",
  "status": 200,
  "content_id": "16dc0746975c4981930c7a3e9c9f09b6",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_07 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_07",
  "status": 200,
  "content_id": "8a83f4a475734f28bd2400be82225cc5",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_08 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_08",
  "status": 200,
  "content_id": "94d7764fa67e467495c1d41b91cca515",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_09 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_09",
  "status": 200,
  "content_id": "0a068d234d034341a7d59f6e2f80ad75",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_10 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_10",
  "status": 200,
  "content_id": "d7cec120d7f94318b21cab0f41165ca9",
  "creator_id": "attacker_flood",
  "guess": "human",
  "confidence": 0.7076,
  "model_score": 0.0287,
  "style_score": 0.65,
  "label": "We think this was probably written by a person.",
  "error": null,
  "verdict": ""
}
```

### FL01_11 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_11",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_12 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_12",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```
