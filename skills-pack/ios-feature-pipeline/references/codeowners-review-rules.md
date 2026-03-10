# CODEOWNERS Review Rules

## Purpose
Infer likely reviewer and approval requirements from the current branch diff using `.github/CODEOWNERS`.

## Workflow
1. Collect changed file paths from the current branch diff.
2. Read `.github/CODEOWNERS`.
3. Match changed paths against CODEOWNERS patterns.
4. Determine the relevant owners for each changed file.
5. Aggregate owners into a reviewer or approval summary.

## Rules
- Base the result on actual changed files only.
- Use CODEOWNERS pattern matching, not guesswork.
- If multiple owners match, include all relevant owners.
- If ownership is ambiguous, say so explicitly.
- Do not invent team names not present in CODEOWNERS.
- Prefer grouping by team when CODEOWNERS entries are team-based.
- If a file has no matching owner, call that out.
