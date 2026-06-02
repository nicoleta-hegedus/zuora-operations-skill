---
name: zuora-operations
description: >
  Your go-to skill for anything Zuora — accounts, subscriptions, orders, product catalog, billing,
  payments, invoices, credit/debit memos, reports, and revenue recognition (RevPro). Use this skill
  whenever the user wants to look up an account, check subscriptions, create or amend subscriptions,
  manage the catalog, investigate billing issues, run reports, handle memos, process payments, do
  billing previews, query Zuora objects, or troubleshoot any billing scenario. Trigger phrases:
  "check this account", "what subscriptions does X have", "why was this invoice wrong", "cancel
  subscription", "set up a product", "run a report", "billing preview". Covers day-to-day ops and
  complex escalations. Excludes Workflows (use zuora-workflow-creator) and billing templates (use
  zuora-template-converter).
---

# Zuora Operations

This skill helps you work with Zuora efficiently — from quick account lookups to complex billing investigations. It covers the full Zuora Billing and Revenue surface except Workflows and billing document templates, which have dedicated skills.

## Two Modes of Operation

**Normal User** — day-to-day tasks: look up an account, check subscription status, find an invoice, see what products are in the catalog, run a report.

**Support/Admin Team** — complex scenarios: investigate billing discrepancies, handle mid-term subscription changes with proration, process credit memos, troubleshoot failed payments, manage bulk operations, run billing previews, query across objects for root-cause analysis.

The skill adapts based on the complexity of the request. Simple lookups get direct answers; complex investigations get a structured approach.

## Available MCP Tools

You have these Zuora MCP tools at your disposal. Pick the right tool for the job — don't default to `ask_zuora` when a direct action tool exists.

| Tool | When to Use |
|------|-------------|
| `query_objects` | **The workhorse.** Look up any Zuora object: accounts, subscriptions, invoices, payments, products, rate plans, charges, credit/debit memos, orders, usages, and 30+ more object types. Supports filtering, sorting, pagination, field selection, and expanding related objects. |
| `get_account_summary` | Quick account overview with recent credit/debit memos. Use when the user asks about a specific account. |
| `create_subscriptions` | Create new subscriptions with any pricing model. Supports new or existing accounts, all 16+ charge models, custom fields. Has a `help` mode for guidance. |
| `cancel_subscriptions` | Cancel subscriptions with configurable policy (end of term, end of last invoice period, specific date). Handles billing and credit options. |
| `renew_subscriptions` | Renew termed subscriptions. Extends the current term with existing renewal settings. |
| `create_products` | Create new products in the catalog. |
| `create_product_rate_plans` | Create rate plans within a product. |
| `create_product_rate_plan_charges` | Create charges within a rate plan. Supports all 16 charge models including tiered, volume, overage, discount, and multi-attribute pricing. Has a `help` mode. |
| `manage_billing_documents` | Generate PDFs, list files, email, or download invoices, credit memos, and debit memos. |
| `manage_billing_previews` | Preview future billing before it happens. Create preview runs, check status, download CSV results. |
| `manage_reports` | Search, list, run, export, and manage Zuora Billing reports. |
| `run_reports` | Run a specific report by name or ID with auto-polling for completion. |
| `manage_revenue_reports` | Zuora Revenue (RevPro) reports — list, run, check status, download. |
| `ask_zuora` | General Zuora knowledge questions — concepts, best practices, how things work. Use when you need to explain something or when no direct action tool applies. |
| `zuora_codegen` | Generate integration code (Java, Python, Node.js, C#, curl). Use only when the user needs code, not for operational tasks. |
| `zuora_approval` | Approve or reject sensitive operations that require confirmation. |

## How to Handle Requests

### Step 1: Identify What the User Needs

Parse the request into one of these categories:

- **Lookup / Investigation** — "show me account X", "what subscriptions does Y have", "find invoices from March"
- **Action** — "create a subscription", "cancel this subscription", "renew it"
- **Catalog Management** — "create a new product", "add a rate plan", "set up tiered pricing"
- **Billing Operations** — "generate an invoice PDF", "run a billing preview", "email this credit memo"
- **Reporting** — "run the aging report", "show me revenue recognition data"
- **Troubleshooting** — "why was this invoice amount wrong", "payment failed, what happened"
- **Knowledge** — "how does proration work", "what are trigger dates"

### Step 2: Pick the Right Tool(s)

**For lookups**, `query_objects` is almost always the answer. It supports 40+ object types with filtering.

Common query patterns:

```
# Find account by name or number
query_objects → objectType: "Accounts", filter: ["name.SW:Acme"] or ["accountnumber.EQ:A00001234"]

# Get subscriptions for an account
query_objects → objectType: "Subscriptions", filter: ["accountid.EQ:{accountId}"]

# Find invoices by date range
query_objects → objectType: "Invoices", filter: ["invoicedate.GE:2026-01-01", "invoicedate.LE:2026-03-31"]

# Look up products in the catalog
query_objects → objectType: "Products", filter: ["name.SW:Enterprise"]

# Find credit memos for an account  
query_objects → objectType: "CreditMemos", filter: ["accountid.EQ:{accountId}"]

# Get rate plan charges for a subscription
query_objects → objectType: "RatePlanCharges", filter: ["accountid.EQ:{accountId}"]

# Check payment history
query_objects → objectType: "Payments", filter: ["accountid.EQ:{accountId}"], sort: ["createddate.DESC"]
```

**For account overviews**, use `get_account_summary` — it returns the account details, subscriptions, invoices, payments, and recent credit/debit memos in one call.

**For actions**, use the specific action tool (create_subscriptions, cancel_subscriptions, renew_subscriptions, etc.).

**For billing documents**, use `manage_billing_documents` with the appropriate operation (generate_pdf, list_files, email, download).

**For reports**, use `run_reports` if you know the report name, or `manage_reports` with `search_reports` to find it first.

### Step 3: Present Results Clearly

- Lead with the answer the user needs, not raw JSON
- For account lookups: show account number, name, status, balance, currency
- For subscriptions: show subscription number, status, term dates, rate plans
- For invoices: show invoice number, date, amount, status, balance
- When investigating issues, connect the dots — don't just dump data

## Complex Scenario Playbooks

These are the patterns for the scenarios the support team handles most often. Read the appropriate reference file for detailed guidance.

### Subscription Lifecycle

For creating, amending, renewing, and cancelling subscriptions, including mid-term changes and proration scenarios, see `references/subscription-lifecycle.md`.

Key points:
- `create_subscriptions` has a `help` mode — use it to explore pricing options and get examples
- For mid-term changes, use Orders API concepts (the create_subscriptions tool wraps this)
- Cancellation policies: EndOfCurrentTerm, EndOfLastInvoicePeriod, SpecificDate
- Always confirm with the user before executing subscription changes

### Billing Investigations

When a user reports a billing discrepancy or asks "why is this invoice amount X":

1. Pull the account summary with `get_account_summary`
2. Query the specific invoice and its items: `query_objects` → Invoices, then InvoiceItems
3. Check for credit/debit memos against the invoice
4. Look at the subscription's rate plan charges to verify pricing
5. Check for proration by looking at charge segment dates vs billing period
6. If usage-based, check `Usages` and `ProcessedUsages` objects

### Product Catalog Management

Building out the catalog follows a top-down structure:

1. **Product** → `create_products` (name, effective dates)
2. **Rate Plan** → `create_product_rate_plans` (name, product ID)
3. **Rate Plan Charge** → `create_product_rate_plan_charges` (charge type, charge model, pricing)

The charge creation tool supports all 16 models. Use `help: true` with a specific ChargeType + ChargeModel combination to get the exact parameters needed.

For details on all pricing models, see `references/pricing-models.md`.

### Revenue Recognition (RevPro)

For Zuora Revenue reports:

1. `manage_revenue_reports` with `list_reports` to see available reports
2. `manage_revenue_reports` with `list_layouts` to find layouts for a report
3. `manage_revenue_reports` with `get_report_filters` to see what filters are needed
4. `manage_revenue_reports` with `run_report` to execute
5. `manage_revenue_reports` with `check_report_run_status` to poll
6. `manage_revenue_reports` with `download_report` to get results

### Billing Previews

To preview what billing will look like before running it:

1. `manage_billing_previews` with operation `create` — specify targetDate and optional filters
2. `manage_billing_previews` with operation `retrieve` — poll until status is "Completed"
3. `manage_billing_previews` with operation `download` — get the CSV results

This is useful for validating charges before a bill run, or forecasting revenue.

## Query Object Reference

The `query_objects` tool supports these object types (most commonly used ones marked with *):

**Core Objects:**
- *Accounts, *Subscriptions, *Invoices, *Payments, *PaymentMethods
- *CreditMemos, *DebitMemos, CreditMemoItems, DebitMemoItems
- *Contacts, ContactSnapshots

**Catalog Objects:**
- *Products, *ProductRatePlans, *ProductRatePlanCharges, ProductRatePlanChargeTiers

**Subscription Detail:**
- *RatePlans, *RatePlanCharges, RatePlanChargeTiers, RatingResults, RatingDetails

**Order Objects:**
- Orderss, OrderActions, OrderLineItems

**Billing Objects:**
- InvoiceItems, InvoiceSchedules, BillingRuns, TaxationItems

**Payment Objects:**
- PaymentApplications, PaymentRuns, PaymentSchedules, PaymentScheduleItems
- Refunds, RefundApplications, RefundApplicationItems, PaymentMethodSnapshots

**Usage Objects:**
- Usages, ProcessedUsages, DailyConsumptionSummarys

**Other:**
- Amendments, Fulfillments, Commitments, CommitmentPeriods
- PrepaidBalances, PrepaidBalanceFunds, PrepaidBalanceTransactions
- Ramps, ValidityPeriodSummarys, SummaryStatements, SummaryStatementRuns
- DeliveryAdjustments, CustomObjects

**Filter operators:** EQ, NE, LT, GT, GE, LE, SW (starts with), IN

**Tips:**
- Use `expand` to pull related objects in one call (e.g., expand subscriptions when querying accounts)
- Use `fields` to limit returned data when you only need specific columns
- Use `sort` for ordering (e.g., `createddate.DESC` for most recent first)
- `pageSize` max is 99; use `cursor` for pagination
- Custom fields are filterable using the `customField__c` format

## Important Guidelines

1. **Always confirm before mutating data.** Creating subscriptions, cancelling, renewing — these change real billing data. Summarize what you are about to do and get the user's OK.

2. **Use `query_objects` for investigation, not `ask_zuora`.** The ask_zuora tool is for knowledge questions. When the user wants actual data from their tenant, query it.

3. **Handle STOP_AND_CONFIRM responses.** The subscription tools may return this status when required fields are missing or invalid. When you see it, stop retrying and ask the user for the correct values.

4. **Chain queries when needed.** Complex investigations often require multiple queries — get the account, then its subscriptions, then the rate plan charges, then the invoices. Build the picture step by step.

5. **Don't guess IDs.** If you need a product rate plan ID or account ID and don't have it, query for it or ask the user. Never fabricate Zuora IDs.

6. **Respect the scope boundaries.** For workflow automation, redirect to the `zuora-workflow-creator` skill. For billing template conversion, redirect to `zuora-template-converter`.

7. **For code generation requests**, use the `zuora_codegen` tool and follow its mandatory workflow: code_guidance → get_model_details → generate code → code_rules.
