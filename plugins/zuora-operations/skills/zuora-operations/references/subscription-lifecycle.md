# Subscription Lifecycle Reference

This reference covers the full lifecycle of a Zuora subscription: creation, amendments, renewal, cancellation, and the common complex scenarios the support team handles.

## Subscription States

- **Draft** — created but not yet activated
- **Active** — currently in effect, being billed
- **Suspended** — temporarily paused (requires workflow/custom logic)
- **Cancelled** — terminated, no longer billed
- **Expired** — reached end of term without renewal

## Creating a Subscription

Use `create_subscriptions` tool. Two paths:

### With Existing Account
```
orderDate: "2026-04-28"
existingAccountNumber: "A00001234"
createSubscriptionJson: {
  "subscribeToRatePlans": [{"productRatePlanId": "<rate-plan-id>"}],
  "terms": {
    "initialTerm": {"period": 12, "periodType": "Month", "termType": "TERMED"},
    "autoRenew": true,
    "renewalSetting": "RENEW_WITH_SPECIFIC_TERM",
    "renewalTerms": [{"period": 12, "periodType": "Month"}]
  }
}
```

### With New Account (created simultaneously)
```
orderDate: "2026-04-28"
newAccountJson: {
  "name": "Acme Corp",
  "currency": "USD",
  "billToContact": {"firstName": "Jane", "lastName": "Doe", "workEmail": "jane@acme.com"}
}
createSubscriptionJson: {
  "subscribeToRatePlans": [{"productRatePlanId": "<rate-plan-id>"}]
}
```

### Help Mode

Use `mode: "help"` with `helpTopic` to get guidance on specific scenarios:
- `quickstart` — step-by-step first subscription
- `create-tiered` — tiered pricing setup
- `create-usage` — usage-based charges
- `add-discount` — applying discounts
- `set-terms` — configuring subscription duration
- `multiple-charges` — combining multiple rate plans
- `pricing-options` — complete reference of all pricing models
- `troubleshooting` — common errors and fixes

## Mid-Term Subscription Changes

This is one of the most common support scenarios. When a customer wants to upgrade, downgrade, or add/remove products mid-cycle:

### Investigation Steps
1. Query the current subscription: `query_objects` → Subscriptions, filter by subscription number
2. Get its rate plans: `query_objects` → RatePlans, filter by subscription ID
3. Get the rate plan charges: `query_objects` → RatePlanCharges, filter by subscription ID
4. Check the current term dates to understand where in the cycle we are

### Proration
Zuora automatically handles proration when subscription changes occur mid-billing period. The proration calculation depends on:
- **Proration setting** on the charge (enabled/disabled)
- **Billing period alignment** settings
- **The effective date** of the change vs. the billing period boundaries

When investigating proration issues, look at:
- The charge segment dates (when the old charge ended, when the new one started)
- The invoice items — they will show prorated amounts if proration was applied
- The rate plan charge's billing period and alignment settings

## Cancellation

Use `cancel_subscriptions` tool. Three cancellation policies:

| Policy | Effect |
|--------|--------|
| EndOfCurrentTerm | Subscription stays active until the term end date, then cancels |
| EndOfLastInvoicePeriod | Cancels at the end of the last invoiced period |
| SpecificDate | Cancels on a specific date (requires `cancellationEffectiveDate`) |

### Cancellation with Credits
- Set `runBilling: true` to generate a final invoice/credit memo
- Set `applyCredit: true` to auto-apply credit memos to outstanding invoices
- Set `applicationOrder` to control priority (CreditMemo first or UnappliedPayment first)

### Common Cancellation Investigation
When a user asks "why was the customer still billed after cancellation":
1. Check the cancellation date vs. the invoice date
2. Check if `EndOfCurrentTerm` was used — the subscription stays active until term end
3. Look for charges with trigger events that may have already been billed
4. Check if there are usage charges that were rated after the cancellation was submitted

## Renewal

Use `renew_subscriptions` tool. Renewals extend the current term using the subscription's existing renewal settings.

Key parameters:
- `subscriptionKey` — subscription number or ID (required)
- `runBilling: true` — generate an invoice for the renewal
- `collect: true` — collect payment (requires `runBilling: true`)
- `targetDate` — date through which to calculate charges

## Troubleshooting Payment Failures

When a payment fails:

1. Check payment details: `query_objects` → Payments, filter by account, sort by `createddate.DESC`
2. Look at the payment method: `query_objects` → PaymentMethods, filter by account
3. Check if the payment method is still valid (expiration date, status)
4. Look at payment runs: `query_objects` → PaymentRuns for batch payment issues
5. Check for refunds: `query_objects` → Refunds, filter by account

## Billing Investigation Checklist

When something looks wrong on an invoice:

1. **Get the account summary** — `get_account_summary` for a quick overview
2. **Pull the invoice** — `query_objects` → Invoices, filter by invoice number
3. **Get invoice items** — `query_objects` → InvoiceItems, filter by invoice ID
4. **Check the subscription charges** — `query_objects` → RatePlanCharges, filter by subscription
5. **Look for adjustments** — `query_objects` → CreditMemos and DebitMemos for the account
6. **Check taxation** — `query_objects` → TaxationItems if tax amounts look wrong
7. **Verify the catalog** — `query_objects` → ProductRatePlanCharges to confirm pricing in the catalog matches what was charged
