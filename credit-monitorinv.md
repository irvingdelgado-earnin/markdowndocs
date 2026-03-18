# Credit Monitoring: acceptTermsAndConditions 500 Errors — Investigation Summary

## Problem Statement

Over a ~2-week window, iOS clients generated **3,204 HTTP 500 errors** on
`POST /credit-monitoring/external/acceptTermsAndConditions`.
Android clients generated **~39 errors** for the same endpoint in the same period.

## Root Cause

Every single 500 is caused by the same scenario: **an already-enrolled user
re-entering the Credit Monitoring enrollment flow and calling acceptTermsAndConditions again.**

The backend's internal service correctly identifies this and throws a 409
(`"Cannot accept Terms and Conditions again; user already enrolled"`), but the
external service has a gRPC-to-HTTP mapping bug that surfaces it as a generic 500.

**Confirmed via Datadog trace** (trace ID: `69a6744c00000000e0087b0da18c2e58`):

```
iOS → linkerd → service-credit-monitoring-external (HTTP 500)
                  → gRPC INTERNAL
                    → service-credit-monitoring-internal
                      → DynamoDB query (credit-monitoring-enrollment table)
                        → user already enrolled
                          → throws HttpStatusException(409)
                            → gRPC wraps as INTERNAL
                              → external maps INTERNAL → HTTP 500 ← the bug
```

## Why the error count is so different between platforms

| Platform | Error count | Reason |
|----------|-------------|--------|
| Android (~39) | 1 call per user | Ignores the error and proceeds — no retry |
| iOS (~3,204) | Dozens per user | `skipTermsAndConditions()` has an infinite retry loop on any error |

The **real user impact is ~39 users across both platforms.** iOS inflates it ~80x
through its retry mechanism.

## Why do enrolled users re-enter the enrollment flow?

The enrollment is a multi-step process:

```
Step 1: acceptTermsAndConditions → writes onBoardingStatus = InProgress, enrollmentStatus = NotEnrolled
Step 2: KYC verification
Step 3: enrollUser → writes enrollmentStatus = Enrolled
```

If a user completes all steps but something causes the app to re-evaluate eligibility
(new session, deep link, stale in-memory state), `GET /external/status` can return a
state where `onBoardingStatus = InProgress` — which makes the client show the enrollment
card again. The user re-enters the flow, and `acceptTermsAndConditions` is called for
a user that is already enrolled in DynamoDB → 409 (surfaced as 500).

This is a rare edge case (~39 users out of the full Credit Monitoring user base over
2 weeks), but the iOS retry loop amplifies it into a noisy error.

## Proposed Fix

### Backend (prerequisite)

Return **HTTP 409** instead of 500 when the user is already enrolled.

The internal service already throws `HttpStatusException(409)`, but the external service's
`acceptTermsAndConditions` route doesn't call `toHttpStatusException()` to convert the
gRPC error — unlike other routes (GetStatus, GetDashboard, etc.) that do.

Fix: use `toHttpStatusException()` in the `acceptTermsAndConditions` route handler,
consistent with other handlers in `CreditMonitoringExternalHandler.kt`.

### iOS (after backend ships 409)

In `CreditMonitoringOnboardingExplainer.swift`, update both code paths:

**`skipTermsAndConditions()` (product discovery — auto-fires on init):**

| Error code | Behavior |
|------------|----------|
| 409 | Log "user already enrolled, should not be in this flow" → stop retrying → dismiss |
| Other | Retry with a max cap (e.g., 3 attempts) instead of infinite |

**`next()` (homescreen — user taps "Get my score"):**

| Error code | Behavior |
|------------|----------|
| 409 | Log "user already enrolled" → dismiss the flow |
| Other | Show error to user (currently silently strands them on the T&C screen) |

### Android (optional hardening)

Android already "works" by accident (ignores the error), but should explicitly check
for 409 and dismiss the flow rather than silently swallowing all errors.

## Summary

| Layer | Issue | Fix | Priority |
|-------|-------|-----|----------|
| Backend | gRPC INTERNAL → HTTP 500 instead of 409 | Use `toHttpStatusException()` in acceptTermsAndConditions route | P1 — unblocks client fixes |
| iOS | Infinite retry loop on any error | Check for 409 (stop + dismiss) vs other errors (retry with cap) | P1 — eliminates error noise |
| iOS | Silent strand on `next()` failure | Handle error visibly or dismiss on 409 | P2 |
| iOS | No client-side observability | Add logging with error code and context | P2 |
| Android | Fire-and-forget ignores all errors | Explicitly handle 409 | P3 — works by accident today |

## Datadog References

- [Traces Explorer (iOS 500s)](https://app.datadoghq.com/apm/traces?query=env%3Aprod+operation_name%3Anetty.request+service%3Aservice-credit-monitoring-external+%40span.kind%3Aserver+-resource_name%3A%28%22POST+%2Fexternal%2FenrollUser%22+OR+%22GET+%2Fexternal%2Fdashboard%22+OR+%22GET+%2Fexternal%2F%22+OR+%22GET+%2Fexternal%2Fstatus%22+OR+%22GET+%2Fexternal%2Fcredit-factor%2Ftotal-accounts%22+OR+%22GET+%2Fexternal%2Fcredit-factor%2Fpayments%22+OR+%22GET+%2Fexternal%2Fcredit-factor%2Fcredit-utilization%22+OR+%22GET+%2Fcontrol%2Fhealth%22+OR+%22GET+%2Fexternal%2Fcredit-factor%2Fhard-inquiries%22%29+%40http.useragent%3A%2AiOS%2A+%40http.status_code%3A500)
- Full trace confirming "already enrolled": `69a6744c00000000e0087b0da18c2e58`
