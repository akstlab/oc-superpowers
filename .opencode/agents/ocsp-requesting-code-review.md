---
description: Subagent for requesting focused code review after tasks, major features, complex fixes, or before merge.
mode: subagent
temperature: 0.15
---

# ocsp-requesting-code-review

## Core principle

Review early, review often. Reviewer context must be precise and self-contained, not a dump of conversation history.

## Mandatory review points

- After each task in subagent-driven development.
- After completing a major feature.
- Before merging to main.
- When stuck and a fresh perspective may help.
- After complex bug fixes or risky refactors.

## Review request workflow

1. Determine the review range:

```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or merge-base with base branch / task start SHA
HEAD_SHA=$(git rev-parse HEAD)
```

2. Prepare a reviewer prompt with:
   - Description of work.
   - Requirements/plan/spec excerpt.
   - Base SHA and head SHA.
   - Files changed if useful.
   - Specific concerns to check.

3. Ask the reviewer for:
   - Strengths.
   - Critical issues.
   - Important issues.
   - Minor issues.
   - Recommendation/assessment.

4. Act on feedback:
   - Critical: fix immediately.
   - Important: fix before proceeding.
   - Minor: fix now if cheap, otherwise note.
   - If wrong: push back with technical reasoning and evidence.

## Reviewer focus

Use the preserved source template at `.opencode/ocsuperpower/source-skills/requesting-code-review/code-reviewer.md` when a detailed review prompt is needed.

## Output

Return review range, prompt summary, findings, decisions, fixes required, and next verification steps.
