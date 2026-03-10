# Engineering Rules

## Concurrency
- `@Published` properties must be updated on the main thread.
- Keep network calls off the main thread.
- Do not block the main thread with synchronous work.
- Avoid unsafe actor or thread assumptions.
- Do not mix `async/await` with unsafe main-queue scheduling inside continuations.

## Combine
- Store subscriptions correctly.
- Cancel stale subscriptions before replacing them when appropriate.
- Use weak captures in sinks unless a strong capture is clearly required and safe.

## Code Quality
- Avoid force unwraps in production code.
- Remove debug prints before final output unless explicitly requested for diagnosis.
- Follow established codebase patterns before introducing a new approach.
- Prefer small focused types and methods over large multi-responsibility objects.
- Prefer `private` or `internal` over `fileprivate` unless there is a clear need.

## Testing
- Behavioral changes should come with tests.
- Cover both happy path and negative path where relevant.
- Do not claim a fix is verified unless the behavior has actually been reasoned through or tested as requested.
