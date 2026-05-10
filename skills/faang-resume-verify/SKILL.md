---
name: faang-resume-verify
description: Hard verification gate that scores a tailored resume on (1) ATS parseability and (2) job-description match. Refuses to ship unless both scores meet threshold. Run after `faang-resume-tailor`, before any export. The plugin will not produce final DOCX/PDF until this skill returns PASS on both gates.
---

# FAANG Resume Verify

## Overview

Two gates. Both must pass. No exceptions, no overrides, no "ship it anyway."

```
                ┌─────────────────────────────────┐
                │   Tailored resume (markdown)    │
                └────────────────┬────────────────┘
                                 │
                ┌────────────────▼────────────────┐
                │       GATE 1: ATS SCORE         │
                │  Threshold: ≥ 95/100            │
                └────────────────┬────────────────┘
                                 │
                       ┌─────────┴─────────┐
                     FAIL                 PASS
                       │                   │
                       ▼                   ▼
                 Return fixes    ┌─────────────────────┐
                 to tailor       │  GATE 2: JD MATCH   │
                                 │  Threshold: ≥ 90/100│
                                 └──────────┬──────────┘
                                            │
                                  ┌─────────┴─────────┐
                                FAIL                 PASS
                                  │                   │
                                  ▼                   ▼
                            Return fixes        SHIP ALLOWED
                            to tailor           Export DOCX/PDF
```

The plugin's contract is simple: **the verification skill is the only path to shipping.** `faang-resume-tailor` produces a draft. This skill scores it. If it fails, the draft loops back to the tailor with specific fixes. The cycle continues until both gates return PASS.

## When to Use

- After `faang-resume-tailor` produces a draft, before any export
- When a user uploads a finished resume and asks "is this FAANG-ready?"
- After any manual edit to a previously-passed resume — re-verify, never trust prior PASS

**When NOT to use:** Mid-tailoring. Verification is the gate, not a feedback tool during writing.

## Inputs

1. **Tailored resume markdown** (output of `faang-resume-tailor`)
2. **Original JD text**
3. **Targeting brief** (the keyword/requirement extraction from the tailor skill)

## GATE 1 — ATS Score (Threshold: ≥ 95/100)

The resume must parse cleanly through ATS systems used by FAANG and FAANG-adjacent companies (Workday, Greenhouse, Lever, Taleo, iCIMS, Ashby, SuccessFactors). 95/100 is the threshold because a single ATS-breaking element can make the resume invisible — there is no "mostly parseable."

### ATS scoring rubric (100 points)

#### A. Structural parseability (40 points — all-or-nothing per item)

| Check | Points | Pass condition |
| ----- | ------ | -------------- |
| **A1.** No tables, text boxes, or sidebars used for layout | 10 | Document is single-column flow text |
| **A2.** No images, icons, or graphics containing text | 10 | Name, contact info, skills are all selectable text |
| **A3.** No headers/footers in page-margin sense (with parseable content trapped there) | 5 | Contact info is in document body, not page header |
| **A4.** Standard bullet characters (`•` or `-`) used consistently | 5 | No `►`, `★`, `✦`, custom Unicode |
| **A5.** Text is selectable in the rendered file (not image-based PDF) | 10 | Cmd-A → Cmd-C → paste produces clean text |

**Any failure here = automatic GATE 1 FAIL regardless of total score.** These are blockers, not deductions.

#### B. Section header parseability (15 points)

| Check | Points | Pass condition |
| ----- | ------ | -------------- |
| **B1.** "Experience" / "Work Experience" / "Professional Experience" header present | 5 | Exact match (case-insensitive) |
| **B2.** "Education" header present | 5 | Exact match |
| **B3.** "Skills" / "Technical Skills" header present | 5 | Exact match |

Creative variants ("Where I've Worked", "My Toolbox") = 0 points for that row. ATS parsers don't recognize them.

#### C. Field extraction (15 points)

| Check | Points | Pass condition |
| ----- | ------ | -------------- |
| **C1.** Candidate name is line 1 of the document, plain text | 5 | Not in an image, not in a sidebar |
| **C2.** Email present in standard `name@domain.tld` format | 3 | Searchable with `@` regex |
| **C3.** Phone present with country code, no decorative formatting | 2 | E.g. `+1 555 123 4567`, not `(.5.5.5.).1.2.3.4.5.6.7` |
| **C4.** Each role has Title, Company, and Date on adjacent lines | 5 | Dates in `Month YYYY – Month YYYY` format |

#### D. Date format consistency (10 points)

| Check | Points | Pass condition |
| ----- | ------ | -------------- |
| **D1.** All role dates use the same format throughout | 5 | No mixing `May 2022 – Present` with `8/2019 - 12/2021` |
| **D2.** No ambiguous date forms (`Q3 '23`, `Spring 2022`, `since 2020`) | 5 | All dates are unambiguous to a parser |

#### E. File-level checks (10 points)

| Check | Points | Pass condition |
| ----- | ------ | -------------- |
| **E1.** File is generated from source (not "Print to PDF") | 3 | Semantic structure preserved |
| **E2.** File size ≤ 1 MB | 3 | Most ATS portals reject larger silently |
| **E3.** File name matches `Firstname-Lastname-Role.{pdf,docx}` | 4 | No `resume.pdf`, no `Resume_FINAL_v7.pdf` |

#### F. Keyword density sanity (10 points)

| Check | Points | Pass condition |
| ----- | ------ | -------------- |
| **F1.** No single keyword appears more than 5 times | 5 | Stuffing detection — modern ATS flag this |
| **F2.** No JD sentence is reproduced verbatim | 5 | Plagiarism detection in modern ATS |

### Gate 1 verdict

```
ATS SCORE = sum of points earned out of 100

≥ 95 → PASS
< 95 → FAIL — return to tailor with item-by-item fixes
```

**Any A-section blocker (A1–A5) = automatic FAIL regardless of total.** If the document has a table or two-column layout, the score is irrelevant — it does not parse, period.

---

## GATE 2 — JD Match Score (Threshold: ≥ 90/100)

The resume must align with the JD's stated requirements. 90/100 is the threshold because: must-haves are non-negotiable (lose any and you fail), but nice-to-haves and signals admit some flexibility.

### JD Match scoring rubric (100 points)

#### A. Must-have coverage (50 points)

Extract every must-have requirement from the JD (phrased with "required," "must," "X+ years," explicit qualifications). For each must-have:

| Evidence quality | Points (per must-have, normalized to 50) |
| ---------------- | ---------------------------------------- |
| ✅ STRONG: direct, recent, quantified evidence in the resume | Full points |
| 🟡 PARTIAL: adjacent skill, older role, or unquantified | Half points |
| ❌ GAP: no evidence in the resume | Zero points |

**Hard rule:** if any must-have is at ❌ GAP, the score for that must-have is zero AND the verdict carries a flag. **A resume with a ❌ GAP on any must-have cannot exceed 80/100 on this gate** regardless of how strong the rest is. This forces the loop back to the tailor — either surface evidence from the source resume that wasn't being used, or escalate the gap to the user.

**Calculation:** If there are N must-haves, each is worth 50/N points. Sum the awarded points.

```
Example with 5 must-haves:
- 5y Go backend         → ✅ STRONG → 10 pts
- Distributed systems   → ✅ STRONG → 10 pts
- Kubernetes experience → 🟡 PARTIAL → 5 pts
- Postgres at scale     → ✅ STRONG → 10 pts
- On-call / production  → 🟡 PARTIAL → 5 pts
                                       ─────
                          Subtotal:    40/50
```

#### B. Top-5 keyword presence (20 points)

Extract the top 5 keywords from the JD (the words a recruiter is keyword-scanning for, distinct from full requirements). Each keyword:

| Status | Points |
| ------ | ------ |
| Appears at least once in real evidence (not stuffed) | 4 |
| Appears in skills list only, not in evidence bullets | 2 |
| Does not appear at all | 0 |

#### C. Headline alignment (10 points)

| Check | Points |
| ----- | ------ |
| Headline mirrors the JD's role title with a specialization clause | 10 |
| Headline is generic ("Software Engineer with X years") | 4 |
| Headline missing or completely off-target | 0 |

#### D. Top-of-resume relevance (10 points)

The first 2 bullets of the most recent role determine 6-second-scan outcome.

| Check | Points |
| ----- | ------ |
| Both top bullets directly address JD's most-emphasized requirement | 10 |
| One of two top bullets addresses the most-emphasized requirement | 6 |
| Top bullets address adjacent skills | 3 |
| Top bullets unrelated to JD focus | 0 |

#### E. Scale-signal alignment (5 points)

| Check | Points |
| ----- | ------ |
| Resume contains a scale signal (RPS, p99, # users, etc.) AND it matches the JD's scale context | 5 |
| Resume contains a scale signal but it doesn't reflect the JD's environment | 3 |
| No scale signal anywhere on the resume | 0 |

#### F. Incident/migration/crisis bullet (5 points)

| Check | Points |
| ----- | ------ |
| Resume contains at least one bullet framed as an incident/migration/scaling crisis | 5 |
| Resume contains operations/reliability work but no crisis framing | 2 |
| No such bullet exists | 0 |

### Gate 2 verdict

```
JD MATCH SCORE = sum of points earned out of 100

≥ 90 AND no ❌ GAP on any must-have → PASS
< 90 OR any must-have at ❌ GAP    → FAIL — return to tailor with gap analysis
```

---

## Process

### Step 1: Run Gate 1 (ATS Score)

Score the resume against the ATS rubric, item by item. Output a structured score card:

```
================================================================
GATE 1: ATS SCORE
================================================================

A. Structural parseability        [40/40]
   ✓ A1. Single-column, no tables    [10/10]
   ✓ A2. No images of text           [10/10]
   ✓ A3. No header/footer trap       [ 5/5 ]
   ✓ A4. Standard bullets            [ 5/5 ]
   ✓ A5. Text selectable             [10/10]

B. Section headers                [15/15]
   ✓ B1. Experience header           [ 5/5 ]
   ✓ B2. Education header            [ 5/5 ]
   ✓ B3. Skills header               [ 5/5 ]

C. Field extraction               [13/15]
   ✓ C1. Name on line 1              [ 5/5 ]
   ✓ C2. Email standard format       [ 3/3 ]
   ✓ C3. Phone with country code     [ 2/2 ]
   ✗ C4. Role lines incomplete       [ 3/5 ]
        → 2 roles missing location on the role line

D. Date format consistency        [10/10]
   ✓ D1. Format consistent           [ 5/5 ]
   ✓ D2. No ambiguous dates          [ 5/5 ]

E. File-level                     [10/10]
   ✓ E1. Source-generated PDF        [ 3/3 ]
   ✓ E2. Size 287KB                  [ 3/3 ]
   ✓ E3. Filename correct            [ 4/4 ]

F. Keyword density                [10/10]
   ✓ F1. No keyword > 5x             [ 5/5 ]
   ✓ F2. No verbatim JD sentences    [ 5/5 ]

----------------------------------------------------------------
TOTAL: 98/100                            STATUS: PASS ✓
================================================================
```

If FAIL: list each failed item with a specific fix the tailor skill can apply. Do not advance to Gate 2.

### Step 2: Run Gate 2 (JD Match Score)

Score the resume against the JD requirements, item by item. Output:

```
================================================================
GATE 2: JD MATCH SCORE
================================================================

JD: Senior Backend Engineer @ Stripe
   https://stripe.com/jobs/listing/...

A. Must-have coverage             [40/50]
   Must-have                          Score      Evidence
   ─────────────────────────────────────────────────────────────
   ✓ 5+ years Go backend             10/10  ✅ "7y Go at Acme since 2018"
   ✓ Distributed systems at scale    10/10  ✅ "14 services, 2M req/min"
   ⚠ Kubernetes production exp        5/10  🟡 "EKS at Acme, never wrote charts"
   ✓ Postgres at scale               10/10  ✅ "Sharded Postgres, 8TB"
   ⚠ On-call / production            5/10  🟡 "Participated in rotation"
   
   No ❌ GAPs found.

B. Top-5 keyword presence         [18/20]
   ✓ go (programming)            4/4   appears 3x in evidence
   ✓ distributed-systems         4/4   appears 2x in evidence
   ⚠ kubernetes                  2/4   skills list only
   ✓ postgres                    4/4   appears 2x in evidence
   ✓ on-call                     4/4   appears 1x in evidence

C. Headline alignment             [10/10]
   ✓ "Senior backend engineer specializing in distributed
      systems and high-throughput payment infrastructure"
      → Mirrors JD title + specialization clause

D. Top-of-resume relevance        [10/10]
   ✓ Top 2 bullets at Acme address payments scale + reliability

E. Scale-signal alignment         [ 5/5]
   ✓ "2M req/min, p99 = 87ms" matches Stripe's scale context

F. Incident/migration bullet      [ 5/5]
   ✓ "Led migration from Mongo → Postgres, zero downtime"

----------------------------------------------------------------
TOTAL: 88/100                            STATUS: FAIL ✗
================================================================

REASON: Score 88 < 90 threshold.

REQUIRED FIXES (return to faang-resume-tailor):
  1. Kubernetes coverage is partial. Either:
     a) Surface stronger K8s evidence from the source resume
        if any exists (check the OSS section, side projects)
     b) Escalate to user: "Did you write any K8s manifests,
        Helm charts, or operate clusters in production?"
  2. On-call coverage is partial. Reframe "Participated in
     rotation" → quantify: how many incidents owned, what
     was the on-call cadence, what was MTTR improvement?

After tailor returns updated draft, re-run faang-resume-verify
from Gate 1.
================================================================
```

### Step 3: Issue ship verdict

Only when **both gates PASS** can the plugin proceed to export. Output:

```
================================================================
FAANG RESUME VERIFICATION: SHIP ALLOWED ✓

  ATS Score:      98/100  PASS
  JD Match Score: 92/100  PASS

The resume meets both gates. Proceeding to ATS-safe DOCX and
PDF export.
================================================================
```

If either gate fails, output:

```
================================================================
FAANG RESUME VERIFICATION: SHIP BLOCKED ✗

  ATS Score:      98/100  PASS
  JD Match Score: 88/100  FAIL

Returning to faang-resume-tailor with fixes listed above.
Do not proceed to export. Re-run verification after tailor
applies fixes.
================================================================
```

## Hard Rules — No Bypasses

These are non-negotiable enforcement rules. The plugin must implement them:

**V1.** **No "ship anyway" override.** Even if the user says "just export it," the plugin refuses if either gate is in FAIL state. The plugin must respond: *"Cannot export — Gate X failed at score Y. Specific fixes required: [list]. Run faang-resume-tailor with these fixes, then re-verify."*

**V2.** **Re-verification after manual edits.** If the user manually edits the resume after a previous PASS, the prior score is invalidated. The plugin must re-run both gates before any export.

**V3.** **No score inflation.** When in doubt on a rubric item, score down, not up. A 🟡 PARTIAL is not a ✅ STRONG. A creative section header is not a standard one. Score honestly.

**V4.** **Maximum 5 verify-tailor loops.** If after 5 cycles the resume still fails, the plugin must escalate to the user with the message: *"After 5 iterations, the resume cannot reach FAANG threshold without additional evidence from you. Specifically, [list of GAPs]. Please provide answers to these questions, or accept that this role may be a stretch given your current evidence."* This prevents infinite loops on candidates whose source material genuinely doesn't match the JD.

**V5.** **Gate 1 blockers stop everything.** Any A-section failure (A1–A5) in Gate 1 means the document is structurally broken for ATS. Do not score Gate 2 — fix the structural problem first.

**V6.** **No partial export.** The plugin produces all three outputs together (markdown, DOCX, PDF) or none. No "DOCX is ready, PDF needs work." Both files come from the same verified source.

## Common Rationalizations

| Rationalization | Reality |
| --- | --- |
| "The score is 89, that's basically 90." | 89 is not 90. The threshold exists to prevent exactly this kind of negotiation. Loop back. |
| "The user is in a hurry, ship it." | The CV exists for years. The 30 minutes saved costs the interview. V1 enforces this. |
| "The Kubernetes gap is fine, the candidate can address it in the interview." | If they make it to the interview. The resume is filtered before then. Surface the evidence or escalate the gap. |
| "Two-column layout looks more professional, the user wants it." | A1 is a hard blocker. The score is automatic FAIL. No exceptions. |
| "Most ATS systems can handle tables now." | Workday and Taleo, the two most common at FAANG-scale companies, still mangle tables in 2026. Refuse. |
| "We've already iterated 5 times, just ship the 88." | V4 says escalate, not ship. The user makes the call about a stretch role, not the plugin. |

## Red Flags

- Gate 1 score is 95/100 but A1–A5 has a failure (the document is structurally broken; total score is misleading)
- Gate 2 score is 91/100 but a must-have is at ❌ GAP (the rubric prevents this combination, but if seen, recheck the scoring)
- Verification has been run 3+ times with the same fix list (the tailor skill is not applying the fixes — escalate)
- The user requests a manual override of either gate (refuse per V1)
- A re-verification produces a higher score than the previous run with no source-resume changes (something was scored too generously)

## Verification (the meta-level)

The verification skill itself must produce evidence. Before issuing PASS:

- [ ] The score card is fully populated — no item left unscored
- [ ] Both gates have explicit numerical scores
- [ ] Every FAIL item has a specific, actionable fix the tailor skill can implement
- [ ] No A-section blocker is being overlooked
- [ ] The JD's must-haves were extracted from the actual JD text, not assumed
- [ ] Loop count is tracked and within V4 limits
- [ ] If PASS: the file paths to the markdown, DOCX, and PDF are confirmed before export
