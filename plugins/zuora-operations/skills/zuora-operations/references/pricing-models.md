# Zuora Pricing Models Reference

This reference covers all 16+ charge models available in Zuora. Use this when setting up product catalog charges or when investigating how a subscription is priced.

## Charge Types

Every charge has a **ChargeType** and a **ChargeModel**:

- **Recurring** — billed on a schedule (monthly, quarterly, annually, etc.)
- **OneTime** — billed once at subscription creation or activation
- **Usage** — billed based on consumption data

## Charge Models by Type

### Recurring Charges

| Model | Use Case | Key Parameters |
|-------|----------|----------------|
| Flat Fee Pricing | Fixed subscription fee ($99/mo) | Price |
| Per Unit Pricing | Per-seat pricing ($10/user/mo) | Price, DefaultQuantity |
| Delivery Pricing | Delivery-based recurring | Price |
| Discount Percentage | % off other charges | DiscountPercentage, ApplyDiscountTo |
| Discount Fixed Amount | Fixed $ off other charges | Price, ApplyDiscountTo |

### OneTime Charges

| Model | Use Case | Key Parameters |
|-------|----------|----------------|
| Flat Fee Pricing | Setup fee, implementation fee | Price |
| Per Unit Pricing | Per-unit one-time charge | Price, DefaultQuantity |

### Usage Charges

| Model | Use Case | Key Parameters |
|-------|----------|----------------|
| Per Unit Pricing | Simple per-unit (e.g., $0.01/API call) | Price, UOM |
| Flat Fee Pricing | Flat fee per usage period | Price, UOM |
| Tiered Pricing | Progressive tiers (1-100: $10, 101-500: $8, 501+: $5) | TierData, UOM |
| Volume Pricing | All units at one price based on total quantity | TierData, UOM |
| Overage Pricing | Free included units, then overage rate | IncludedUnits, Price, UOM |
| Tiered With Overage | Tiered pricing with overage component | TierData, IncludedUnits, UOM |
| High Watermark Tiered | Peak usage determines tier | TierData, UOM |
| High Watermark Volume | Peak usage determines volume price | TierData, UOM |
| Prerated Pricing | Prorated usage charges | TierData, UOM |
| Prerated Per Unit | Prorated per-unit charges | Price, UOM |
| MultiAttributePricing | Formula-based complex pricing | ConfigurationJson, UOM |

## Creating Charges via MCP

Use `create_product_rate_plan_charges` with `help: true` and the specific ChargeType + ChargeModel to get exact parameter requirements and examples for that combination.

### Tiered Pricing Example (Usage)

```
ChargeType: "Usage"
ChargeModel: "Tiered Pricing"
ProductRatePlanId: "<rate-plan-id>"
UOM: "API Calls"
ProductRatePlanChargeTierData: "[
  {\"Tier\": 1, \"StartingUnit\": 1, \"EndingUnit\": 1000, \"Price\": 0.01, \"PriceFormat\": \"PerUnit\"},
  {\"Tier\": 2, \"StartingUnit\": 1001, \"EndingUnit\": 10000, \"Price\": 0.008, \"PriceFormat\": \"PerUnit\"},
  {\"Tier\": 3, \"StartingUnit\": 10001, \"EndingUnit\": null, \"Price\": 0.005, \"PriceFormat\": \"PerUnit\"}
]"
```

### Volume Pricing Example

```
ChargeType: "Usage"
ChargeModel: "Volume Pricing"
ProductRatePlanId: "<rate-plan-id>"
UOM: "GB"
ProductRatePlanChargeTierData: "[
  {\"Tier\": 1, \"StartingUnit\": 1, \"EndingUnit\": 100, \"Price\": 0.10, \"PriceFormat\": \"PerUnit\"},
  {\"Tier\": 2, \"StartingUnit\": 101, \"EndingUnit\": 1000, \"Price\": 0.07, \"PriceFormat\": \"PerUnit\"},
  {\"Tier\": 3, \"StartingUnit\": 1001, \"EndingUnit\": null, \"Price\": 0.04, \"PriceFormat\": \"PerUnit\"}
]"
```

## Discount Charges

Discounts can be applied to specific charge types:

- **ApplyDiscountTo**: "ONETIMERECURRINGUSAGE", "ONETIMERECURRING", "ONETIMEDISCOUNT", "RECURRINGUSAGE", "RECURRING", "USAGE", "ONETIME"
- **DiscountLevel**: "subscription" (all charges), "rateplan" (same rate plan only), "account" (all charges on account)

## Billing Period Options

For recurring charges: Month, Quarter, Annual, Semi-Annual, Specific Months, Subscription Term, Week, Specific Weeks, Specific Days.

## Trigger Events

When billing starts for a charge:
- **ContractEffective** — when the subscription contract goes into effect (default)
- **ServiceActivation** — when services/products are activated
- **CustomerAcceptance** — when the customer accepts the services
