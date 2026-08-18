# Return to Run — Portal Proof of Concept

An interactive proof of concept for **The Lab — Rehab and Performance**: a web portal that replaces a
return-to-running progression spreadsheet with a patient-facing portal and a clinician caseload console.

**Live demo:** https://floor16tech.github.io/RunLabPOC/

Single self-contained `index.html`. No build step, no dependencies, no network calls — open it in any browser.

---

## What it does

The original workbook prescribes a weekly running volume, collects three answers after every run, scores the
week, and decides whether the next week goes up, holds, or comes back down. That logic is ported verbatim:

```
run score  = 0.4 · (1 − 0.025 · pain_during)
           + 0.4 · (1 − 0.025 · pain_next_morning)
           + 0.2 · (symptoms back to baseline ? 1 : 0.5)

week score = Σ(km · run score) ÷ Σ(km)

≥ 0.91 → increase volume     ≥ 0.90 → hold     below → reduce 10%
```

When symptoms return to baseline this simplifies to `1 − 0.01 · (pain_during + pain_after)`, so the working
rule is: *settled and combined pain ≤ 9 → progress; = 10 → hold; anything else → deload.*

## Three roles, switchable from the top bar

| Role | What it covers |
|---|---|
| **Patient** | Next run and its distance, a three-question log, weekly progress, the plan, and one message thread with their physiotherapist. The scoring maths is never shown. |
| **Clinician** | Caseload triage with red flags sorted to the top, per-patient week-by-week breakdown, a live programme builder, algorithm overrides with a recorded reason, and a chat list — one thread per patient. |
| **Administrator** | Protocol templates, clinic-wide scoring thresholds, staff and patient accounts, audit log, privacy and retention settings. |

A programme picker in Patient view switches between six demonstration cases: mid-programme after a deload,
a patient going backwards, a red-flagged bone stress reaction, one at goal volume and ready for discharge,
one not yet started, and one with a missed run.

## Defects found in the original workbook

These are corrected in the engine, and each correction is an individually switchable setting under
**Administrator → Algorithm**:

1. **Cell `L4`** (week 1, Monday) scores pain at `0.05` per point with `0.75` for "No", while every other run
   uses `0.025` and `0.5` — one run per programme was graded on a different scale.
2. **Columns R–W** (weeks 4–8) compare a response score against `1.05` and read Green/Yellow/Red cells that are
   never populated, so weeks 6–8 never progressed.
3. **A blank answer scored as "No."** An unlogged run entered `SUMPRODUCT` as zero and silently capped the week
   at 0.90, forcing a hold. Missing data is now excluded rather than punished.
4. **No ceiling at the goal volume** — the spreadsheet kept multiplying past 100%. Now capped, with the patient
   flagged as ready for discharge.
5. **No hard safety stop** — a 9/10 run cost only 10% of volume. A severe run now halts progression and raises
   a red flag to the clinician.

One further observation surfaced in the UI: at the workbook's defaults (30% starting reduction, 15% weekly
increase) the goal volume is reached in **three consecutive green weeks**, so an 8-week grid only makes sense
for a patient who has several hold or deload weeks along the way.

---

## Disclaimer

Proof of concept only. **Not a medical device and not for clinical use.** All patients, programmes and messages
are fabricated demonstration data — no real patient information appears anywhere. Progression logic adapted from
a return-to-running spreadsheet by Maciek Krolikowski, PT #8857.

Built by [Floor 16 Technologies](https://floor16.com).
