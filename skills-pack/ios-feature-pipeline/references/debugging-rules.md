# Debugging Rules

## Temporary Decisive Prints
- When debugging behavior, prefer temporary, high-signal log statements over slow breakpoint-heavy workflows when appropriate.
- Use a single consistent emoji marker per debugging session so logs are easy to filter in the console.
- Keep messages short, decisive, and state-oriented.
- Log before and after critical transitions.
- Log actor or thread context when debugging concurrency issues.
- Remove all temporary debug prints before final production-ready output.

## Hypothesis-Driven Logging
- Debug prints should help prove or eliminate a hypothesis.
- Do not add noisy or speculative logs.
- Place logs at decision boundaries, not everywhere.
- Prefer one hypothesis at a time.
