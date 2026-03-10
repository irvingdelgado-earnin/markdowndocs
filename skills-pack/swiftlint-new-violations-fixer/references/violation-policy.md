# Violation Policy

## Allowed Scope
- Changed Swift files only
- New violations only
- Minimal patch only

## Forbidden
- Fixing baseline violations
- Refactoring unrelated code
- Reformatting unrelated areas
- Expanding scope outside the diff

## Decision Rule
If a fix would risk changing behavior, do not apply it silently.
Report it as remaining work instead.
