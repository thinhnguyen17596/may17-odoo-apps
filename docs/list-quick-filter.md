# List Quick Filter — inline column filter bar for Odoo 19 list views

**List Quick Filter is a paid Odoo 19 module (Community and Enterprise) that adds a configurable
filter bar directly above any list view**, so a user filters records by several columns at once
without opening Odoo's advanced search panel.

🛒 **[Get it on the Odoo Apps Store](https://apps.odoo.com/apps/modules/19.0/list_quick_filter)** — $40, one-time.

## What it does

Odoo's search panel is powerful and hidden. A user who wants "orders for this customer, in this
state, over this amount" has to open the panel, add three filters, and remember how. List Quick
Filter puts those fields on the screen, above the rows, where they are obvious.

| | |
|---|---|
| **Operators** | 8 — equals, contains, greater than, less than and more |
| **Field types** | many2one and many2many with autocomplete tags, selection, boolean, date, datetime, text, numbers |
| **Scope** | any model — Contacts, Sales, Purchase, Invoicing, Inventory, or your own |
| **Configuration** | per action, so each menu carries its own filter set |
| **Dependencies** | 2 — `base`, `web` |
| **Odoo core files modified** | 0 |

## Key features

- Inline filter bar above any list (tree) view
- Multi-field filtering with AND logic — every filter narrows the result further
- Many2one and many2many autocomplete with multi-select tags
- Selection and boolean dropdowns, date and datetime pickers
- Per-action configuration: the same model can carry different filters in different menus
- Drag-and-drop ordering of the filter fields
- Custom label per field
- Dark mode support and a responsive layout for tablets
- No Odoo core modification — safe for upgrades

## Frequently asked questions

### How is this different from Odoo's own search panel?

Odoo's panel is a dropdown a user has to open, and its filters are defined by a developer in the
view. List Quick Filter shows the fields permanently above the list, and an administrator chooses
those fields per menu from the interface — no view inheritance, no XML.

### Can I use it on a custom model?

Yes, on any model in the database, including models created with Odoo Studio.

### Does every list view get the filter bar?

No. You configure which action gets which fields, so the bar appears only where it is useful.

### Which Odoo versions are supported?

Odoo 19, Community and Enterprise.

### Is support included?

Yes. Bug fixes for the purchased Odoo series, questions answered within 48 hours by the author:
**huuthinh17596@gmail.com**

---

*Odoo list view filter · Odoo column filter · Odoo quick search · Odoo tree view filter ·
inline filter bar Odoo 19*
