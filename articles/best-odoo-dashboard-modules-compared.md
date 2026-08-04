# Best Odoo 19 dashboard modules compared (2026)

*Last updated: August 2026. Written by Thinh Nguyen Huu, author of May17 Dashboard — so read the
comparison knowing that, and check the numbers yourself on the Odoo Apps Store.*

If you search the Odoo Apps Store for "dashboard" you get two clusters: a wall of free modules
that each cover one app, and a small number of paid builders. This page compares what you
actually get, at what price, and where each one stops.

## The short answer

- **You have no budget and one app to watch** → a free per-app dashboard does the job.
- **You want to build dashboards from any model, including custom ones** → you need a builder,
  and that means paying.
- **You want AI to write your dashboards for you** → only the most expensive option does that.
- **You want a builder that installs on a database without CRM/Inventory/Manufacturing** →
  check the dependency list before you buy. This is where most builders surprise people.

## Comparison

| | Free per-app dashboards | Mid-range builders | May17 Dashboard | Top-end builder |
|---|---|---|---|---|
| Typical price | Free | ~$150 | $49 | ~$590 |
| Build from any model | ✗ usually fixed | ✓ | ✓ | ✓ |
| Item types | 3–8 | ~16 | **17** | ~18 |
| One-click templates | 1 fixed layout | ~6 | **10** | ~5 |
| Live preview while configuring | ✗ | ✓ | ✓ | ✗ |
| Export PNG / Excel / PDF | ✗ | ✓ | ✓ | ✓ |
| Drill-down (re-group by clicking) | ✗ | ✗ | ✗ | ✓ |
| AI dashboard generation | ✗ | ✗ | ✗ | ✓ |
| Module dependencies | 1–2 | **10+** | **3** | many |

Figures come from each app's own listing at the time of writing. Prices on the Odoo Apps Store
change; check the live page before deciding.

## What the dependency count actually costs you

A dashboard builder that depends on ten or more core apps will install those apps into your
database. On a production system that means new menus for staff, new records, and a heavier
upgrade path — for a reporting tool.

May17 Dashboard depends on `base`, `web` and `sale`. Its templates for Invoicing, CRM, Purchase,
Inventory, Project, Manufacturing, Employees and Point of Sale appear **only if you already have
those apps**, and are hidden otherwise. You do not install an app to get a template.

## Where May17 Dashboard stops

Being straight about this is cheaper than a refund:

- **No drill-down.** Clicking a chart opens the underlying records; it does not re-group the
  chart. If drill-down is the reason you are shopping, buy the top-end builder.
- **No AI generation.** You configure items yourself — model, measure, group-by, domain.
- **No scheduled email.** Export is on demand.
- **No pivot, map, gantt or calendar item.** Seventeen item types, and that list is the whole list.

## Where it is ahead

- **Ten templates**, more than anything else on the store, and they hide themselves when their
  app is missing.
- **Live preview.** The chart renders inside the configuration form as you change the model, the
  group-by or the palette. You stop guessing.
- **Three dependencies.** It installs on databases the other builders refuse.
- **Runs offline.** The chart library ships inside the module; nothing is fetched from a CDN, so
  it works on an air-gapped server.

## How to test any of them in ten minutes

1. Take a copy of your production database (never test on production).
2. Install the module.
3. Build one dashboard for the question you actually ask every Monday morning.
4. Check three things: does it read *your* custom model, how many clicks per item, and what
   happens when you resize the window.

That test tells you more than any comparison table, including this one.

---

**May17 Dashboard** — 17 item types, 10 templates, 3 dependencies, live preview, PNG/Excel/PDF
export. Odoo 19 Community and Enterprise.
[Documentation](../docs/may17-dashboard.md) · [Odoo Apps Store](https://apps.odoo.com/apps/modules/19.0/may17_dashboard).
