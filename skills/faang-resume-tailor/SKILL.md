---
name: faang-resume-tailor
description: Tailors an existing resume/LinkedIn to a specific FAANG SWE/SRE/ML job description, applying the locked 24-rule FAANG rulebook. Inputs: JD text + existing resume/LinkedIn. Outputs: tailored markdown draft for the verify gate. Does NOT export — verification must pass first.
---

# FAANG Resume Tailor

## Overview

This skill takes a candidate's existing resume (or LinkedIn export) plus a job description, and produces a tailored draft that conforms to the FAANG SWE/SRE/ML rulebook. The output is markdown — exporting to DOCX/PDF requires `faang-resume-verify` to PASS first.

**Tailoring, not generation.** The skill never invents evidence. It reframes, reorders, sharpens, and surfaces what's already in the source. Gaps against the JD are explicitly flagged for the candidate to fill.

## When to Use

- Candidate has a JD and an existing resume/LinkedIn and wants to apply
- Candidate has applied with a generic resume and wants to tailor for a specific opening
- A previously-tailored resume needs re-tailoring for a different JD

**When NOT to use:** Building a resume from scratch with no existing source material — that's a different workflow (intake interview), out of scope for this plugin.

## Inputs

1. **Job description** — pasted text or URL
2. **Existing resume** — markdown, DOCX, PDF, or LinkedIn profile export

## The Rulebook (Locked, 24 Rules)

These rules are non-negotiable. The verify skill enforces them numerically.

### Bullet Rules

- **B1.** Strict XYZ format: *"Accomplished [X] as measured by [Y] by doing [Z]."* Order can flex; all three must be present.
- **B2.** 100% quantification, zero exceptions. Every bullet has a number. If the source bullet lacks one, ask the candidate before writing.
- **B3.** Bullet counts: most recent role 3–5, prior 3–4, older 1–2, internships 1.
- **B4.** Banned bullet starts: Helped, Assisted, Supported, Worked on, Was responsible for, Participated in, Contributed to (alone), Involved in.
- **B5.** Tailoring rule: never invent evidence. Reframe what's there; flag gaps.

### Scope & Specificity

- **S1.** Named systems & tech required for technical bullets (`Postgres` not "database", `Kafka` not "queue", `EKS` not "cloud"). Optional for leadership/process bullets.
- **S2.** At least one scale signal somewhere on the resume: RPS, QPS, p50/p95/p99 latency, # services, DAU/MAU, data volume (TB/PB), $ARR, throughput.
- **S3.** At least one bullet framed as an incident, migration, or scaling-crisis ("what broke / what you fixed at scale").

### Layout, Header, Section Order

- **L1.** Strict 1 page. Always. No exceptions.
- **L2.** Experience first. Always. (FAANG internships count as experience.)
- **L3.** Header: name, headline, email, phone with country code, LinkedIn, GitHub, optional portfolio (Google Scholar for ML). Nothing else.
- **L4.** Single-column, ≤2 fonts, 10.5–11pt body, 0.5–0.75" margins, plain bullets, dates as `Month YYYY – Month YYYY`.
- **L5.** Section order: Experience → Education → Skills → Publications/Projects/OSS (only if material).

### The Cut List

- **C1.** GPA — never show.
- **C2.** Soft skills — banned as a section.
- **C3.** Hobbies, interests, languages spoken (unless JD requires multilingual), volunteer work — cut entirely.
- **C4.** Roles older than 10 years — cut entirely.

### Process

- **P1.** Plugin requires JD + existing resume/LinkedIn as input.
- **P2.** Output: tailored markdown + ATS-safe DOCX + ATS-safe PDF + recruiter-style review (only after verify PASS).
- **P3.** Anchor: SWE/SRE/ML engineering roles only.

## Process

### Step 1: Parse inputs

- Extract the JD text. Identify the role title, company, required years of experience, must-have technologies, nice-to-haves, and the "what you'll do" section.
- Parse the existing resume into structured sections: header, experience (per role), education, skills, projects, publications.
- Note any source-resume content that violates the cut list (C1–C4) — these will be removed.

### Step 2: Build the targeting brief

For the JD, produce:

```
TARGETING BRIEF
─────────────────────────────────────────────────────────────
Company:        [name]
Role title:     [verbatim from JD]
JD URL/source:  [...]

Top 5 keywords (what the recruiter is scanning for):
  1. [phrase from JD]
  2. ...

Must-haves (verbatim from JD):
  M1. [requirement]
  M2. ...

Nice-to-haves:
  N1. [requirement]
  N2. ...

Hidden disqualifiers:
  - Location:           [...]
  - Work authorization: [...]
  - On-call:            [...]

Coverage table (source resume vs JD):
  Requirement              Status      Source evidence
  ─────────────────────────────────────────────────────────
  M1. 5y Go backend       ✅ STRONG    "7y Go at Acme since 2018"
  M2. K8s production       🟡 PARTIAL  "EKS at Acme, no charts"
  M3. Postgres at scale    ❌ GAP      (no evidence in source)
  ...

Gaps requiring user input:
  - M3: No Postgres-at-scale evidence in source. Ask candidate:
    "Did you operate Postgres in production? At what scale?"
─────────────────────────────────────────────────────────────
```

**Surface gaps to the user immediately.** Do not write bullets for evidence that doesn't exist. If a must-have is at ❌ GAP, ask the candidate before continuing.

### Step 3: Apply the cut list

Remove from the source resume, in order:

1. GPA — wherever it appears (C1)
2. Any "Soft Skills," "Core Competencies," "Personal Attributes" section (C2)
3. Hobbies, interests, languages (unless JD requires), volunteer work (C3)
4. Any role with end date more than 10 years before today (C4)
5. "References available upon request," "Objective," street address, photo, DOB

### Step 4: Rewrite the header

```
[NAME]
[One-line targeted headline mirroring JD role title + 1 specialization clause]
City, ST · email@domain.com · +1 555 123 4567 · linkedin.com/in/handle · github.com/handle
[for ML candidates: + scholar.google.com/...]
```

The headline is critical. It must **mirror the JD's role title** — if the JD says "Senior Software Engineer, Distributed Systems," the headline is something like "Senior software engineer specializing in distributed systems and high-throughput backend platforms," not "Passionate engineer with 8 years of experience."

### Step 5: Rewrite each role's bullets

For each role, in reverse chronological order, with bullet count per B3:

For each bullet from the source:

1. **Identify the underlying achievement.** Not the responsibility — the outcome.
2. **Apply XYZ format (B1):** Accomplished [X] as measured by [Y] by doing [Z].
3. **Verify quantification (B2):** Does the bullet contain a number? If not, return to source. If source has none, ask candidate.
4. **Verify named tech (S1):** For technical bullets, named system/tool/framework? If generic, return to source for specifics.
5. **Verify verb (B4):** Does it start with a strong verb (not banned list)?
6. **Tighten to ≤2 lines.**
7. **Check JD alignment:** Does this bullet's keywords match the targeting brief? If yes for top-5 keyword, mark it.

After all roles are written, verify:
- Resume contains at least one scale signal (S2). If not, return to source / ask candidate.
- Resume contains at least one incident/migration/crisis bullet (S3). If not, prompt: *"FAANG resumes need at least one incident/migration/scaling story. Tell me about a time something was broken or at its limit, and what you did."*

### Step 6: Trim Education and Skills

**Education:**
- Degree, institution, graduation year. No GPA (C1).
- Honors only if highly distinctive (summa cum laude, Phi Beta Kappa, etc.).
- Relevant coursework only for new grads with <2 years experience.

**Skills:**
- Grouped by category (Languages / Infra / Data / Practices).
- Truth list only — only items used in real shipped work.
- No soft skills (C2).
- No "Microsoft Office," "Email," etc.

### Step 7: Compress to 1 page

Run a length check. If >1 page (L1):

1. First, cut bullets tagged WEAK in any role.
2. Then, compress older roles (5+ years back) toward 1–2 bullets.
3. Then, drop the entire oldest role if you must (subject to C4 — already removed if >10 years).
4. Then, drop Skills section to fewer categories.
5. If still >1 page after all of the above, **stop and escalate to user:** *"Cannot fit on 1 page. Either drop one of these [list], or accept 2 pages knowing it violates FAANG conventions."*

### Step 8: Produce the markdown draft

Output the tailored resume as markdown, suitable for input to `faang-resume-verify`. Save it to a file.

```markdown
# [Name]

**[Targeted headline]**

City, ST · email@domain.com · +1 555 123 4567 · linkedin.com/in/handle · github.com/handle

## Experience

### Senior Backend Engineer · Acme Corp · May 2022 – Present
*Acme builds payment infrastructure for marketplaces; team of 6 owns the core ledger.*

- [Bullet 1, XYZ format, quantified, named tech]
- [Bullet 2, ...]
- [Bullet 3, ...]

### Backend Engineer · Beta Inc · Aug 2019 – Apr 2022
*[Context line.]*

- [Bullet 1, ...]
- ...

## Education

**MS Computer Science**, Stanford University, 2019
**BS Computer Science**, UIUC, 2017

## Skills

- **Languages:** Go, Python, TypeScript, SQL
- **Infra:** AWS (EKS, RDS, S3), Terraform, GitHub Actions
- **Data:** Postgres, Redis, Kafka, dbt
- **Practices:** On-call rotation, incident command, RFCs, mentoring
```

### Step 9: Hand off to verify

**Do not export.** The skill's contract ends here. Output:

```
TAILOR DRAFT COMPLETE.

Markdown saved to: /tmp/resume-tailored.md
Targeting brief:   /tmp/targeting-brief.md

Next: run faang-resume-verify. Both gates must PASS before
DOCX/PDF export. If verify returns FAIL, this skill receives
the fix list and re-runs.
```

## Common Rationalizations

| Rationalization | Reality |
| --- | --- |
| "The bullet doesn't have a number, but it's a strong verb." | B2 is absolute. Either find the number in the source, ask the candidate, or cut the bullet. |
| "The candidate doesn't have Postgres experience but I'll just add it to skills." | B5 forbids inventing evidence. Surface the gap — let verify catch it if it slips through. |
| "Two pages is fine for a senior candidate, FAANG will understand." | L1 is absolute. Strict 1 page. Compress or escalate to candidate; do not silently spill. |
| "The headline 'Software Engineer with 8 years experience' is fine." | It's not targeted. Mirror the JD's role title with a specialization clause. |
| "I don't need to flag the must-have gap, the candidate will handle it." | They won't — they don't see the gap because they wrote the source resume. Flag every ❌ GAP explicitly. |

## Red Flags

- Writing a bullet that the candidate hasn't confirmed contains a real number
- Adding a technology to skills that doesn't appear in any bullet
- Producing a 2-page draft and proceeding anyway
- A must-have at ❌ GAP that wasn't flagged to the candidate
- The headline doesn't change between different JDs (it should)
- Source resume has 8 bullets in current role; tailored has 8 bullets too (B3 says 3–5 — you're not tailoring, you're reformatting)

## Verification

Before handing off to `faang-resume-verify`:

- [ ] Targeting brief saved with coverage table
- [ ] All ❌ GAPs surfaced to user with specific questions
- [ ] All cut-list items (C1–C4) removed from source
- [ ] Header rewritten with targeted headline (not generic)
- [ ] Every bullet starts with non-banned verb, contains a number, follows XYZ structure
- [ ] Bullet counts per role match B3
- [ ] At least one scale signal present (S2)
- [ ] At least one incident/migration/crisis bullet present (S3)
- [ ] Document fits on 1 page at 10.5–11pt body
- [ ] Markdown draft saved to file for verify skill input
