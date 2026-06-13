---
name: requirement-validation
description: "Requirement Validation workflow for reviewing requirement specifications against a golden standard. Use when user asks to validate requirements, audit requirement quality, perform spec review, check SHALL language, traceability, verifiability, compliance scope, or asks in Chinese terms like requirement validation, req review, xuqiu shenhe, guifan jiancha, Golden Standard check."
---

# requirement-validation

Validate requirement specifications using a consistent, auditable workflow.

## Quick start
- Use prompt: [.github/prompts/requirement-validation.prompt.md](.github/prompts/requirement-validation.prompt.md)
- Chinese report template: [.github/skills/requirement-validation/references/cn-review-template.md](.github/skills/requirement-validation/references/cn-review-template.md)
- English report template: [.github/skills/requirement-validation/references/en-review-template.md](.github/skills/requirement-validation/references/en-review-template.md)

## When to use
Use this skill when the user asks to:
- review requirement specification quality
- check compliance with a requirement-writing standard
- find ambiguity, non-testable statements, or weak traceability
- provide actionable audit recommendations

## Inputs expected
- Target requirement specification file(s)
- Reference standard file (for example executive summary or full standard)
- Optional scope (full doc, specific chapter, only FR/NFR, etc.)

## Validation criteria (baseline)
Apply these checks in order:
1. Normative language
- Mandatory requirements use SHALL.
- SHOULD/MAY are only non-normative unless explicitly intended.

2. Requirement typing
- BR, UR, FR, NFR, CON are clearly separated.
- Mixed types in one statement are flagged.

3. Syntax discipline
- Prefer single-statement atomic requirements.
- For complex conditions, use molecular/table form.
- Avoid unfinished placeholders such as "shall ...".

4. Quality gate (A-AMCCMTUV)
- Agreed
- Atomic or Molecular
- Complete
- Consistent
- Modifiable
- Traceable
- Unambiguous
- Verifiable

5. Traceability and IDs
- Stable unique IDs exist.
- Upstream source and downstream verification links exist.
- Duplicate IDs, placeholders, and unresolved aliases are flagged.

6. Verification quality
- Each normative requirement has objective pass/fail criteria.
- Acceptance criteria include condition, action, and measurable result.

7. Compliance statement quality
- External standard compliance is explicit.
- Scope (full/partial), version, and unsupported behavior are stated.

8. Notes governance
- Notes are clearly non-normative.
- No hidden requirements inside notes.

## Severity model
- High: blocks implementation/testing or creates audit risk.
- Medium: weakens consistency/maintainability and may cause rework.
- Low: style/readability issue without immediate delivery risk.

## Required output format
Return results in this structure:
1. Findings first, ordered by severity.
2. Each finding includes:
- rule violated
- impact
- file path and line reference
- concrete fix recommendation
3. Open questions/assumptions.
4. Optional short improvement plan (phased, smallest-first).

## Review workflow
1. Read reference standard and target spec.
2. Build a rule-to-evidence map.
3. Run pattern search for placeholders, weak modal verbs, duplicate IDs, and missing acceptance criteria.
4. Produce severity-ranked findings with citations.
5. Propose minimal, executable remediation steps.

## Fast heuristics
Use quick scans for these patterns:
- "should" in normative sections
- "shall ..." or other incomplete statements
- placeholder actor/ID text like "???", "TBD", "USxx"
- repeated requirement IDs
- broad multi-clause statements with multiple obligations
- requirements lacking measurable criteria

## Do and do not
Do:
- prioritize defects and audit risks over summaries
- provide line-anchored evidence
- keep recommendations implementation-ready

Do not:
- rewrite entire documents unless requested
- treat notes as requirements
- hide critical findings behind high-level summaries

## Output template assets
- Chinese template: [.github/skills/requirement-validation/references/cn-review-template.md](.github/skills/requirement-validation/references/cn-review-template.md)
- English template: [.github/skills/requirement-validation/references/en-review-template.md](.github/skills/requirement-validation/references/en-review-template.md)
- Prompt entry: [.github/prompts/requirement-validation.prompt.md](.github/prompts/requirement-validation.prompt.md)
