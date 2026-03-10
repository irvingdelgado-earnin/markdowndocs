---
name: offers-promotions-context
description: Provides business and technical context for the EarnIn offers and promotions domain in the cashout EWA app, including internal and third-party offers, entry points, routing, eligibility, backend-driven and legacy client-side behavior, REST and GraphQL integrations, getPromotions usage, analytics, and common implementation patterns for link-oriented offers. Use when the user asks to implement, review, debug, or plan work related to offers, promos, marketplace products, discovery placements, post-cashout upsells, promo coordinators, landing pages, or promotion entry points.
compatibility: Best paired with the ios-feature-pipeline and ios-pr-reviewer skills.
metadata:
  author: Irving Delgado
  version: 1.0.0
  category: product-context
  tags: [offers, promotions, marketplace, discovery, analytics, cashout]
---

# Offers Promotions Context

## Purpose
Provide the product, architecture, analytics, and implementation context needed for offers and promotions work before coding or review begins.

## Domain Rules
- Offers may be internal or third-party.
- Offers can appear in multiple entry points and must be implemented consistently across them when required.
- Some offer logic is legacy client-side and some is backend-driven. Do not assume ownership without checking.
- Some landing pages are backend-driven and some legacy ones are not.
- If an offer is opened from the discovery list, show the expanded discovery card rather than a landing page.
- Landing pages are shown only from other placements.
- If users are enrolled, usually skip landing pages and route directly to the web view.
- Data may come from REST APIs, Swagger-generated code, GraphQL, or getPromotions depending on the flow.
- Navigation and product flow should follow the established coordinator-based architecture.
- If entry-point coverage is unclear, ask which placements are in scope.

## Required References
- `references/glossary.md`
- `references/architecture-overview.md`
- `references/entry-points.md`
- `references/generic-link-oriented-offer-guide.md`
- `references/business-rules.md`
- `references/analytics-rules.md`
- `references/analytics-event-patterns.md`
- `references/open-questions.md`
