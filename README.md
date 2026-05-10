# FAANG Resume

**Tailors an existing resume to a specific FAANG SWE/SRE/ML job description, then refuses to ship until it passes both ATS and JD-match verification gates.**

```
JD + existing resume
        │
        ▼
┌──────────────────┐
│      TAILOR      │  Apply 24-rule FAANG rulebook
└────────┬─────────┘  Surface gaps, never invent evidence
         │
         ▼
┌──────────────────┐  GATE 1: ATS Score      ≥ 95/100
│      VERIFY      │  GATE 2: JD Match Score ≥ 90/100
└────────┬─────────┘  FAIL → loop back. PASS → ship.
         │
         ▼
┌──────────────────┐
│      EXPORT      │  Markdown + ATS-safe DOCX + PDF
└──────────────────┘  + recruiter-style review
```

## What makes this plugin different

Most resume tools generate plausible-sounding text and call it done. This plugin enforces a **strict, locked rulebook** developed in dialogue with a resume-builder expert, and **refuses to produce final output** until two numerical gates pass:

- **ATS Score ≥ 95/100** — the resume parses cleanly through Workday, Greenhouse, Lever, Taleo, and other ATS systems used by FAANG-tier companies
- **JD Match Score ≥ 90/100** — the resume's evidence aligns with the JD's stated must-haves, with no ❌ GAP on any required qualification

If either gate fails, the plugin loops back to tailoring with **specific fixes**, not vague suggestions. Maximum 5 cycles before it escalates to the user instead of producing weak output.

## The rulebook (24 rules, locked)

### Bullets
- **B1.** Strict XYZ format: *"Accomplished [X] as measured by [Y] by doing [Z]"*
- **B2.** 100% quantification — every bullet has a number, no exceptions
- **B3.** Bullet counts: most recent 3–5 / prior 3–4 / older 1–2 / internships 1
- **B4.** Banned verbs: Helped, Assisted, Supported, Worked on, Was responsible for, Participated in, Contributed to, Involved in
- **B5.** Never invent evidence — tailor what's there, flag gaps

### Scope & specificity
- **S1.** Named systems & tech required for technical bullets (`Postgres` not "database")
- **S2.** At least one scale signal on the resume (RPS, p99, # users, etc.)
- **S3.** At least one incident/migration/scaling-crisis bullet — the bar-raiser pattern

### Layout
- **L1.** Strict 1 page. Always. No exceptions.
- **L2.** Experience first. Always.
- **L3.** Header: name, headline, email, phone, LinkedIn, GitHub, optional portfolio. Nothing else.
- **L4.** Single-column, ≤2 fonts, 10.5–11pt body, plain bullets, dates as `Month YYYY – Month YYYY`
- **L5.** Section order: Experience → Education → Skills → Publications/Projects (only if material)

### Cut list
- **C1.** GPA — never show
- **C2.** Soft skills section — banned (demonstrate in bullets only)
- **C3.** Hobbies, interests, languages, volunteer work — cut entirely
- **C4.** Roles older than 10 years — cut entirely

### Process
- **P1.** Inputs required: JD + existing resume/LinkedIn
- **P2.** Outputs: markdown + ATS-safe DOCX + ATS-safe PDF + recruiter review
- **P3.** Anchor: SWE/SRE/ML engineering roles only

## The verification gates

### Gate 1: ATS Score (≥ 95/100)

Six categories, 100 points total:

| Category | Points | What it checks |
| -------- | ------ | -------------- |
| Structural parseability | 40 | Single-column, no tables/text-boxes, no images-of-text, plain bullets, selectable text |
| Section header parseability | 15 | Standard headers (Experience / Education / Skills) |
| Field extraction | 15 | Name on line 1, standard email/phone format, role lines parseable |
| Date format consistency | 10 | Same format throughout, no ambiguous dates (`Q3 '23` etc.) |
| File-level | 10 | Source-generated PDF, ≤ 1MB, correct filename convention |
| Keyword density sanity | 10 | No keyword stuffing, no JD verbatim |

Any structural failure (tables, two-column, images-of-text) = automatic FAIL regardless of score.

### Gate 2: JD Match Score (≥ 90/100)

Six categories, 100 points total:

| Category | Points | What it checks |
| -------- | ------ | -------------- |
| Must-have coverage | 50 | Every JD must-have has STRONG/PARTIAL/GAP evidence rating |
| Top-5 keyword presence | 20 | Each top-5 JD keyword appears at least once in real evidence |
| Headline alignment | 10 | Headline mirrors JD role title + specialization |
| Top-of-resume relevance | 10 | First 2 bullets of recent role address JD's biggest ask |
| Scale-signal alignment | 5 | Scale signal matches JD's environment |
| Incident/migration bullet | 5 | At least one bar-raiser pattern bullet present |

Any must-have at ❌ GAP caps the score at 80/100, forcing a loop back.

## Hard rules (no bypasses)

- **V1.** No "ship anyway" override — even on user request
- **V2.** Manual edits after PASS invalidate the score; re-verify required
- **V3.** When in doubt on a rubric item, score down not up
- **V4.** Maximum 5 tailor → verify cycles, then escalate to user
- **V5.** Gate 1 structural blockers stop everything — fix structure before scoring Gate 2
- **V6.** All three outputs ship together (markdown + DOCX + PDF) or not at all

## Project structure

```
faang-resume/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── faang-resume-tailor/
│   │   └── SKILL.md       # Tailor the resume to the JD using the 24 rules
│   └── faang-resume-verify/
│       └── SKILL.md       # Hard ATS + JD-match gates, refuses to ship if fail
├── .claude/commands/
│   └── faang-resume.md    # End-to-end slash command
└── README.md
```

## Quick start

**Claude Code:**

```
/plugin marketplace add <your-user>/faang-resume
/plugin install faang-resume@faang-resume-marketplace
/faang-resume
```

Then paste your JD and provide your existing resume. The plugin will tailor, verify, and ship — or refuse to ship and tell you exactly what's missing.

**Other agents (Cursor, Gemini CLI, Windsurf, Codex):** the SKILL.md files are plain Markdown — drop them into your tool's rules directory and they work unchanged.

## License

MIT — use these skills in your job search, with your clients, in your own tools.
