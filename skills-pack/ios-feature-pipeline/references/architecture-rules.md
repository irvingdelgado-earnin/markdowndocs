# Architecture Rules

## Default Pattern
Use MVVM plus Coordinator for new work unless the surrounding module has a strong established architecture that should be preserved.

## View
- Renders state
- Sends user intents
- Does not own business logic
- Does not own navigation flow
- Does not instantiate services directly

## ViewModel
- Owns presentation state
- Handles user intents
- Uses abstractions for domain or service interactions
- May emit navigation intents
- Does not directly perform routing or build destination screens

## Coordinator
- Owns navigation flow
- Builds screens and wires dependencies
- Handles route decisions and child flows

## Boundaries
- Business rules belong outside Views.
- Navigation flow belongs outside Views.
- Avoid leaking network DTOs directly into UI-facing layers.
