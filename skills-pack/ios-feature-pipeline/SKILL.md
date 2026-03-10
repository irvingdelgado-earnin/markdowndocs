---
name: ios-feature-pipeline
description: Implements production-ready iOS changes through a phased workflow: requirement analysis, ambiguity checks, MVVM plus Coordinator planning, implementation, lint cleanup, test coverage, review, CODEOWNERS review analysis, and PR draft generation. Use when the user asks to build or change an iOS feature, generate Swift or SwiftUI code, refactor a flow, make a change production ready, fill a PR template from the current branch, or determine likely reviewers from the current diff.
compatibility: Claude Code or any environment with repository access. Best when git diff, CODEOWNERS, and project files are available.
metadata:
  author: Irving Delgado
  version: 1.0.0
  category: ios-engineering
  tags: [ios, swift, swiftui, mvvm, coordinator, feature-implementation, pr]
---

# iOS Feature Pipeline

## Purpose
Turn product, engineering, and design input into production-ready iOS implementation output using your preferred architecture, debugging style, and PR workflow.

## Critical Rules
- Default to MVVM plus Coordinator for new work unless the surrounding module clearly follows another established pattern that should be preserved.
- Keep business logic out of Views.
- Keep routing and flow ownership out of Views and out of direct routing logic in ViewModels.
- Prefer existing design system components over custom UI.
- If no strong design system match exists, call it out explicitly and ask for review.
- Never modify Xcode project files automatically.
- If files are created, list their paths and tell the user exactly which ones to add manually in Xcode.
- If intent is ambiguous or cannot be determined confidently, do not assume.
- State what is unclear and ask what needs clarification and from whom, such as Product, Design, Backend, Analytics, or iOS.
- `@Published` state must be updated on the main thread.
- Avoid force unwraps in production code.
- During debugging, temporary decisive debug prints are allowed. Use one consistent emoji marker per debugging session and remove those prints before final production-ready output.

## Workflow

### Phase 1 - Understand the request
- Restate the change in product and engineering terms.
- Identify the user flow, business rules, success criteria, affected layers, and likely risk areas.
- Identify any missing or ambiguous requirements.

### Phase 2 - Clarify before coding
- If behavior, UX, naming, ownership, analytics, or business logic is unclear, stop and ask concise clarification questions.
- For each unclear point, suggest the likely owner: Product, Design, Backend, Analytics, or iOS.
- Do not guess on product meaning, feature scope, or ownership boundaries.

### Phase 3 - Plan the implementation
- Map the work to MVVM plus Coordinator unless the surrounding module clearly follows another established pattern that should be preserved.
- Prefer existing design system components and established patterns already used in the module.
- Define:
  - files to create or modify
  - View responsibilities
  - ViewModel responsibilities
  - Coordinator responsibilities
  - service, GraphQL, REST, Swagger, and use-case boundaries
  - analytics, feature flags, and edge cases if relevant

### Phase 4 - Offers and promotions checks
When the task touches offers or promotions, determine:
- whether the offer is internal or third-party
- whether it is link-oriented or another offer type
- whether the offer is backend-driven or legacy for landing-page content
- which entry points are in scope
- whether the entry point is the discovery list or another placement
- whether the correct intermediate surface is expanded discovery card, landing page, or direct web view
- whether enrolled users should bypass the landing page and go directly to the web view
- whether discovery, home, carousel, and post-cashout all need updates
- whether routing action, landing page id, product id, backend product key, eligibility, experiment properties, analytics, and cashout requirement checks all need updates
- whether visibility or ranking logic is client-owned legacy behavior or backend-driven behavior
If any of the above is unclear, ask instead of assuming.

### Phase 5 - Implement
- Generate the smallest correct implementation that satisfies the requirement.
- Keep Views lightweight and declarative.
- Put presentation state and UI-facing behavior in the ViewModel.
- Put navigation and flow ownership in the Coordinator.
- Reuse existing abstractions and dependency injection boundaries when present.
- Never silently invent custom UI when a design system component is a reasonable fit.

### Phase 6 - Quality pass
- Fix readability, lint-style, and maintainability issues.
- Remove force unwraps.
- Reduce unnecessary complexity and duplication.
- Ensure UI-bound async updates are main-thread safe.
- Check Combine subscriptions and weak captures where relevant.
- If debugging was used, remove temporary emoji-tagged prints unless the user explicitly wants a diagnostic patch.

### Phase 7 - Test pass
- Add or update tests for behavioral changes.
- Cover happy path, error path, and meaningful edge cases.
- Prefer readable tests with Given, When, Then structure.
- Call out any behavior that cannot be verified confidently from the available context.

### Phase 8 - Review pass
- Review the result as if it were a PR.
- Check architecture boundaries, concurrency safety, testability, maintainability, analytics completeness, and rollout readiness.
- Identify remaining risks, open questions, and follow-up work.

### Phase 9 - Reviewer and approval analysis
Before generating the PR draft:
- inspect the current branch diff
- collect changed file paths
- read `.github/CODEOWNERS`
- determine which code owners match the changed paths
- summarize which teams or owners likely need approval or review
- call out uncertainty if ownership cannot be determined confidently from the CODEOWNERS rules
If CODEOWNERS matching is ambiguous:
- show the affected files
- show the likely matching owner patterns
- do not guess beyond the available rules

### Phase 10 - PR draft generation
When the user asks for production-ready output:
- generate a reviewer-friendly PR description based on the current branch changes
- fill only sections relevant to the current change
- remove irrelevant sections instead of leaving placeholders
- be specific, concise, and easy for reviewers to scan
- summarize implementation details from the actual code changes, not generic product copy
- mention UI changes only if the branch includes UI changes
- mention testing details only if they are relevant to the change
- include variable details only when Firebase or Optimizely variables were added or changed
- preserve only the relevant checklist group for the PR type: Feature PR, Bug fix PR, Chore PR, or 3rd party SDK PR
- do not invent screenshots, variables, tests, Jira IDs, reviewer names, or ship milestones if they are not known

## Final Output
Return, as applicable:
1. Clarifications needed
2. Implementation plan
3. Files created or modified
4. Code
5. Tests
6. Risks and follow-ups
7. Manual Xcode add steps for any new files
8. Likely reviewers or approval groups from CODEOWNERS
9. PR draft when production-ready output is requested

## References
Use these files when needed:
- `references/engineering-rules.md`
- `references/architecture-rules.md`
- `references/ui-rules.md`
- `references/clarification-policy.md`
- `references/pr-template-rules.md`
- `references/codeowners-review-rules.md`
- `references/debugging-rules.md`
- `assets/output-template.md`
- `assets/pr-template.md`
