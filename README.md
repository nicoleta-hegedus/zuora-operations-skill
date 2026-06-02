# zuora-operations

> Zuora operations skill for Claude Cowork — accounts, subscriptions, billing, payments, invoices, and revenue recognition.

A Claude Cowork skill that provides day-to-day Zuora Billing operations via MCP tools: account lookups, subscription lifecycle management, billing investigations, product catalog setup, reports, and revenue recognition (RevPro).

---

## What it covers

| Area | Capabilities |
|------|-------------|
| Accounts | Lookup, summary, balance, contacts |
| Subscriptions | Create, amend, renew, cancel, investigate |
| Billing | Invoice queries, billing previews, bill runs |
| Payments | Payment history, method checks, failure troubleshooting |
| Credit/Debit Memos | Create, query, apply |
| Product Catalog | Products, rate plans, all 16+ charge models |
| Reports | Billing reports, revenue recognition (RevPro) |
| Troubleshooting | Billing discrepancies, proration issues, payment failures |

---

## Install

### Claude Cowork (desktop)

1. Open the **Claude** desktop app and click the **Cowork** tab
2. Click **Customize** in the left sidebar
3. Under plugins, click **+** > **Add marketplace**
4. Enter the repository:

```
nicoleta-hegedus/zuora-operations-skill
```

5. Click **Sync**
6. Find **zuora-operations** in the list and click **Install**

### Claude Code (CLI)

```
/plugin marketplace add nicoleta-hegedus/zuora-operations-skill
/plugin install zuora-operations@zuora-operations-skill
/reload-plugins
```

### Update

```
/plugin marketplace update zuora-operations-skill
/plugin update zuora-operations@zuora-operations-skill
```

---

## Usage

### Natural language

```
Check account A00001234
What subscriptions does Acme Corp have?
Why was invoice INV-0005678 amount wrong?
Cancel subscription S-00012345 at end of term
Set up a product with tiered usage pricing
Run a billing preview for next month
```

### How it works

The skill uses Zuora MCP tools (`query_objects`, `get_account_summary`, `create_subscriptions`, etc.) to interact directly with your Zuora tenant. It adapts based on request complexity:

- **Simple lookups** get direct answers
- **Complex investigations** get a structured multi-step approach with connected-dots analysis

---

## Requirements

- Claude Cowork (any plan with skill support)
- Zuora MCP connector configured and authenticated
- Access to a Zuora tenant (sandbox or production)

---

## Related skills

- **zuora-workflow-creator** — for Zuora Workflow automation
- **zuora-template-converter** — for billing document template conversion (Word to HTML)

---

## License

MIT — see [LICENSE](LICENSE)
