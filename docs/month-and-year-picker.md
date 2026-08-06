# Month and Year Picker — month-only and year-only widgets for Odoo 19 date fields

**Month and Year Picker is a paid Odoo 19 module (Community and Enterprise) that adds four field
widgets showing only the month or only the year on date and datetime fields**, hiding the day
entirely. A billing period reads "March 2026" instead of "04/03/2026".

🛒 **[Get it on the Odoo Apps Store](https://apps.odoo.com/apps/modules/19.0/may17_month_picker)** — $9, one-time.

## The four widgets

| Widget | Field type | Shows |
|---|---|---|
| `month` | date | March 2026 |
| `month_datetime` | datetime | March 2026 |
| `year` | date | 2026 |
| `year_datetime` | datetime | 2026 |

## How to use

```xml
<field name="billing_month" widget="month"/>
<field name="report_period" widget="month_datetime"/>
<field name="fiscal_year" widget="year"/>
<field name="budget_year" widget="year_datetime"/>
```

The widgets also appear in the Odoo Studio widget selector, so a period field can be switched to a
month picker without touching XML.

## Where it is used

Billing month, invoice month, subscription period, payroll month, attendance period, fiscal year,
budget year, contract year, accounting period, production planning — anywhere the day carries no
meaning and only invites wrong input.

## Installing it — where to find it in the Apps list

Odoo's Apps screen opens with an **Apps** filter already switched on, and that filter shows only
full applications. This module is a field widget rather than an application, so it does not appear
until you take the filter off. That single step is what people miss.

1. Open **Apps** and press **Update Apps List**.
2. Click the small **x** on the **Apps** tag in the search box to remove the filter.
3. Search for **Month and Year Picker**, or for the technical name `may17_month_picker`.
4. Press **Install**.

There is no menu to open afterwards — the widget becomes available on date fields, so the next step
is putting `widget="month"` on one.

## Technical details

- Frontend only: 2 source files, no Python model, no database change
- Stores a standard date value (the first day of the selected month or year), so reports, filters
  and groupings keep working
- Works in form, list and kanban views
- Depends on `web` only
- No Odoo core modification

## Frequently asked questions

### Does it change how the date is stored?

No. The field keeps a normal date value — the first day of the chosen month or year. Only the
display and the picker change, so every existing filter, group-by and report still works.

### Can I use it on a field created with Odoo Studio?

Yes. The widgets are listed in the Studio widget selector for date and datetime fields.

### Why not just use a character field for the period?

A character field cannot be filtered by range, grouped by quarter, or compared to another date.
Keeping a real date and hiding the day gives you the readable label and the maths.

### Which Odoo versions are supported?

Odoo 19, Community and Enterprise.

---

*Odoo month picker · Odoo year picker · month and year picker Odoo 19 · hide day from date field ·
Odoo date widget month only · fiscal year picker Odoo*
