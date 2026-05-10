---
description: Tailor an existing resume to a specific FAANG SWE/SRE/ML job description, verify it passes both ATS and JD-match gates, then export ATS-safe DOCX/PDF.
---

# /faang-resume

End-to-end FAANG resume tailoring with hard verification gates.

## Inputs

Ask the user for:

1. **Job description** — paste text or provide URL
2. **Existing resume** — markdown, DOCX, PDF, or LinkedIn export

## Workflow

```
┌──────────────────────────────────────────────────────────────┐
│  1. faang-resume-tailor                                      │
│     - Parse JD + source resume                               │
│     - Build targeting brief                                  │
│     - Apply 24-rule rulebook                                 │
│     - Surface gaps to user                                   │
│     - Produce tailored markdown draft                        │
└─────────────────────────┬────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────┐
│  2. faang-resume-verify (HARD GATES)                         │
│                                                              │
│     GATE 1: ATS Score        ≥ 95/100 required               │
│     GATE 2: JD Match Score   ≥ 90/100 required               │
│                                                              │
│     If FAIL → loop back to tailor with specific fixes        │
│     If PASS → proceed to export                              │
│     Max 5 loops; then escalate to user                       │
└─────────────────────────┬────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────┐
│  3. Export                                                   │
│     - ATS-safe DOCX (fonts embedded, no track changes)       │
│     - ATS-safe PDF (text-based, ≤ 1 MB, source-generated)    │
│     - Filename: Firstname-Lastname-Role.{pdf,docx}           │
│                                                              │
│  4. Recruiter-style review of the final result               │
│     - 6-second scan output                                   │
│     - Top 3 bullets identified                               │
│     - Suspect claims to defend in interview                  │
│     - Gaps that may come up in screen                        │
└──────────────────────────────────────────────────────────────┘
```

## Hard rules (no overrides)

- The plugin **refuses to export** unless both gates PASS
- Manual edits after PASS invalidate the score; re-verify required
- Maximum 5 tailor → verify cycles before escalation to user
- All three outputs (markdown, DOCX, PDF) ship together or not at all
