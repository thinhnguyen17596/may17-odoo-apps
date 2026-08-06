# How to get a Profit and Loss statement in Odoo 19 Community

**Odoo Community has no Profit and Loss report.** Neither Balance Sheet, Trial Balance, nor Aged
Receivable. They are not hidden behind developer mode or an unticked checkbox — the module that
produces them, `account_reports`, ships only with Odoo Enterprise.

This is the single most common surprise for a company that installs Odoo Community, invoices
happily for three months, and then tries to close the books.

This guide covers what you actually have, the four ways out, and how to pick.

## First, confirm it for yourself

Before believing anyone, check your own database. Turn on developer mode, then open
**Settings → Technical → Database Structure → Models** and search for `account.report`.

The model exists. It has no data, and no engine behind it. In Community, `account` ships the table
that *defines* what a financial report is; `account_reports`, which knows how to *run* one, is
Enterprise. That is why the Accounting menu shows journals, taxes and invoices but no
**Reporting → Profit and Loss**.

What Community does give you:

| You have | You do not have |
|---|---|
| Journals, journal items, chart of accounts | Profit and Loss |
| Taxes and tax reports (some localisations) | Balance Sheet |
| Invoices, bills, payments, reconciliation | Trial Balance |
| Partner Ledger through list views and filters | Aged Receivable / Aged Payable |
| Pivot and graph views on `account.move.line` | Any of the above as a printable statement |

That last row matters, and it is the honest starting point for option 1.

## Option 1 — Build it from a pivot view, free

`account.move.line` is a normal Odoo model, so you can group it.

1. Turn on developer mode.
2. Open **Accounting → Accounting → Journal Items**.
3. Filter to **Posted** entries and your date range.
4. Switch to the **Pivot** view.
5. Group rows by **Account**, and measure **Balance**.

You now have every account and its balance for the period. Add a filter on the account type to
separate income from expense, and you have the shape of a Profit and Loss.

**What this gets you:** real numbers, free, today, exportable to a spreadsheet.

**What it does not get you:** the sign convention. Income accounts carry credit balances, so revenue
appears as a negative number and your Profit and Loss reads upside down. There is no Gross Profit
line, no Net Profit line, no section subtotals, and nothing your accountant will accept as a
statement. You are rebuilding those in Excel every month.

For one look at one period, this is enough. As a monthly routine, it is not.

## Option 2 — Upgrade to Odoo Enterprise

The official answer, and the right one if you need the whole accounting suite: bank
synchronisation, follow-ups, asset management, the full localisation reports and audit trail.

**What to weigh:** Enterprise is priced per user per month, for every user in the database, not
only the accountants. A ten-person company buying Enterprise to get one report is paying for
nine seats that will never open it. Check the [current pricing](https://www.odoo.com/pricing)
against what a statement is worth to you.

## Option 3 — OCA modules

The Odoo Community Association publishes free financial reporting modules, historically under
`account-financial-reporting`.

**What to weigh:** they are free and open source, which is genuinely valuable. But each Odoo
release needs a migration, and the port lands when a volunteer does it — sometimes months after the
release, sometimes not at all for a given module. Check that the specific report you need exists on
the **19.0** branch before planning around it, and be honest about who in your team will fix it if
the port is late.

## Option 4 — A module that computes the statements from journal entries

The reports do not actually need Odoo's Enterprise engine. A Profit and Loss is a grouped sum over
`account.move.line`, split by account type, with the signs flipped for reading. Any module can
compute that.

[AI Assistant and Reports](https://apps.odoo.com/apps/modules/19.0/ai_assistant_reports) does
exactly this. It builds five statements — Profit and Loss, Balance Sheet, Trial Balance, Aged
Receivable and Aged Payable — from journal items, so they work on Community. You ask for one in
plain English:

> *"Profit and loss for this quarter"*
> *"Which customers owe us the most?"*
> *"Balance sheet as of the end of last month"*

and the statement is drawn in the page, with a link to open it as its own page and a button to
download it as a spreadsheet where the numbers are numbers rather than text.

It is a paid module, $249 one-time, for Odoo 19 Community and Enterprise. It connects to a
self-hosted Ollama server — no API key, nothing leaves your network — or to Claude, OpenAI, Gemini
or Azure OpenAI with your own key.

**What to weigh:** it costs money, and it needs the Accounting app installed. The statements are
computed from journal entries, which means they match your ledger but they are not a
localisation-certified filing document — if your country requires a specific legal format, check
that separately.

## How to choose

| Your situation | Take |
|---|---|
| One statement, once, and you have a spreadsheet | Option 1, the pivot view |
| You need the full accounting suite, and the per-user cost works | Option 2, Enterprise |
| You have in-house Odoo developers and prefer open source | Option 3, OCA |
| You want the statements now, on Community, without a migration project | Option 4 |

## Frequently asked questions

### Does Odoo 19 Community really have no Profit and Loss?

Correct. The reports live in `account_reports`, which is Enterprise-only. Community ships the
`account.report` model definition but no engine to run it, which is why the model exists in the
database while the menu does not.

### Can I just install account_reports on Community?

No. It is not published for Community and it is not open source, so there is nothing to install.

### Is the pivot-view method accurate?

The numbers are, because they come straight from posted journal items. The presentation is not:
income appears negative, there are no subtotals, and there is no Net Profit line. Accurate figures,
wrong shape.

### Will these reports satisfy an auditor?

They reflect your ledger exactly, which is what an auditor traces. Whether they satisfy a *filing*
requirement depends on your country's prescribed format — that is a localisation question, and
worth confirming with your accountant regardless of which option you pick.

### What about multi-company?

Statements that add several companies together must say so. If a report shows one total across
three ledgers without telling you, that number is easy to misread. Check that whichever option you
choose states the scope on the statement itself.

---

*Odoo Community profit and loss · Odoo 19 P&L Community · Odoo balance sheet Community ·
Odoo Community financial reports · Odoo aged receivable Community · account_reports Enterprise only ·
Odoo Community accounting reports · free profit and loss Odoo*
