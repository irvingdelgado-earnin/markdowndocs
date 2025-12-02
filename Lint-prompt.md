You are operating inside Cursor, which gives you the ability to run shell commands.

Your task is to review ONLY the Swift code changes introduced in the current branch and ensure that NO new SwiftLint violations are added. You must NOT fix or modify any existing lint violations from the base branch.

====================================================================
# STEP 1 — GATHER THE DIFF AUTOMATICALLY

Run the following command to collect all Swift changes against the base branch:

git diff main...HEAD -- '*.swift'

If the project uses a different primary branch (e.g., develop), automatically detect it using `git symbolic-ref refs/remotes/origin/HEAD` and substitute accordingly.

Use the output of the git diff as the SOLE source of truth for code changes.

Do NOT ask me to paste a diff.
Do NOT say that no diff was provided.
Do NOT request additional input.
You must gather the diff yourself by running the command above.

====================================================================
# STEP 2 — SWIFTLINT RULESET (STRICT — DO NOT DEVIATE)

Use the following full SwiftLint rules as the authoritative configuration when evaluating violations:

[INSERT FULL RULESET EXACTLY AS PROVIDED]

Auto-correct formatting rules (secondary pass):

[INSERT AUTOCORRECT RULESET EXACTLY AS PROVIDED]

====================================================================
# STEP 3 — CRITICAL LOGIC: APPLY RULES **ONLY TO NEW VIOLATIONS**

You must follow these constraints:

1. Examine ONLY the lines modified in the git diff.
2. A violation should be fixed ONLY IF:
   • It appears in a line added or modified by this branch, AND  
   • The violation did NOT exist in the base branch.
3. Pre-existing violations MUST be ignored.
4. Unchanged code MUST NOT be modified.
5. Make the smallest possible fix to satisfy the lint rule.
6. Do not refactor or improve code unless required by the lint rule.

====================================================================
# STEP 4 — REQUIRED OUTPUT FORMAT

### SECTION A — Summary
- Number of newly introduced violations
- Number of violations fixed
- Notes on any ambiguous cases

### SECTION B — Corrected Patch (Unified Diff)
Provide the corrected version of ONLY the modified code, formatted as a unified diff ready to apply with `git apply`.

### SECTION C — New Violation Audit
For EACH violation introduced by this branch:
- Rule name
- Original snippet
- Corrected snippet
- Clear explanation of the issue and fix

### SECTION D — Confirmation
State exactly:

“All newly introduced SwiftLint violations have been fixed. No pre-existing violations were modified.”

====================================================================

BEGIN NOW:
1. Run the git diff command above.
2. Analyze the results.
3. Produce the output exactly as instructed.
