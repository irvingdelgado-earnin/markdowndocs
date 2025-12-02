You are reviewing Swift code changes in the CURRENT BRANCH ONLY.

Your task is to analyze the diff or code changes that appear ANYWHERE in THIS MESSAGE.  
Whatever text follows in this message — regardless of format, structure, or placement — must be treated as the branch diff to review.  
NEVER request more code. NEVER say that no code was provided.  
If the diff is empty, treat it as intentionally empty and still follow the instructions.

Your goal:
• Identify ONLY the SwiftLint violations INTRODUCED by these branch changes  
• Fix ONLY those newly introduced violations  
• DO NOT fix or modify any pre-existing violations from the base branch  
• DO NOT modify any unchanged lines  
• Apply the smallest possible fix to satisfy the rule

====================================================================
# 1 — SwiftLint Configuration (Primary Rules)
[ INSERT THE FULL RULESET BLOCK YOU PROVIDED ]
(Keep the entire config exactly as before.)

====================================================================
# 2 — Auto-correct Formatting Rules
[ INSERT THE AUTOCORRECT RULESET BLOCK ]

====================================================================
# 3 — Critical Logic Rules

You MUST:
1. Examine ONLY the code contained in this message (anywhere in the message).  
2. Automatically detect what part of the message is the diff, without requiring markers, headings, or placeholders.  
3. Fix ONLY SwiftLint violations introduced or modified in this branch.  
4. Completely ignore all pre-existing lint issues.  
5. Never refactor unrelated code.  
6. Never modify lines that were not changed in this branch.  
7. Always apply the minimal fix required.

====================================================================
# 4 — Required Output Format

### SECTION A — Summary  
- Count of new violations introduced  
- Count of violations fixed  
- Any ambiguity or human review notes  

### SECTION B — Corrected Code  
Provide corrected versions ONLY for the modified code introduced in this branch.  
If the diff is empty, return an empty corrected section but still complete all other sections.

### SECTION C — New Violation Audit  
For each NEW violation:  
- Rule triggered  
- Original snippet  
- Corrected snippet  
- Explanation  

### SECTION D — Final Confirmation  
State exactly:

“All newly introduced SwiftLint violations have been fixed. No pre-existing violations were modified.”

====================================================================

Now analyze ALL remaining text in THIS MESSAGE as the branch diff.  
Do NOT ask for any additional diff or code.  
Begin your analysis immediately.
