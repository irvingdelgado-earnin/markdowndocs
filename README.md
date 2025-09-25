# Updated ProductDiscovery Eligibility Report
## Complete Analysis with Specific API Details

This report provides the **exact eligibility logic** and **specific API endpoints** for each product that the backend must replicate to maintain the same behavior.

---

## Product Eligibility Logic - Complete Analysis with API Details

### 1. BalanceShield (ProductId: 108)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    if cashOutLinkUseCase.isEnabled {
        return false
    }
    return !bankedEwaDecision.isBlockedThroughBankedEWA && 
           !bankedEwaDecision.isConsideredForBankedEWA
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.balanceShield`
- **Data Type**: `ProductsBalanceShield`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        balanceShield: ProductsBalanceShield {
            alertsEnabled: Bool                    // ← Used for audience calculation
            enrollmentStatus: BalanceShieldEnrollmentStatus  // ← Used for enrollment
            tipInCents: Int
            cashOutInCents: Int
            selectedThresholdInCents: Int
            selectedAutoTransferThresholdInCents: Int?
            maxThresholdInCents: Int?
            automaticTransferOption: BalanceShieldTransferMethod?
            feeAmountInCents: Int
            balanceShieldFailedReason: String?
        }
    }
}
```

#### Feature Flags Required:
- `cashOutLinkUseCase.isEnabled` (boolean)
- `bankedEwaDecision.isBlockedThroughBankedEWA` (boolean)
- `bankedEwaDecision.isConsideredForBankedEWA` (boolean)

#### Business Logic:
- ❌ **Block if**: `cashOutLinkUseCase.isEnabled == true`
- ❌ **Block if**: `bankedEwaDecision.isBlockedThroughBankedEWA == true`
- ❌ **Block if**: `bankedEwaDecision.isConsideredForBankedEWA == true`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let audience = balanceShieldSettings.alertsEnabled
    ? "enrolled"      // If alerts are enabled
    : "notEnrolled"   // If alerts are disabled
```

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isTransfersEnabled = [.enrolled, .enrolledWithoutFee, .enrolledButFeePending]
    .contains(balanceShieldSettings.enrollmentStatus)
let isEnrolled = balanceShieldSettings.alertsEnabled || isTransfersEnabled
```

---

### 2. CreditMonitoring (ProductId: 104)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let statusResponse = homeRepository.cardStatus.value.wrappedValue?.status
    else { return false }
    return !statusResponse.hidePromoCard
}
```

#### API Dependencies:
- **Primary API**: `GET /user/credit-monitoring/status` (separate API call)
- **Response Structure**: `StatusResponse`
- **Data Type**: `StatusResponse`

#### Specific API Data Required:
```swift
// From GET /user/credit-monitoring/status
StatusResponse {
    hidePromoCard: Bool           // ← Used for eligibility
    enrollmentStatus: EnrollmentStatus  // ← Used for enrollment
    // ... other fields
}
```

#### Feature Flags Required: None

#### Business Logic:
- ❌ **Block if**: `statusResponse` is null/undefined
- ❌ **Block if**: `statusResponse.hidePromoCard == true`
- ✅ **Allow if**: `statusResponse.hidePromoCard == false`

#### Dynamic Audience: Static `defaultAudienceFilter`

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = statusResponse.enrollmentStatus == .enrolled
```

---

### 3. AutoInsurance (ProductId: 103)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let autoInsurance = products.autoInsurance else {
        return false
    }
    return marketplaceConfiguration.isAutoInsuranceEnabled &&
           autoInsurance.enrollmentStatus == .enrolled
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.autoInsurance`
- **Data Type**: `ProductsAutoInsurance`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        autoInsurance: ProductsAutoInsurance {
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
            eligibleForPromo: Bool                 // ← Used for audience
            // ... other fields
        }
    }
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.isAutoInsuranceEnabled` (boolean)

#### Business Logic:
- ❌ **Block if**: `autoInsurance` is null/undefined
- ❌ **Block if**: `marketplaceConfiguration.isAutoInsuranceEnabled == false`
- ❌ **Block if**: `autoInsurance.enrollmentStatus != .enrolled`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let audience = products.autoInsurance?.eligibleForPromo == true
    ? "discoveryPageAndTileControl"
    : "discoveryPageOnlyControl"
```

#### Enrollment Logic: Always `false`

---

### 4. Savings (ProductId: 106)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let savingsUserStatus = store.state.savingsUserStatusResponse.value.wrappedValue
    else { return false }
    return savingsUserStatus.enrollmentStatus != .inactive && 
           !savingsUserStatus.needsAppUpdate
}
```

#### API Dependencies:
- **Primary API**: `GET /user/savings/status` (separate API call)
- **Response Structure**: `GetUserStatusResponse`
- **Data Type**: `GetUserStatusResponse`

#### Specific API Data Required:
```swift
// From GET /user/savings/status
GetUserStatusResponse {
    enrollmentStatus: SavingsEnrollmentStatus  // ← Used for eligibility
    needsAppUpdate: Bool                       // ← Used for eligibility
    showKycMessage: Bool
    kycPassed: Bool
    depositBlocked: Bool
}
```

#### Feature Flags Required: None

#### Business Logic:
- ❌ **Block if**: `savingsUserStatus` is null/undefined
- ❌ **Block if**: `savingsUserStatus.enrollmentStatus == .inactive`
- ❌ **Block if**: `savingsUserStatus.needsAppUpdate == true`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience: Static `defaultAudienceFilter`

#### Enrollment Logic: Complex logic involving tip jars, multi-product status, and onboarding

---

### 5. Adjoe (ProductId: 116)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard marketplaceConfiguration.isAdjoeEnabled else {
        return false
    }
    guard let adjoeProduct = products.marketplace?.first(
        where: { $0.productName == .adjoeplaytime }
    ),
          let savingsEnrollmentStatus = store.state.savingsUserStatusResponse.value.wrappedValue?.enrollmentStatus,
          savingsEnrollmentStatus != .inactive
    else {
        return false
    }
    return adjoeProduct.enrollmentStatus != .notEligible
}
```

#### API Dependencies:
- **Primary API 1**: `GET /user/home-state`
- **Primary API 2**: `GET /user/savings/status` (Savings dependency)
- **Primary API 3**: `GET /user/adjoe/entitlements` (for enrollment)
- **Response Structure 1**: `HomeStateResponse.products.marketplace`
- **Response Structure 2**: `GetUserStatusResponse`
- **Response Structure 3**: `ProductSavingsEntitlement`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.adjoeplaytime
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
            eligibleForPromo: Bool                 // ← Used for audience
        }
    }
}

// From GET /user/savings/status
GetUserStatusResponse {
    enrollmentStatus: SavingsEnrollmentStatus  // ← Must be != .inactive
}

// From GET /user/adjoe/entitlements
ProductSavingsEntitlement {
    accruedAmountCents: Cents                   // ← Used for enrollment
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.isAdjoeEnabled` (boolean)

#### Business Logic:
- ❌ **Block if**: `marketplaceConfiguration.isAdjoeEnabled == false`
- ❌ **Block if**: Adjoe product not found in marketplace
- ❌ **Block if**: `savingsEnrollmentStatus == .inactive` (Savings dependency)
- ❌ **Block if**: `adjoeProduct.enrollmentStatus == .notEligible`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let isEnrolledInAdjoe = accruedAmountCents > 0
let audience = adjoeProduct.eligibleForPromo == true && !isEnrolledInAdjoe
    ? ProductDiscovery.Configuration.defaultAudienceFilter
    : "adjoeListOnly"
```

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = accruedAmountCents > 0
```

---

### 6. EarnInCard (ProductId: 102)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard earninAccountExperimentManager.livePayWaitlistEnabled else {
        return false
    }
    switch cardAudience {
    case .ineligible, .waitlisted:
        return false
    case .neverWaitlisted, .graduated:
        return true
    }
}
```

#### API Dependencies:
- **Primary API 1**: `GET /user/home-state`
- **Primary API 2**: `GET /user/earnin-account` (Card funnel status)
- **Response Structure 1**: `HomeStateResponse.productEnrollment`
- **Response Structure 2**: `EarninAccountResponse`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    productEnrollment: ProductEnrollment {
        getFunnelSummary(for: .earninCard): ProductFunnelSummary {
            status: ProductFunnelStatus        // ← Used for audience calculation
            waitlistStatus: WaitlistStatus     // ← Used for audience calculation
        }
    }
}

// From GET /user/earnin-account
EarninAccountResponse {
    cardFunnelStatus: CardFunnelStatus        // ← Used for audience calculation
}
```

#### Feature Flags Required:
- `earninAccountExperimentManager.livePayWaitlistEnabled` (boolean)

#### Business Logic:
- ❌ **Block if**: `livePayWaitlistEnabled == false`
- ❌ **Block if**: `cardAudience == .ineligible`
- ❌ **Block if**: `cardAudience == .waitlisted`
- ✅ **Allow if**: `cardAudience == .neverWaitlisted`
- ✅ **Allow if**: `cardAudience == .graduated`

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly (very complex logic):
let funnelStatus = earninAccountResponse.cardFunnelStatus ?? .ineligible
let eligibleFunnelStatus = [.eligible, .kycPassed, .kycSoftFail].contains(funnelStatus)

guard let cardEnrollment = homeStateResponse.productEnrollment?.getFunnelSummary(for: .earninCard)
else {
    return eligibleFunnelStatus ? .graduated : .ineligible
}

let cardEnrollmentStatus = cardEnrollment.status
let waitlistStatus = cardEnrollment.waitlistStatus

if cardEnrollmentStatus.isStartedOrNeverEnrolled {
    switch waitlistStatus {
    case .neverWaitlisted:
        return "earnInCardWaitlistEligible"
    case .waitlisted:
        return "earninCardWaitlisted"
    case .bypassWaitlist, .graduated:
        return "earnInCardEligible"
    }
}

return eligibleFunnelStatus ? "earnInCardEligible" : "earnInCardNotEligible"
```

#### Enrollment Logic: Complex logic involving funnel status and waitlist status

---

### 7. PersonalLoan (ProductId: 117)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let personalLoanProduct = personalLoanProduct(from: products)
    else { return false }
    return personalLoanProduct.enrollmentStatus != .notEligible
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.marketplace`
- **Data Type**: `MarketplaceProduct`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.personalloan
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
        }
    }
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.isPersonalLoanEnabled` (boolean)

#### Business Logic:
- ❌ **Block if**: Personal loan product not found in marketplace
- ❌ **Block if**: `marketplaceConfiguration.isPersonalLoanEnabled == false`
- ❌ **Block if**: `personalLoanProduct.enrollmentStatus == .notEligible`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience: Static `defaultAudienceFilter`

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = personalLoanProduct.enrollmentStatus == .enrolled
```

---

### 8. DirectDepositAccount (ProductId: 110)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard
        !bankedEwaDecision.isConsideredForBankedEWA,
        earninAccountExperimentManager.directDepositDiscoveryTileEnabled
    else { return false }
    return earlyPayDecision.shouldShowDiscovery()
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.depositAccount`
- **Data Type**: `ProductsDepositAccount`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        depositAccount: ProductsDepositAccount {
            accountStatus: DepositAccountStatus     // ← Used for audience calculation
            enrollmentStatus: EnrollmentStatus     // ← Used for audience calculation
            recentDepositAmount: Int?              // ← Used for audience calculation
            recentDepositTimestamp: Date?         // ← Used for audience calculation
        }
    }
}
```

#### Feature Flags Required:
- `earninAccountExperimentManager.directDepositDiscoveryTileEnabled` (boolean)
- `bankedEwaDecision.isConsideredForBankedEWA` (boolean)
- `earlyPayDecision.shouldShowDiscovery()` (complex decision logic)

#### Business Logic:
- ❌ **Block if**: `bankedEwaDecision.isConsideredForBankedEWA == true`
- ❌ **Block if**: `directDepositDiscoveryTileEnabled == false`
- ❌ **Block if**: `earlyPayDecision.shouldShowDiscovery() == false`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let depositAccount = productsResponse.depositAccount
let isDDNeverReceived = depositAccount.recentDepositAmount == nil &&
                       depositAccount.recentDepositTimestamp == nil

let audience = (depositAccount.accountStatus == .active && depositAccount.enrollmentStatus == .enrolled) ||
               (depositAccount.accountStatus == .pending && earlyPayDecision.isEnrolledAndHasReceivedDirectDeposit)
    ? "enrollmentCompleteDDActive"
    : "enrollmentNotStarted"
```

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = earlyPayDecision.isEnrolledAndHasReceivedDirectDeposit
```

---

### 9. RecurringBills (ProductId: 112)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    if repository.hasEarninCard {
        return true
    } else if experiment.isEnabledForCashoutUsers {
        return !repository.isMultiProduct || repository.isCashOutUser
    } else {
        return false
    }
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.recurringBills`
- **Data Type**: `ProductsRecurringBills`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        recurringBills: ProductsRecurringBills {
            enrollmentStatus: EnrollmentStatus     // ← Used for enrollment
        }
    }
}
```

#### Feature Flags Required:
- `experiment.isEnabledForCashoutUsers` (boolean)
- `repository.hasEarninCard` (boolean)
- `repository.isMultiProduct` (boolean)
- `repository.isCashOutUser` (boolean)

#### Business Logic:
- ✅ **Always allow if**: User has EarnIn card
- ✅ **Allow if**: `experiment.isEnabledForCashoutUsers == true` AND (`!isMultiProduct` OR `isCashOutUser`)
- ❌ **Block if**: All above conditions are false

#### Dynamic Audience: Static `defaultAudienceFilter`

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = recurringBills.enrollmentStatus == .enrolled
```

---

### 10. AffordableBenefits (ProductId: 119)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let affordableBenefitstProduct = affordableBenefitstProduct(from: products)
    else { return false }

    guard affordableBenefitstProduct.enrollmentStatus == .eligible ||
            affordableBenefitstProduct.enrollmentStatus == .enrolled else {
        return false
    }

    return marketplaceConfiguration.inAffordableBenefitsExp
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.marketplace`
- **Data Type**: `MarketplaceProduct`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.affordableBenefits
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
            eligibleForPromo: Bool                 // ← Used for audience
        }
    }
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.inAffordableBenefitsExp` (boolean)

#### Business Logic:
- ❌ **Block if**: Affordable benefits product not found in marketplace
- ❌ **Block if**: `enrollmentStatus != .eligible` AND `enrollmentStatus != .enrolled`
- ❌ **Block if**: `marketplaceConfiguration.inAffordableBenefitsExp == false`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let audience = product.eligibleForPromo
    ? ProductDiscovery.Configuration.defaultAudienceFilter
    : "affordableBenefitsListOnly"
```

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = affordableBenefitstProduct.enrollmentStatus == .enrolled
```

---

### 11. QuickCash (ProductId: 114)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let quickCashProduct = getQuickCashProduct(products: products) else { return false }
    let eligibleForQuickCash = quickCashProduct.enrollmentStatus == .eligible
    return marketplaceConfiguration.isQuickCashEnabled && eligibleForQuickCash
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.marketplace`
- **Data Type**: `MarketplaceProduct`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.quickcash
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
            eligibleForPromo: Bool                 // ← Used for audience
        }
    }
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.isQuickCashEnabled` (boolean)

#### Business Logic:
- ❌ **Block if**: QuickCash product not found in marketplace
- ❌ **Block if**: `marketplaceConfiguration.isQuickCashEnabled == false`
- ❌ **Block if**: `quickCashProduct.enrollmentStatus != .eligible`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let audience = quickCashProduct.eligibleForPromo
    ? ProductDiscovery.Configuration.defaultAudienceFilter
    : "quickCashListOnly"
```

#### Enrollment Logic: Always `true`

---

### 12. AutoRefinance (ProductId: 115)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard let marketplaceProducts = products.marketplace,
          marketplaceConfiguration.isAutoRefinanceEnabled else {
        return false
    }

    var autoRefinanceProduct: MarketplaceProduct?
    marketplaceProducts.forEach { marketplaceProduct in
        if marketplaceProduct.productName == .autorefinance {
            autoRefinanceProduct = marketplaceProduct
        }
    }

    return autoRefinanceProduct?.enrollmentStatus != .notEligible
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.marketplace`
- **Data Type**: `MarketplaceProduct`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.autorefinance
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
            eligibleForPromo: Bool                 // ← Used for audience
        }
    }
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.isAutoRefinanceEnabled` (boolean)

#### Business Logic:
- ❌ **Block if**: Marketplace products not found
- ❌ **Block if**: `marketplaceConfiguration.isAutoRefinanceEnabled == false`
- ❌ **Block if**: Auto refinance product not found in marketplace
- ❌ **Block if**: `autoRefinanceProduct.enrollmentStatus == .notEligible`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience Calculation:
```swift
// Backend must calculate this exactly:
let audience = autoRefinanceProduct?.eligibleForPromo == true
    ? ProductDiscovery.Configuration.defaultAudienceFilter
    : "autoRefinanceListOnly"
```

#### Enrollment Logic: Always `false`

---

### 13. CashbackRewards (ProductId: 118)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    guard
        let marketplaceProducts = products.marketplace,
        let cashbackRewardsProduct = marketplaceProducts.first(where: { $0.productName == .cashbackrewards })
    else { return false }

    guard cashbackRewardsProduct.enrollmentStatus != .notEligible else {
        return false
    }

    return isInExperiment
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.marketplace`
- **Data Type**: `MarketplaceProduct`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.cashbackrewards
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
        }
    }
}
```

#### Feature Flags Required:
- `marketplaceConfiguration.cashbackRewardsHoldoutExp == .control` (boolean)
- `marketplaceConfiguration.cashbackRewardsExp == .enabled` (boolean)

#### Business Logic:
- ❌ **Block if**: Marketplace products not found
- ❌ **Block if**: Cashback rewards product not found in marketplace
- ❌ **Block if**: `cashbackRewardsProduct.enrollmentStatus == .notEligible`
- ❌ **Block if**: `isInExperiment == false` (both feature flags must be true)
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience: Static `"cashbackRewardsListOnly"`

#### Enrollment Logic:
```swift
// Backend must calculate this exactly:
let isEnrolled = cashbackRewardsProduct.enrollmentStatus == .enrolled
```

---

### 14. RoadsideAssistance (ProductId: 111)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    marketplaceRepository.updateEnrollmentDate()
    guard let marketplaceProducts = products.marketplace,
        roadsideAssistanceConfiguration.isEnabled else {
        return false
    }

    var roadsideAssistanceProduct: MarketplaceProduct?
    marketplaceProducts.forEach { marketplaceProduct in
        if marketplaceProduct.productName == .roadsideassistance {
            roadsideAssistanceProduct = marketplaceProduct
        }
    }

    return marketplaceRepository.enrolledIntoRoadsideAssistanceDate == nil &&
        roadsideAssistanceProduct?.enrollmentStatus != .notEligible
}
```

#### API Dependencies:
- **Primary API**: `GET /user/home-state`
- **Response Structure**: `HomeStateResponse.products.marketplace`
- **Data Type**: `MarketplaceProduct`

#### Specific API Data Required:
```swift
// From GET /user/home-state
HomeStateResponse {
    products: Products {
        marketplace: [MarketplaceProduct] {
            productName: MarketplaceProductName.roadsideassistance
            enrollmentStatus: EnrollmentStatus     // ← Used for eligibility
        }
    }
}
```

#### Feature Flags Required:
- `roadsideAssistanceConfiguration.isEnabled` (boolean)
- `marketplaceRepository.enrolledIntoRoadsideAssistanceDate` (date tracking)

#### Business Logic:
- ❌ **Block if**: Marketplace products not found
- ❌ **Block if**: `roadsideAssistanceConfiguration.isEnabled == false`
- ❌ **Block if**: Roadside assistance product not found in marketplace
- ❌ **Block if**: `enrolledIntoRoadsideAssistanceDate != nil` (already enrolled)
- ❌ **Block if**: `roadsideAssistanceProduct.enrollmentStatus == .notEligible`
- ✅ **Allow if**: All above conditions are false

#### Dynamic Audience: Static `defaultAudienceFilter`

#### Enrollment Logic: Complex logic involving marketplace repository

---

### 15. TransferOut (ProductId: 105)

#### Eligibility Logic:
```swift
func isEligible(products: Products) -> Bool {
    true  // Always eligible for everyone
}
```

#### API Dependencies: None

#### Feature Flags Required: None

#### Business Logic: 
- ✅ **Always allow**: No conditions, always eligible

#### Dynamic Audience: Static `defaultAudienceFilter`

#### Enrollment Logic: Always `true`

---

## Summary of API Dependencies

### Required API Endpoints:
1. **`GET /user/home-state`** - Main product data (ALL products need this)
2. **`GET /user/savings/status`** - Savings enrollment status (Savings + Adjoe)
3. **`GET /user/credit-monitoring/status`** - Credit monitoring status (CreditMonitoring only)
4. **`GET /user/adjoe/entitlements`** - Adjoe entitlements (Adjoe only)
5. **`GET /user/earnin-account`** - EarnIn account status (EarnInCard only)

### Feature Flags Required:
- `cashOutLinkUseCase.isEnabled`
- `marketplaceConfiguration.isAutoInsuranceEnabled`
- `marketplaceConfiguration.isAdjoeEnabled`
- `marketplaceConfiguration.isPersonalLoanEnabled`
- `marketplaceConfiguration.isQuickCashEnabled`
- `marketplaceConfiguration.isAutoRefinanceEnabled`
- `marketplaceConfiguration.inAffordableBenefitsExp`
- `marketplaceConfiguration.cashbackRewardsHoldoutExp`
- `marketplaceConfiguration.cashbackRewardsExp`
- `earninAccountExperimentManager.livePayWaitlistEnabled`
- `earninAccountExperimentManager.directDepositDiscoveryTileEnabled`
- `experiment.isEnabledForCashoutUsers`
- `roadsideAssistanceConfiguration.isEnabled`
- `bankedEwaDecision.isBlockedThroughBankedEWA`
- `bankedEwaDecision.isConsideredForBankedEWA`

---

## Backend Migration Requirements

### 1. New API Endpoint:
```swift
GET /api/user/products/discovery
{
  "products": [
    {
      "id": "balance_shield",
      "title": "Get balance alerts",
      "isEligible": true,
      "isEnrolled": false,
      "audience": "notEnrolled",
      "backgroundColor": "#D5C2FE",
      "icon": "currency-dollar-shield",
      "badge": null
    }
  ]
}
```

### 2. Backend Logic Requirements:
- **Replicate all 18 DiscoverableProduct eligibility logic**
- **Handle all feature flags and experiment logic**
- **Calculate dynamic audience values**
- **Determine enrollment status for each product**
- **Return only eligible products in correct order**

### 3. Data Sources:
- **All current API responses** must be available in backend
- **All feature flags** must be accessible in backend
- **All experiment configurations** must be available in backend

This updated report provides the backend team with **exact API endpoints**, **specific data structures**, and **precise eligibility logic** for each product that must be preserved during the migration.
