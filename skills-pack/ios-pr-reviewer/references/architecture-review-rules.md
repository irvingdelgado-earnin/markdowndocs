# Architecture Review Rules

- Business logic should not live in Views.
- Navigation flow should not be implemented directly in Views.
- ViewModels should not directly perform routing when a Coordinator should own it.
- New work should generally align to MVVM plus Coordinator unless the local module pattern clearly differs.
- Prefer existing dependency boundaries and established factory patterns.
- Flag custom UI when a design system component appears to be the better fit.
- Call out architectural drift instead of silently accepting hybrid patterns.
