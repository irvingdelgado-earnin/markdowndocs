You are operating inside Cursor and you have full ability to run shell commands, read and modify files, generate patches, and apply changes.

Your mission:
AUTOMATICALLY detect and fix ONLY the SwiftLint violations introduced by the current branch—never pre-existing ones—using REAL SwiftLint output, not inference.

You must ALWAYS follow this workflow:

====================================================================
# STEP 1 — DETECT THE BASE BRANCH
Automatically determine the default base branch using:

git symbolic-ref refs/remotes/origin/HEAD | sed 's@refs/remotes/origin/@@'

Call the base branch: BASE_BRANCH.

====================================================================
# STEP 2 — COLLECT CHANGED SWIFT FILES
Gather the list of Swift files changed in this branch:

git diff --name-only ${BASE_BRANCH}...HEAD -- '*.swift'

If none are found, proceed with empty analysis but NEVER ask the user for content.

====================================================================
# STEP 3 — RUN SWIFTLINT ON CHANGED FILES
For each changed file, run:

swiftlint lint --reporter json --path <FILE>

Gather all outputs into a combined JSON structure.

This REAL SwiftLint JSON output is the ONLY valid source of lint violation data.
You must NEVER infer or guess lint violations.

====================================================================
# STEP 4 — IDENTIFY WHICH VIOLATIONS ARE *NEW*
A violation is considered NEW when BOTH conditions are true:

1. The violation appears on a line that was modified or added in this branch.
   Use:
      git diff ${BASE_BRANCH}...HEAD -U0 <FILE>
   to detect modified/added lines precisely (line-level).
2. The violation did NOT exist in the base version of the file.
   Compare against:
      git show ${BASE_BRANCH}:<FILE>
   or ensure the violation location corresponds to changed context.

You MUST ignore:
- Violations on unchanged lines
- Violations already present in the base branch
- Violations outside the diff region
- Violations in files that didn't change

====================================================================
# STEP 5 — FIX ONLY NEWLY INTRODUCED VIOLATIONS
For each new violation:
- Apply the smallest possible fix.
- Produce a unified diff patch.
- Do NOT modify unrelated code.
- Do NOT refactor.
- Do NOT reformat unless required by the rule.

If a file contains no new violations, do not output a patch for that file.

====================================================================
# STEP 6 — APPLY PATCHES WHEN APPROVED
When your patch is ready, present it clearly and in unified diff format.
If I say "apply", you must apply the patch using Cursor’s applyChanges or patch tool.

====================================================================
# STEP 7 — VERIFY CLEANLINESS
After applying patches, rerun SwiftLint on the changed files.
If any NEW violations remain, repeat fixes until zero remain.

Do NOT stop until:
- All newly introduced violations are fixed
- No pre-existing lines were modified
- SwiftLint reports success

====================================================================
# STEP 8 — OUTPUT FORMAT
Always output these sections:

### SECTION A — Summary
- Number of changed files
- Number of newly introduced violations
- Number of fixes applied

### SECTION B — Unified Patch (ONLY if fixes are needed)
