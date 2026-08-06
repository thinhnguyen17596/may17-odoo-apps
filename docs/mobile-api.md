# Mobile App API and Sync — REST API reference for Odoo 19

**Mobile App API and Sync is a paid Odoo 19 module (Community and Enterprise) that turns an Odoo
database into a backend a mobile app can talk to:** a versioned REST API, one revocable token per
device, offline synchronisation and push notification delivery.

🛒 **[Get it on the Odoo Apps Store](https://apps.odoo.com/apps/modules/19.0/mobile_api)** — $149, one-time.

This page is the API reference. Every request and response below was taken from a running server,
not written from the source.

## Why it exists

Odoo already has JSON-RPC and it already has API keys. What it does not have is the three things a
phone app needs:

| A mobile app needs | Odoo gives you |
|---|---|
| A credential per device, revocable on its own | One API key per **user**, global, equal to the password over XML-RPC |
| To know which phone is calling | Nothing — a key carries no device identity |
| "What changed since I was last online?" | Nothing |
| Push notifications | Nothing |
| A notification the user can come back to and mark read | Nothing |

## At a glance

| | |
|---|---|
| **Base URL** | `https://your-odoo/api/mobile/v1` |
| **Transport** | Plain HTTP, JSON body, real status codes — not Odoo's JSON-RPC envelope |
| **Method** | `POST` on every endpoint |
| **Auth** | `Authorization: Bearer <token>` |
| **Content type** | `application/json` |
| **Row cap** | 500 per call |

Responses are the JSON object itself. There is no `result` wrapper to unpack:

```json
{"ok": true, "model": "res.partner", "records": [ ... ]}
```

Errors carry the reason in the status line as well as the body, so an app can branch on the status
code without parsing prose:

```json
{"ok": false, "error": {"code": "invalid_token", "message": "Sign in again."}}
```

| Status | `code` | Means |
|---|---|---|
| `200` | — | Success |
| `400` | `bad_request` | The body was not JSON, or an argument was missing or wrong |
| `401` | `invalid_credentials` | Wrong login or password at registration |
| `401` | `invalid_token` | Missing, expired or revoked token — sign in again |
| `403` | `forbidden` | The model or field is not exposed, or the user is not allowed |

## Setting it up first

Nothing is reachable until an administrator says so. In Odoo, open **Mobile API → Exposed Models**
and add each model the app needs, ticking the fields and the operations it may use.

An empty field list exposes **nothing**, not everything — a half-finished configuration cannot leak
a table.

---

## `POST /auth/register`

Exchange a password for a device token. The password is sent once, here; the app stores the token
and never the password.

**Request**

```json
{
  "login": "nam@example.com",
  "password": "••••••••",
  "device_uid": "9f1c2e77-...",
  "device_name": "Nam - iPhone 15",
  "platform": "ios",
  "app_version": "1.0.0"
}
```

`device_uid` is generated once by the app and kept for the life of the install. Registering twice
from the same device replaces the previous token instead of leaving a second working one behind.

**Response `200`**

```json
{
  "ok": true,
  "token": "e0efd85e0a8a5af36c0880a8d4d746d1384c0...",
  "expires_on": "2026-11-04 05:05:40",
  "device_id": 1,
  "user": {"id": 5, "name": "Mobile User", "login": "nam@example.com"}
}
```

**The token is returned here and nowhere else, ever again.** Only a hash is stored. If the app
loses it, register again.

**Response `401`**

```json
{"ok": false, "error": {"code": "invalid_credentials", "message": "Wrong login or password."}}
```

The reply is identical whether the login is unknown or the password is wrong. Telling them apart
would let someone check which email addresses have accounts.

Registration goes through Odoo's own login path, so repeated failures are throttled and logged
exactly like failed web logins.

---

## `POST /auth/me`

Confirm the token still works, and find out who it belongs to. Useful at app start.

**Request** `{}` · **Headers** `Authorization: Bearer <token>`

**Response `200`**

```json
{
  "ok": true,
  "user": {"id": 5, "name": "Mobile User", "login": "nam@example.com"},
  "device": {"id": 1, "name": "Nam - iPhone 15"},
  "expires_on": "2026-11-04 05:05:40"
}
```

A `401` here means the token expired, was revoked, or the device was deleted. Send the user back to
the sign-in screen.

---

## `POST /auth/revoke`

Sign this device out. The token stops working on the very next call.

**Request** `{}` → **Response** `{"ok": true}`

Revoking one device does not touch the person's other devices, their web session, or their
password.

---

## `POST /models`

Ask what this installation exposes. Call it once at startup instead of hard-coding a schema — an
administrator can then expose another model without anybody shipping a new build to the stores.

**Response `200`**

```json
{
  "ok": true,
  "models": [
    {
      "model": "res.partner",
      "label": "Contact",
      "fields": ["id", "name", "email", "phone", "city"],
      "can_read": true,
      "can_create": true,
      "can_write": true,
      "syncable": true
    }
  ]
}
```

---

## `POST /read`

Search one exposed model.

**Request**

```json
{"model": "res.partner", "domain": [["city", "=", "Hanoi"]], "limit": 50, "offset": 0, "order": "name asc"}
```

`domain` is an ordinary Odoo domain, as a JSON array. It can only **narrow** what the administrator
already allowed — the configured filter always applies on top.

**Response `200`**

```json
{"ok": true, "model": "res.partner", "records": [{"id": 7, "name": "Acme Ltd", "email": false, "phone": false, "city": "Hanoi"}]}
```

Only exposed fields come back, always including `id`. `limit` is capped at 500.

---

## `POST /sync`

The endpoint the offline story is built on. One call returns everything that changed since the app
was last online.

**Request**

```json
{"model": "res.partner", "since": "2026-08-06 05:07:23"}
```

Omit `since` for the first, full download.

**Response `200`**

```json
{
  "ok": true,
  "model": "res.partner",
  "changed": [ {"id": 7, "name": "Acme Ltd", "city": "Hue", ...} ],
  "ids": [1, 7, 8, 9],
  "count": 4,
  "cursor": "2026-08-06 05:14:06",
  "complete": true
}
```

| Field | What the app does with it |
|---|---|
| `changed` | Insert or update each of these locally |
| `ids` | The complete set the app should be holding — **delete anything local that is not in here** |
| `cursor` | Send it back as `since` on the next sync |
| `count` | How many records are in scope |
| `complete` | `false` means more changes are waiting; sync again immediately |

### How deletions are handled, and the limit of it

Odoo keeps no tombstones. A deleted row simply is not there, so there is nothing to send. The
alternative — a deletion log — would put a write in the path of every `unlink` in the database,
which is a heavy price for every user of the system to pay for the benefit of the phone ones.

So `ids` carries the authoritative set instead, and the app removes whatever is missing. Ids are
small, so this stays cheap into the tens of thousands of rows. Watch `count`: when a model grows
past roughly fifty thousand records, the id list becomes the expensive part of the response and it
is time to narrow the exposed filter for that model.

### Always sync with the server's cursor, never the phone's clock

Use the `cursor` from the previous response as `since`. A phone's clock can be minutes off, and a
clock that is fast will skip records. The cursor is read on the server *after* the queries run, so
a record written while the sync was in flight is picked up next time rather than missed.

---

## `POST /create`

**Request**

```json
{"model": "res.partner", "values": {"name": "From Phone", "city": "Da Nang"}}
```

**Response `200`** → `{"ok": true, "model": "res.partner", "id": 10}`

**Response `403`** if a field is not exposed:

```json
{"ok": false, "error": {"code": "forbidden", "message": "These fields are not exposed to the mobile app: is_company"}}
```

...or if the **user** is not allowed to create that model, even when the administrator ticked
"Create". Both have to agree. See *Security* below.

---

## `POST /write`

**Request**

```json
{"model": "res.partner", "ids": [7], "values": {"city": "Hue"}}
```

**Response `200`** → `{"ok": true, "model": "res.partner", "written": 1}`

Up to 500 ids per call, and the same values are written to all of them.

---

## `POST /push/register`

Store the FCM or APNs token for this device, after the user grants notification permission.

**Request** `{"push_token": "fcm-token-..."}` → **Response** `{"ok": true}`

Send `null` to stop notifications for this device without signing it out.

---

## `POST /notifications`

List this user's notifications, newest first. A push is a banner that may never
arrive, or may be swiped off a lock screen and never seen again; this is where
the message actually lives, so the app can show a list and a badge whether or
not the push got through.

**Request**

```json
{"limit": 50, "offset": 0, "unread_only": false}
```

**Response `200`**

```json
{
  "ok": true,
  "unread": 2,
  "notifications": [
    {
      "id": 6,
      "title": "Đơn hàng SO0042",
      "body": "Administrator: Khách *yêu cầu giao sớm* 2 ngày.",
      "res_model": "res.partner",
      "res_id": 9,
      "category": "message",
      "is_read": false,
      "read_on": null,
      "create_date": "2026-08-06 13:03:36"
    }
  ]
}
```

`unread` is sent on every list so the badge stays right without a second call.
Empty fields are `null`, never `false` — a typed client cannot decode a field
that is sometimes a boolean and sometimes a string. `res_model` and `res_id`
are what the app deep-links on when the row is tapped.

`category` is `"message"` for a mirrored Odoo message, `"activity"` for an
assigned activity, and whatever an automated action passed for anything else.

---

## `POST /notifications/read`

Mark notifications read, or all of them at once.

**Request** `{"ids": [6, 7]}` or `{"all": true}`

Pass `{"ids": [6], "read": false}` to mark one unread again.

**Response `200`** → `{"ok": true, "updated": 2, "unread": 0}`

`all` exists because an app cannot implement "mark all read" by listing first
when there are hundreds. The record rule is what stops one user marking
another's, so an id belonging to somebody else is refused rather than ignored.

---

## Where notifications come from

Three sources, and none of them needs the app developer to do anything:

1. **Odoo messages.** Anything Odoo decides a user should be told about — a
   comment on a record they follow, an approval note, a message from a
   colleague — is mirrored to their phone, for every Odoo app installed, with
   no per-app configuration. Odoo already resolves followers and recipients;
   this listens rather than re-deciding, so the app never disagrees with the
   web client about who hears what. HTML is flattened to plain text and cut to
   300 characters.
   Switch it off under **Settings → Mobile API → Send Odoo messages to the app**.
2. **Assigned activities.** When work is assigned to somebody, they are told.
   Activities do not go through the inbox, so without this they are visible
   only to a person already looking at the web client — exactly the person who
   does not need a phone.
3. **Anything else, from Python.** One call, from an automated action, a server
   action or another module:

   ```python
   env['mobile.notification'].notify(
       users,                      # a res.users recordset
       title='Đơn nghỉ phép được duyệt',
       body='Nghỉ phép 12-14/08 đã được quản lý duyệt.',
       res_model='hr.leave', res_id=leave.id,
       category='approval',
   )
   ```

   It stores first and pushes second, so a Firebase outage costs the banner and
   not the message. `push_state` on each row records what happened: `sent`,
   `failed` with the reason, or `no_device` when the person has not installed
   the app yet.

---

## Security

### The token cannot do what a password can

Odoo's own `auth='bearer'` accepts only a **global** API key, which over XML-RPC is equivalent to
the user's password. A token that lives in an app bundle on a phone must be able to do less than
that, so tokens issued here carry their own scope and are refused everywhere else.

Verified against a running server: the same token that works on this API is refused by
`/xmlrpc/2/object` with `Access Denied`.

### The configuration can only narrow, never widen

Every request runs as the person signed in on the phone. Record rules, multi-company scoping and
access rights apply exactly as they do in the web client.

This means both the whitelist and Odoo's own permissions have to agree. Ticking "Create" on a model
does not grant anything: a user without the right group is still refused, and the message says so.
Untick it and even a full administrator is refused through the API.

### What is never reachable

Models holding credentials — `ir.config_parameter`, `ir.mail_server`, `res.users.apikeys` and the
device table itself — are refused outright and cannot be added to the exposed list.

### Reads use the replica where there is one

Every read endpoint is marked read-only, so a deployment with `db_replica_host` configured serves
phones from the replica and they stop competing with the people working in the primary.

---

## A minimal React Native client

```js
const BASE = 'https://your-odoo/api/mobile/v1';

async function call(path, body, token) {
  const res = await fetch(BASE + path, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
    },
    body: JSON.stringify(body),
  });
  const data = await res.json();
  if (res.status === 401) throw new NeedsSignIn(data.error.message);
  if (!res.ok) throw new Error(data.error.message);
  return data;
}

// Sign in once, then keep the token in secure storage - never the password.
const { token } = await call('/auth/register', {
  login, password,
  device_uid: await getStableDeviceId(),
  device_name: Device.deviceName,
  platform: Platform.OS,
});

// Later, every time the app comes online.
async function sync(model, cursor, token) {
  const res = await call('/sync', { model, since: cursor }, token);
  await db.upsert(model, res.changed);
  await db.deleteMissing(model, res.ids);   // this is how deletions arrive
  return res.cursor;                         // store it, send it next time
}
```

Store the token in Keychain or Keystore, not in `AsyncStorage`. Catch `401` in one place and send
the user to the sign-in screen from there.

---

## Frequently asked questions

### Does this module include a mobile app?

No. It is the backend an app talks to. You build the app — React Native, Flutter, native, anything
that can send an HTTP request — and this gives it a safe way in.

### Is it really REST, or Odoo's JSON-RPC?

Plain HTTP. A JSON body in, the JSON object itself out, and real status codes. There is no `result`
wrapper and no JSON-RPC envelope, precisely so a client can act on `401` from the status line.

### What happens when a token expires?

The next call returns `401 invalid_token`. Register again with login and password. Tokens last 90
days by default; change it with the `mobile_api.token_days` system parameter.

### Someone lost their phone. What do I do?

Open **Mobile API → Devices**, find it and press **Revoke**. That token stops working immediately.
Their password, their web session and their other devices are untouched.

### Can two apps share one login?

Yes. Each install sends its own `device_uid` and gets its own token, and each can be revoked
separately.

### Does it work on Odoo Online?

Yes. Providers are reached over plain HTTP and there is no Python package to install, so it runs on
Odoo Online, Odoo.sh and on-premise alike.

### How does the app handle a conflict — both sides changed the same record?

It does not decide for you. `/sync` tells the app what changed on the server; what to do when the
phone also changed it is a product decision, and the module deliberately does not guess. Most apps
either take the server's version or ask the user.

### Is support included?

Yes. Bug fixes for the purchased Odoo series, questions answered within 48 hours by the author:
**huuthinh17596@gmail.com**

---

*odoo mobile api · odoo rest api · odoo 19 rest api · odoo api for mobile app · odoo bearer token ·
odoo device token · odoo offline sync · odoo delta sync · odoo react native · odoo flutter api ·
odoo push notification · build mobile app odoo*
