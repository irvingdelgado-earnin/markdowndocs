# LLM iOS Development Rules - Non-Negotiable

## 1. Threading & Concurrency

### Rule 1.1: `@Published` properties MUST be updated on the main thread

// ❌ FORBIDDEN - Never do this
Task {
    let result = await networkCall()
    self.state = .content  // @Published from background thread
}

// ✅ REQUIRED - Always dispatch UI updates to main
Task {
    let result = await networkCall()
    await MainActor.run {
        self.state = .content
    }
}### Rule 1.2: Never mix `async/await` with `DispatchQueue.main` inside continuations

// ❌ FORBIDDEN - Deadlock risk
await withCheckedContinuation { continuation in
    somePublisher
        .receive(on: DispatchQueue.main)  // Will deadlock if caller is on main
        .sink { continuation.resume(returning: $0) }
}### Rule 1.3: Keep network calls on background, only UI updates on main

- Network operations: Background thread
- `@Published` mutations: Main thread via `MainActor.run`
- Never block the main thread with synchronous network calls

---

## 2. SwiftUI & Observation

### Rule 2.1: Verify `@ObservedObject` lifecycle

- `@ObservedObject` does NOT own the object
- Ensure the ViewModel is retained (use `@StateObject` for ownership or DI with `.shared` scope)
- Never create ViewModels inline as default parameter values if they can be recreated

### Rule 2.2: State changes must be atomic and complete before SwiftUI observes

- All related `@Published` properties should be updated in a single `MainActor.run` block
- Never update multiple `@Published` properties across different dispatch contexts

---

## 3. Combine Subscriptions

### Rule 3.1: Store subscriptions properly to prevent memory leaks

// ❌ FORBIDDEN - Subscription immediately deallocated
publisher.sink { ... }

// ✅ REQUIRED - Store the cancellable
cancellable = publisher.sink { ... }### Rule 3.2: Cancel previous subscriptions before creating new ones

private var subscription: AnyCancellable?

func refresh() {
    subscription?.cancel()
    subscription = nil
    subscription = publisher.sink { ... }
}### Rule 3.3: Use `[weak self]` in all Combine sinks

.sink { [weak self] value in
    guard let self else { return }
    // ...
}---

## 4. Code Quality

### Rule 4.1: Never leave debug prints in production code

- All `print()` statements used for debugging MUST be removed before PR
- Use structured logging (`logger.notice`, etc.) for production logs

### Rule 4.2: Understand existing patterns before implementing

- Read existing implementations in the codebase (e.g., `extractValue()` in `Loadable.swift`)
- Follow established patterns, don't invent new ones

### Rule 4.3: Verify Equatable implementations when using `==` with enums

- Check custom `==` operators for enums with associated values
- Don't assume associated values are compared unless verified

---

## 5. Testing & Verification

### Rule 5.1: Always test on device/simulator before claiming fix

- Don't just compile - run the app
- Test the exact scenario that was broken
- Reproduce edge cases (rapid refresh, app launch, etc.)

### Rule 5.2: Use debug prints strategically when diagnosing issues

- Log state transitions with clear identifiers
- Log before AND after critical operations
- Log thread/actor context when debugging threading issues

### Rule 5.3: Check Instruments for performance issues

- Allocations: Memory leaks, excessive allocations
- Time Profiler: CPU hotspots
- SwiftUI View Updates: Excessive redraws

---

## 6. Problem Solving

### Rule 6.1: Understand root cause before implementing fix

- Don't guess - use logs, breakpoints, Instruments
- Trace the full execution path
- Identify exactly WHERE and WHY the issue occurs

### Rule 6.2: One fix at a time

- Don't combine multiple changes
- Test each fix independently
- Revert if fix doesn't work, don't layer more changes

### Rule 6.3: Consider all callers and edge cases

- Check who calls the method being modified
- Consider rapid successive calls
- Consider error states and early exits

---

## 7. PR & Documentation

### Rule 7.1: Frame changes appropriately

- Enhancements: Focus on improvements, benefits, better architecture
- Bug fixes: Be clear about what was broken and why
- Don't expose internal issues unnecessarily in public-facing descriptions

### Rule 7.2: Update tests for all behavioral changes

- If behavior changes, tests must verify new behavior
- Mock dependencies correctly for async flows
- Cover error paths, not just happy paths

---

## Summary: The 3 Cardinal Rules

1. **`@Published` = Main Thread ONLY** - No exceptions
2. **Understand before implementing** - Read existing code, verify assumptions
3. **Test the actual behavior** - Don't trust "it compiles"
