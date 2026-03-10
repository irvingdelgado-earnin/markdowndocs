---
name: ios-pr-reviewer
description: Performs a principal-level iOS pull request review using PR diff analysis, architecture checks, test quality review, code smell detection, offers-domain checks, and ready-to-paste GitHub review comments. Use when the user asks to review an iOS PR, analyze a Swift pull request, generate review comments, assess architecture or test coverage in a PR, or do a professional code review for iOS changes.
compatibility: Claude Code or any environment with repository and PR diff access. Works best with git or GitHub CLI access.
metadata:
  author: Irving Delgado
  version: 1.0.0
  category: ios-engineering
  tags: [ios, swift, pr-review, architecture, testing, github]
---

# iOS PR Reviewer

## Purpose
Review iOS pull requests like a principal engineer and return actionable, paste-ready feedback.

## Critical Rules
- Base findings on the actual PR diff and relevant surrounding code.
- Prefer high-signal comments over comment spam.
- Flag violations of MVVM plus Coordinator expectations where relevant.
- Flag custom UI when an existing design system component should likely be used.
- Do not invent certainty where the diff lacks context.
- If something is unclear, state the uncertainty explicitly.
- Prioritize correctness, architecture, threading, safety, analytics completeness, and test quality over style nits.

## Workflow

### Step 1 - Get the PR number
If no PR number is given, ask for it.

### Step 2 - Fetch PR data
Use the available repo tooling to fetch:
- PR metadata
- PR diff
- existing review comments
- issue comments
- reviews

### Step 3 - Analyze the diff
Review:
- intent of the PR
- correctness and regression risk
- architecture boundaries
- concurrency or thread-safety issues
- test quality and missing coverage
- design system adherence
- code smells and maintainability concerns

### Step 4 - Offers and promotions domain checks
When the PR touches offers or promotions, check:
- whether all in-scope entry points were updated
- whether discovery-list behavior shows expanded discovery card instead of landing page
- whether enrolled users bypass landing pages and go directly to web view when appropriate
- whether backend-driven versus legacy landing-page ownership was respected
- whether routing action, product id, landing page id, backend product key, eligibility, analytics, and cashout gating were updated consistently

### Step 5 - Severity classification
Classify findings as:
- Critical Issues
- Major Issues
- Moderate Issues
- Minor Issues or Nits

### Step 6 - Generate review output
Return:
- PR summary
- severity-grouped findings
- test quality review
- PR-level feedback
- ready-to-paste GitHub review comments

## References
Use these files when needed:
- `references/review-rubric.md`
- `references/architecture-review-rules.md`
- `references/test-review-rules.md`
- `references/offers-review-rules.md`
- `assets/github-review-template.md`
