---
name: swiftlint-new-violations-fixer
description: Detects and fixes only SwiftLint violations newly introduced on the current branch by comparing changed Swift files against the base branch using real SwiftLint JSON output and git diff line data. Use when the user asks to fix branch-only SwiftLint violations, clean up newly introduced lint issues, generate a patch for current-branch lint errors, or verify that changed Swift code is lint-clean without touching pre-existing violations.
compatibility: Claude Code or any environment with git and SwiftLint available.
metadata:
  author: Irving Delgado
  version: 1.0.0
  category: ios-engineering
  tags: [swiftlint, swift, lint, patch, git]
---

# SwiftLint New Violations Fixer

## Purpose
Automatically detect and fix only SwiftLint violations introduced by the current branch.

## Critical Rules
- Use real SwiftLint JSON output as the only source of truth for violations.
- Never infer or guess violations.
- Never fix pre-existing violations.
- Never modify unrelated code.
- Apply the smallest possible change.
- Do not refactor unless required to resolve the specific violation safely.

## Workflow

### Step 1 - Detect base branch
Determine the default base branch.

### Step 2 - Collect changed Swift files
Get changed Swift files relative to the base branch.

### Step 3 - Run SwiftLint
Run SwiftLint on changed files and collect JSON output.

### Step 4 - Identify newly introduced violations
Treat a violation as new only if:
1. It is on a modified or added line in the branch
2. It was not already present in the base version of the file
Ignore:
- unchanged lines
- pre-existing violations
- unchanged files
- violations outside the diff region

### Step 5 - Fix only new violations
Make minimal targeted fixes only.

### Step 6 - Present patch
Return:
- summary
- unified patch if needed

### Step 7 - Verify
Rerun SwiftLint and ensure no newly introduced violations remain.

## References
Use these files when needed:
- `references/violation-policy.md`
- `assets/patch-output-template.md`
