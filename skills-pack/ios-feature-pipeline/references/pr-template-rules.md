# PR Template Rules

## Purpose
Generate a reviewer-friendly PR description from the current branch changes.

## Core Rules
- Fill only sections relevant to the change.
- Remove irrelevant sections completely.
- Be specific and easy for reviewers to scan.
- Base content on actual branch changes, not assumptions.
- Do not duplicate product copy from Jira.
- Prefer implementation detail, behavioral impact, and reviewer-relevant context.

## Section Rules

### Jira Link
- Include only if known.
- Do not invent ticket numbers.

### Implementation Details
- Summarize what changed technically.
- Mention architecture decisions, important flows, flags, variables, dependency changes, and risks when relevant.
- Keep it concise but concrete.

### UI Changes
- Include only if the PR changes UI.
- Mention affected screens and user-visible behavior.
- Do not claim screenshots exist unless provided.

### Testing Details
- Include only relevant items.
- Mention actual testing performed or required follow-up.
- Keep VoiceOver, UI tests, TestPlans, and mock data notes only when they apply.

### Variable Details
- Include only if Firebase or Optimizely variables were added or changed.
- Mention default values when known.
- Do not invent variable names or defaults.

### Checklist Selection
Keep only one checklist group unless multiple truly apply:
- Feature PRs
- Bug fix PRs
- Chore PRs
- 3rd party SDK PRs
