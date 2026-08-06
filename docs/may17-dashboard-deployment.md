# May17 Dashboard — deployment and administration guide

**This guide is for whoever runs the Odoo server**, not for the person building dashboards. It
covers pointing dashboard reads at a database replica, how the two permission groups work, and how
to diagnose the two problems that actually come up.

For what the module does and how to build a dashboard, see the
[product page](may17-dashboard.md).

Applies to **May17 Dashboard 19.0.7.8.0** and later, on Odoo 19 Community and Enterprise.

---

## Serving dashboard reads from a read replica

A dashboard is aggregation — grouped counts and sums over whole tables. That is the shape of query
that slows a primary database down while people are trying to work in it. From 19.0.7.8.0 every
route that only reads is marked read-only, so Odoo serves it from its read-only connection pool,
which points at the replica.

### What to configure

Nothing in the module. Two options on the Odoo server:

```ini
[options]
db_replica_host = replica.internal      ; hostname of the standby
db_replica_port = 5432
```

The same values can come from the environment instead, which is usually easier in a container:

```
PGHOST_REPLICA=replica.internal
PGPORT_REPLICA=5432
```

The replica uses the same database name, user and password as the primary.

### If you do not configure a replica

Everything keeps working. Odoo's `Registry.cursor(readonly=True)` falls back to a read/write cursor
on the primary when no replica exists, so a single-database deployment runs exactly the same code
with no switch to turn on and nothing to undo.

### Which routes go where

| Route | Database |
|---|---|
| `fetch_dashboard_data` | replica |
| `fetch_item_data` | replica |
| `get_dashboard_list` | replica |
| `get_list_data_offset` | replica |
| `get_templates` | replica |
| `export_xlsx` | replica |
| Creating, editing, deleting, reordering, saving layout, ticking a to-do | primary |

Building and editing dashboards always reaches the primary. Only viewing them moves.

### Replication lag: agree this with your users first

A replica is behind the primary — usually by well under a second, but it is never exactly current.
Two consequences worth stating before anyone reports them as bugs:

- A record created a moment ago may not be counted on the dashboard yet.
- **Editing a dashboard writes to the primary and the next read comes from the replica**, so an
  administrator who saves an item and immediately refreshes can briefly see the previous version.
  Refreshing again resolves it.

For aggregated figures this is the trade being made deliberately: slightly stale numbers in
exchange for not loading the database everyone else is working in. If a dashboard must be exact to
the second, do not configure a replica — the module then reads the primary as before.

## How to prove reads really come from the replica

Marking a route read-only only promises it does not write. It proves nothing about *which* server
answered. So the module ships a route that asks PostgreSQL directly, over the same connection every
dashboard read uses: `pg_is_in_recovery()` is **true on a standby and false on a primary**.

Sign in as a Settings administrator and call it:

```bash
curl -s -b cookies.txt -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","method":"call","params":{}}' \
     https://your-odoo/may17_dashboard/replica_status
```

The reply names the server and states the verdict in plain language:

| `served_by_replica` | `replica_configured` | `cursor_readonly` | What it means |
|---|---|---|---|
| `true` | `true` | `true` | **Working.** Dashboard reads are answered by the standby. |
| `false` | `true` | `true` | Configured, but this read still came from the primary — `db_replica_host` points at the primary, or the server was not restarted. |
| `false` | `true` | `false` | Configured, but the route was not served read-only. Check the module is 19.0.7.9.0 or later. |
| `false` | `false` | `false` | **No replica configured — nothing is wrong.** Reads go to the primary, which is correct on a single-database deployment. |

The second row is the one worth watching for: Odoo connects happily to whatever host it is given,
so a replica host that actually points at the primary looks completely healthy while offloading
nothing. Nothing in the log would tell you.

`cursor_readonly` is `false` on the last row by design, not by fault: Odoo opens a read-only
connection pool only once `db_replica_host` is set, so there is no read-only connection to be
served on until you configure one.

### A second, cheaper check: that no read secretly writes

A route marked read-only that writes anyway still works — PostgreSQL refuses the write, Odoo opens
a connection to the primary and retries the whole request — but it is then served twice, and half
the benefit is gone. To confirm that never happens, start Odoo with:

```
odoo-bin --dev=replica
```

which makes the read-only cursor genuinely read-only against the same database, open every kind of
dashboard, then search the log:

```
grep "retrying with a read/write cursor" odoo.log
```

**No matches is the correct result.**

### Confirming from the database side

If you would rather not trust the application at all, watch the standby while someone opens a
dashboard:

```sql
-- run this on the replica
SELECT count(*), state, query
FROM pg_stat_activity
WHERE datname = '<your database>'
GROUP BY state, query;
```

Aggregation queries against your business tables should appear there, not on the primary.

---

## Permissions

The module ships two groups, under **Dashboards** in a user's access rights:

| Group | Can |
|---|---|
| **User** | Open dashboards shared with them. Cannot create or configure anything. |
| **Manager** | Everything a User can, plus create dashboards, configure items and manage menus. |

Every internal user is a Dashboard **User** automatically, so a dashboard shared with a role is
visible without an administrator granting anything first. Assign **Manager** to the people who
build dashboards.

### Why Manager needs to read field metadata

Configuring an item means choosing a model and a field, so a Manager needs read access to
`ir.model.fields` and `ir.model` — Odoo 19 gives ordinary internal users none. From 19.0.7.7.1 the
module grants that read, **to the Manager group only**. It is deliberately not granted to the User
group, because every internal user is implied into that one and granting it there would expose the
whole database schema to everybody.

Viewing a dashboard needs no such right: an item's configuration is read on the viewer's behalf
after their permission to see the item has been checked.

### Record rules

Each dashboard carries a company and record rules isolate data per company. The domain variables
`%UID` and `%MYCOMPANY` resolve at render time, so one dashboard definition shows each person their
own figures.

---

## Troubleshooting

### Every card says "You are not allowed to access 'Fields' (ir.model.fields) records"

Charts, KPIs and list views fail while simple count tiles still work, and administrators see a
perfectly good dashboard.

**Cause:** a version before 19.0.7.7.1. Odoo 19 gives internal users no access at all to
`ir.model.fields`, and an item stores each of its "which field" settings as a link to that model,
so an ordinary user could not read the item's own configuration. Count tiles survived because a
count is the one computation that never reads a field name.

**Fix:** upgrade to 19.0.7.7.1 or later and update the module (`-u may17_dashboard`). No data
migration and no configuration change.

### Dashboards still load from the primary

Call `/may17_dashboard/replica_status` first — it tells you which of the four states you are in
without guessing. Then check, in order:

1. `db_replica_host` is set in the config file the server actually loaded — not a different one.
2. **That host is a standby, not the primary.** Run `SELECT pg_is_in_recovery();` against it; it
   must return `true`. This is the most common mistake, and nothing else reports it.
3. The server was restarted after the change.
4. The module is 19.0.7.9.0 or later; earlier versions do not mark any route read-only.
5. The log has no `retrying with a read/write cursor` warnings, which would mean a read route is
   writing and being retried on the primary.

### A dashboard shows figures a user should not see

Dashboard items read business data with elevated rights by design, so that a shared dashboard shows
the same total to everyone it is shared with. Control who sees a dashboard by sharing it with the
right roles, not by relying on each viewer's record rules over the underlying model.

---

## Upgrading

Update the module after copying a new version into the addons path:

```
odoo-bin -u may17_dashboard -d <database> --stop-after-init
```

The module modifies no Odoo core file and stores its data in its own tables, so upgrades carry no
migration step.

---

## Support

**huuthinh17596@gmail.com** — answered within 48 hours by the author. Bug fixes are included for
the purchased Odoo series.

---

*Odoo dashboard read replica · Odoo db_replica_host · Odoo readonly route · Odoo dashboard
permissions · Odoo dashboard troubleshooting · May17 Dashboard administration*
