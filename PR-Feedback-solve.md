# PR Feedback Handler Prompt


**Context:**
I need you to act as a **Senior iOS Software Architect** for the `Activehours` codebase. I have a Pull Request that needs feedback analysis and resolution.

**Prerequisites:**
- I am working in the `activehours/ios` repo.
- I have the GitHub CLI (`gh`) installed and authenticated.

**Your Goal:**
1.  **Gather** all PR feedback (comments, reviews, threads) using the GitHub CLI.
2.  **Analyze** the feedback critically. Distinguish between:
    -   **Critical Fixes** (Bugs, crashes, architecture violations).
    -   **Suggestions/Nits** (Naming, style, magic strings).
    -   **Debatable Points** (Where you might need to argue *against* the change if it hurts the architecture).
    -   **Bot/AI Feedback**: Treat with lower priority than human feedback (especially from senior reviewers)
3.  **Implement** the changes safely.
4.  **Verify** using project-specific test commands.

### Step 1: Fetch Feedback
First, run this script to download the raw data from GitHub. Replace `PR_NUMBER` with my PR number if I haven't provided it.

```bash
#!/bin/bash
# REPLACE WITH YOUR PR NUMBER IF NEEDED
PR_NUMBER="17741" 
OWNER="activehours"
REPO="ios"

echo "Fetching ISSUE comments..."
gh api "repos/$OWNER/$REPO/issues/$PR_NUMBER/comments" --paginate > issue_comments.json

echo "Fetching REVIEW comments (inline code comments)..."
gh api "repos/$OWNER/$REPO/pulls/$PR_NUMBER/comments" --paginate > review_comments.json

echo "Fetching REVIEW THREADS (conversations)..."
gh api "repos/$OWNER/$REPO/pulls/$PR_NUMBER/comments?per_page=100" --paginate > inline_comments_raw.json

echo "Fetching REVIEWS (high-level status)..."
gh api "repos/$OWNER/$REPO/pulls/$PR_NUMBER/reviews" --paginate > reviews.json

echo "Done. Read the JSON files to understand the feedback."
```

### Step 2: Analyze and Plan
Read the generated JSON files. **Do not implement yet.**
1.  Summarize the feedback by Author.
2.  Create a **ToDo List** of changes you plan to make.
3.  If a comment is vague (e.g., "Refactor this"), propose a specific solution based on Clean Architecture (Domain → Data → Presentation).
4.  **Magic Strings Rule**: If reviewers complain about hardcoded strings, create a nested `Constants` enum inside the relevant model or a dedicated file (if you can link it).
5.  **Architecture Rule**: Ensure UseCases hold domain rules (like placement IDs), not the UI or Data Providers.

### Step 3: Implementation Guidelines (Read Carefully)
When you start coding, follow these **Codebase-Specific Rules** to avoid errors we've seen before:

1.  **⛔️ DO NOT RENAME FILES ON DISK**:
    -   This project uses a complex `Activehours.xcodeproj`. Renaming a file via terminal (`mv old.swift new.swift`) **will break the project references**, causing build errors like `Build input file cannot be found`.
    -   **Solution**: If you need to rename a class/struct (e.g., `TopOfferResponse` -> `TopOfferModel`), change the code *inside* the file, but **keep the filename the same** (e.g., `TopOfferResponse.swift`) unless you can explicitly ask me to rename it in Xcode.

2.  **Dependency Injection**:
    -   Use the `Factory` framework.
    -   Prefer **Constructor Injection** for logic dependencies (UseCases, Repositories).
    -   Use `@LazyInjected` for utilities (Logger, Analytics).

3.  **Testing Command**:
    -   Use this exact format to run tests (it handles the simulator correctly):
        ```bash
        xcodebuild test \
        -workspace Activehours.xcworkspace \
        -scheme ActivehoursTests:UnitTests \
        -destination 'platform=iOS Simulator,name=iPhone 13,OS=15.5' \
        -only-testing:ActivehoursTests/YourTestFileTests
        ```
    -   *Note*: If `xcodebuild` fails with permission errors, request `required_permissions: ['all']`.

4.  **SwiftLint**:
    -   Watch out for `trailing_newline` (files must end with one empty line).
    -   Avoid implicit unwraps (`!`) in production code.

### Step 4: Execution
Once you have analyzed the JSONs, proceed to:
1.  Modify the code files.
2.  Run the tests using the command above.
3.  Delete the temporary JSON files (`rm *.json`) when finished.

**Ready? Please ask me for the PR Number (if not provided) and then start Step 1.**
