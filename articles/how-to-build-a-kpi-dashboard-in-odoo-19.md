# How to build a KPI dashboard in Odoo 19 without a developer

Odoo ships graph and pivot views, but they live inside one model and one menu. The moment you
want revenue, open invoices and this month's new customers **on one screen**, you are told to
call a developer. You do not have to.

This walks through building a real dashboard in Odoo 19 in about ten minutes.

## What you need

- Odoo 19, Community or Enterprise.
- A dashboard builder module. This guide uses
  [May17 Dashboard](https://apps.odoo.com/apps/modules/19.0/may17_dashboard); the steps map onto
  any builder that reads arbitrary models.

## Step 1 — Start from a template, then edit

Open **Dashboard → Dashboards → New**, turn on edit mode and press **Add**. Pick the template
matching the app you care about — Sale Order, Invoicing, CRM, Purchase, Inventory, Project,
Manufacturing, Employees, Point of Sale or Contacts.

One click builds 9 to 13 configured items: KPI tiles, charts, a list and a to-do. Now you are
editing something that already works instead of staring at an empty grid.

## Step 2 — Understand the four fields that define every item

Every item, whatever its shape, is the same four decisions:

| Field | What it answers |
|---|---|
| **Model** | Which records? (`sale.order`, `account.move`, your own Studio model) |
| **Measure** | Which number? Leave it empty to count records. |
| **Group By** | Which axis? A date group-by gains a granularity: day, week, month, quarter, year. |
| **Domain** | Which subset? `[('state', '=', 'sale')]` |

If you can answer those four in a sentence — *"count sale orders, by month, where state is
sale"* — you can build the item.

## Step 3 — Watch the preview instead of saving and reloading

The chart renders inside the form as you configure. Switch the item from bar to line to treemap
and watch the same data change shape; that is how you find the chart that reads best, in seconds
instead of a save-refresh-repeat loop.

## Step 4 — Common recipes

**Revenue per month:** model `account.move`, domain
`[('move_type', '=', 'out_invoice'), ('state', '=', 'posted')]`, measure `amount_total`,
aggregation Sum, group by `invoice_date` with granularity Month, type Line.

**Top 10 customers:** same model and domain, group by `partner_id`, sort by the measure
descending, record limit 10, type Horizontal Bar.

**Pipeline by stage:** model `crm.lead`, domain `[('type', '=', 'opportunity')]`, measure
`expected_revenue`, group by `stage_id`, type Bar.

**Only my records:** add `[('user_id', '=', %UID)]` to the domain. `%UID` resolves to the person
looking at the dashboard, so one dashboard serves the whole team.

**Only this company:** `%MYCOMPANY` resolves to the active company.

**Target versus actual:** enable the target on a tile and set its value; the tile shows the gap
with an arrow. On a chart, the target draws as a dashed line across the plot.

## Step 5 — Make it readable

- Put the four or five numbers people ask about in tiles on the top row.
- One trend chart, not three. A dashboard answers a question; it is not an archive.
- Set an auto-refresh interval only for dashboards someone actually leaves open.
- Use the date filter at the top rather than baking dates into each domain.

## Step 6 — Get the numbers out

Charts download as PNG, any item or the whole dashboard exports to `.xlsx`, and the dashboard
prints to PDF from the browser — useful for the Monday meeting where somebody always asks for
"the file".

## The mistake worth avoiding

Grouping by a field the database cannot group on — a one2many, or a non-stored computed field —
gives an empty chart. Group by something stored: a many2one, a selection, a date, a stored
number. If a field does not appear in the Group By list, that is why.

---

*Written by Thinh Nguyen Huu, author of
[May17 Dashboard](https://apps.odoo.com/apps/modules/19.0/may17_dashboard) — 17 item types,
10 one-click templates, 3 dependencies, Odoo 19 Community and Enterprise.*
