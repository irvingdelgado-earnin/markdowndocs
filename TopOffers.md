# AI Offer Card - Backend Response Mapping

## Complete Backend Response Structure

Based on existing promotions API patterns, here's the expected response structure:

```json
{
  "offers": [
    {
      "listingId": "auto-insurance-top-offer",
      "listingVersion": 1,
      "isEligible": true,
      "category": "auto insurance",
      "offerActions": [
        {
          "actionContext": "VIEW_ELIGIBLE_OFFER_DETAILS",
          "actionType": "OFFER_ACTION_TYPE_APP_NAVIGATION",
          "url": "earnin://auto-insurance?campaign=top-offer-1"
        },
        {
          "actionContext": "ENROLL_OFFER",
          "actionType": "OFFER_ACTION_TYPE_WEBVIEW",
          "url": "https://api.earnin.net/marketplace-external/external/v1/auto-insurance-webview"
        }
      ],
      "templates": [
        {
          "templateId": "top-offer-1",
          "viewElements": {
        "title": {
          "value": "You may be overpaying for car insurance. See how much you can save.",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "cta-text": {
          "value": "Save on car insurance",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "card-icon": {
          "value": "https://d3jlszua3q4cyr.cloudfront.net/content/mobile/promos/auto_insurance/car_illustration.svg",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "header-text": {
          "value": "RECOMMENDED",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "header-icon": {
          "value": "recommended-badge-icon",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
            "cta-context-action": {
              "value": "VIEW_ELIGIBLE_OFFER_DETAILS",
              "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
            },
        "border-primary-color": {
          "value": "#4161e4",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "border-secondary-color": {
          "value": "#4161e4",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "border-tertiary-color": {
          "value": "#4161e4",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "badge-gradient-start": {
          "value": "#4161e4",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "badge-gradient-end": {
          "value": "#a544ee",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        },
        "cta-button-border-color": {
          "value": "#4161e4",
          "source": "VIEW_ELEMENT_SOURCE_CONSTANT"
        }
          }
        }
      ]
    }
  ]
}
```

## Mapping: Backend Response → UI Model Properties

### Offer-Level Properties

| Backend Property | UI Model Property | Source | Notes |
|------------------|-------------------|--------|-------|
| `offers[].listingId` | `campaignId` | Offer level | Campaign/listing identifier |
| `offers[].isEligible` | Used for `isCtaEnabled` | Offer level | User eligibility status |
| `offers[].offerActions[]` | `ctaActionUrl` | Offer level | Matched by `cta-context-action` |

### Template ViewElements Properties

| Backend ViewElement Key | UI Model Property | Required | Default Value | Notes |
|------------------------|-------------------|----------|---------------|-------|
| `title` | `title` | ✅ Yes | None | Main card title text |
| `cta-text` | `ctaButtonText` | ✅ Yes | None | CTA button text |
| `card-icon` | `carIllustrationUrl` | ⚠️ Optional | `nil` | URL to car illustration image |
| `header-text` | `recommendedBadgeText` | ⚠️ Optional | `nil` | Badge text. If present, badge will be shown |
| `header-icon` | N/A | ⚠️ Optional | None | Icon identifier for badge (for future use) |
| `cta-context-action` | N/A | ✅ **Yes** | None | Matches against `offerActions[].actionContext` to find URL |
| `border-primary-color` | `borderColors.primaryColor` | ⚠️ Optional | `"#4161e4"` | Hex color for innermost border |
| `border-secondary-color` | `borderColors.secondaryColor` | ⚠️ Optional | `"#4161e4"` | Hex color for middle border (with blur) |
| `border-tertiary-color` | `borderColors.tertiaryColor` | ⚠️ Optional | `"#4161e4"` | Hex color for outermost border (with blur) |
| `badge-gradient-start` | `recommendedBadgeGradient.startColor` | ⚠️ Optional | `"#4161e4"` | Gradient start color for badge text |
| `badge-gradient-end` | `recommendedBadgeGradient.endColor` | ⚠️ Optional | `"#a544ee"` | Gradient end color for badge text |
| `cta-button-border-color` | `ctaButtonBorderColor` | ⚠️ Optional | `"#4161e4"` | Hex color for CTA button border |

## How CTA Action URL is Resolved

The `ctaActionUrl` is **NOT** directly in viewElements. Instead:

1. Extract `cta-context-action` value from viewElements (e.g., `"VIEW_ELIGIBLE_OFFER_DETAILS"`)
2. Find matching `offerAction` in `offerActions[]` where `actionContext == cta-context-action`
3. Use the matched action's `url` as `ctaActionUrl`

Example:
- `viewElements["cta-context-action"].value` = `"VIEW_ELIGIBLE_OFFER_DETAILS"`
- Match against `offerActions[]` array
- Use `offerActions[].url` where `actionContext == "VIEW_ELIGIBLE_OFFER_DETAILS"`

## Client-Side Derived Properties

These properties are **NOT** directly in the response - they are derived from content and business logic:

| Property | Derivation Logic |
|----------|------------------|
| `showRecommendedBadge` | `true` if `header-text` exists in viewElements, `false` otherwise |
| `isCtaEnabled` | `true` if matching `offerAction.url` exists AND `offers[].isEligible == true`, `false` otherwise |
| `recommendedBadgeText` | Uses `header-text` value if present, otherwise defaults to `"RECOMMENDED"` if badge should show |
| `ctaActionUrl` | Resolved from `offerActions[]` by matching `actionContext` with `cta-context-action` |
| `campaignId` | Uses `offers[].listingId` |

## Recommendations

### Minimal Required Structure (MVP)
For MVP, backend must provide:
- **Offer level**: `listingId`, `isEligible`, at least one `offerAction` with `actionContext` matching the template's `cta-context-action`
- **Template level**: `title`, `cta-text`, `cta-context-action` in viewElements

Everything else (colors, icons, etc.) can use defaults defined in the UI model.

```json
{
  "offers": [
    {
      "listingId": "auto-insurance-top-offer",
      "isEligible": true,
      "offerActions": [
        {
          "actionContext": "VIEW_ELIGIBLE_OFFER_DETAILS",
          "actionType": "OFFER_ACTION_TYPE_APP_NAVIGATION",
          "url": "earnin://auto-insurance"
        }
      ],
      "templates": [
        {
          "templateId": "top-offer-1",
          "viewElements": {
            "title": { "value": "...", "source": "..." },
            "cta-text": { "value": "...", "source": "..." },
            "cta-context-action": { "value": "VIEW_ELIGIBLE_OFFER_DETAILS", "source": "..." }
          }
        }
      ]
    }
  ]
}
```

### Recommended Structure (Full Flexibility)
Include all color/border properties if you want backend control over styling per campaign/template.

### Color Format
All colors should be provided as hex strings (e.g., `"#4161e4"`) without alpha channel.

### Boolean Values
Boolean properties should be strings: `"true"` or `"false"` (will be converted to Bool in iOS).

### Optional Elements
- If `card-icon` is missing, UI will show a placeholder/default image
- If `header-text` is missing, recommended badge will not be shown
- If color properties are missing, defaults from UI model will be used
- If no matching `offerAction` is found for `cta-context-action`, CTA will be disabled

## Client-Side Logic

The following UI properties are **derived** client-side:

1. **`showRecommendedBadge`**: 
   - Set to `true` if `header-text` viewElement exists
   - Set to `false` otherwise

2. **`isCtaEnabled`**: 
   - Set to `true` if:
     - Matching `offerAction` found (where `actionContext == cta-context-action`)
     - AND `offers[].isEligible == true`
   - Set to `false` otherwise

3. **`ctaActionUrl`**:
   - Find `offerAction` where `actionContext` matches `cta-context-action` value
   - Use that action's `url` property
   - If no match found, set to `nil`

This approach keeps UI state logic on the client side while backend provides content, styling, and actions.

