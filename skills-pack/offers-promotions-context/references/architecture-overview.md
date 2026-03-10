# Architecture Overview

## High-Level Flow
Entry points route into the appropriate coordinator layer, which then drives landing page, eligibility, and web-view behavior.

## Entry Points
- Discovery icon or tile
- Discovery page or list
- Legacy home promo card
- CXP home promo carousel
- Post-cashout upsell

## Routing
- Discovery surfaces use discovery navigation delegates and product coordinators.
- Promotions surfaces use the promo coordinator.
- Post-cashout uses the cross-sell coordinator.

## Product Flow
A generic link-oriented offer typically includes:
- product identity setup
- routing action
- product id and landing page id wiring
- eligibility and enrollment checks
- URL fetching
- optional landing page or expanded discovery card
- web view presentation
- analytics
- DI registration
