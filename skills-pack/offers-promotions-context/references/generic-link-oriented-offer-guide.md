# Generic Link-Oriented Offer Guide

Use Previ-style implementation as the reference model for a generic link-oriented marketplace offer.

## What usually must be aligned
- Promo routing action
- Landing page id
- Product id for discovery
- Backend product key for marketplace web view
- Eligibility and experiment properties
- Coordinator and DI registration
- Discovery page or list wiring
- Discovery tile wiring
- Legacy home promo card wiring when relevant
- CXP promo carousel wiring when relevant
- Post-cashout routing when relevant
- Cashout requirement checks
- Analytics across placements and states

## Important flow rules
- Discovery list opens the expanded discovery card, not the landing page.
- Other placements may open the landing page for not-enrolled users.
- Enrolled users usually skip onboarding surfaces and open the web view directly.
- Not every offer is backend-driven in the same way. Confirm whether landing page ownership is backend-driven or legacy.

## Implementation Checklist
1. Define offer identity across routing, landing page id, discovery product id, and backend product key.
2. Define eligibility, enrollment checks, and experiment properties.
3. Add or wire the offer coordinator.
4. Update every in-scope entry point.
5. Update analytics across all in-scope surfaces.
6. Update cashout requirement logic when needed.
7. Confirm feature flags, variables, and backend configuration requirements.
