# Architecture Decisions -- Email Ingestion + PWA

Decisions, rejected options, and open tasks from the May 2026 design session.
Supersedes the bookmarklet/scraping approach in `data-capture.md`.

---

## Context

The Florida State Parks reservation portal (`reserve.floridastateparks.org`) runs
on Tyler Technologies "Recreation Dynamics" (formerly USeDirect v11.15.5.0),
an ASP.NET WebForms SaaS. There is no public API. All state is encoded in
`__VIEWSTATE`. Any scraping approach requires maintaining an authenticated
session and replaying WebForms POSTs -- fragile and ToS-risky.

The pivot: use the confirmation email the portal already sends as the data
transport layer. Email is durable, ToS-safe, and carries all fields needed
for barcode display.

---

## Decisions

### D1 -- Email ingestion via forwarding, not browser automation

**Decision:** Users forward Florida State Parks confirmation emails to
`intake@rsdp.com`. The server parses them and pushes data to the app.

**Why:** No API exists. Browser scraping requires stored credentials and
breaks on UI changes. The confirmation email already contains all required
fields (confirmation number, date, space, name, pass number).

**Forwarding options:**

- Manual forward (minimum viable -- anyone who has forwarded an email can do this)
- Mail.app automated rule (toil reducer for power users; documented but not required)

**Rejected:** Per-user forwarding address (`red-mountain-goat@rsdp.com`).
Initial design used unique addresses per user for implicit routing. Rejected
because FROM: address verification on a single `intake@rsdp.com` provides
equivalent authentication with less complexity and no namespace to manage.

---

### D2 -- FROM: address is the account identity (v1)

**Decision:** The user's email address is the account primary key. One email
address per account. Incoming email to `intake@rsdp.com` is routed by matching
FROM: against registered accounts.

**Why:** Florida State Parks allows one email address per account. Our target
users are annual pass holders with established park accounts -- not one-off
visitors. Keeping email = account identity avoids a join table for v1.

**Rejected scope:** Multi-email-per-account. Some users have multiple email
clients and might forward from a different address than they registered with.
Not served in v1.

**Future work (TODO-01):** Add `account_emails` join table to support multiple
FROM: addresses per account when the use case justifies it.

---

### D3 -- Unknown sender triggers registration, not an error

**Decision:** Email arriving at `intake@rsdp.com` from an unrecognized address
triggers the registration flow: server replies with a pairing code (8-digit
text + QR image) that the user enters in the app.

**Why:** Eliminates a separate `register@rsdp.com` address and a separate
registration step. The first forward IS the registration.

**Rejected:** `register@rsdp.com` as a dedicated registration address. Adds
an address to maintain and a step users must know about before using the
service.

---

### D4 -- One-off visitors are out of scope (v1)

**Decision:** Users who make a single reservation without a Florida State Parks
annual pass account are not served. They can use the confirmation email directly.

**Why:** They are not the target market. Supporting them requires handling
multiple unrelated email addresses per person (each one-off might come from a
different account) and adds complexity without commensurate value.

**Future work (TODO-02):** Revisit if user research shows meaningful demand.

---

### D5 -- Three-level email parsing pipeline

**Decision:** Parse each incoming email through three levels in order:

1. **HTML/CSS targeting** -- extract fields using known USeDirect CSS class
   names (`hd_label_text_sc`, `hd_label_text_nb`, `ys_box_subtitle`,
   `lblFooterTitle`, `lblFooterText`). Fast, deterministic, brittle.
2. **Text + LLM extraction** -- strip HTML to plain text, send to LLM with a
   structured extraction prompt. Slower, more robust to template changes.
3. **Raw email fallback** -- store and surface the original email in the app
   for the user to read manually. No data loss; preserves the official document.

Raw email is always stored regardless of parse level, enabling re-parse if
Level 1 logic improves after a template change.

**Rejected:** Single-strategy parsing. Template changes are inevitable with
no change notifications from Tyler Technologies.

---

### D6 -- Composite key: (confirmation_number, date)

**Decision:** Reservations are uniquely identified by `(confirmation_number,
date)`. This handles multi-day reservations (a single confirmation number
covering consecutive dates) as separate display records, one per day.

**Why:** Display logic is date-centric -- "show today's barcode." Each date
must be independently selectable. A camping reservation for 3 nights on one
confirmation number should produce 3 displayable records.

---

### D7 -- Multiple reservations on the same date: warn and defer

**Decision:** If parsing yields more than one reservation for the same date,
warn the user in the app and show the first one. Do not crash or silently pick.

**Why:** Extremely unlikely for day-use parking, but the data model must not
assume it cannot happen.

**Future work (TODO-03):** Design a selection UI for same-day multiple
reservations when a real case is observed.

---

### D8 -- PWA, not native apps

**Decision:** Build as a Progressive Web App. Single codebase, no App Store
submissions, installs to home screen on both iOS and Android.

**PWA = Progressive Web App:** A website that behaves like a native app.
User visits the URL in Safari or Chrome, taps "Add to Home Screen," and it
installs -- launches full-screen with no browser chrome, works offline via
Service Worker + IndexedDB, receives push notifications.

**Why:** No separate iOS and Android codebases to maintain. The target use
case (full-screen barcode display, offline data) is achievable in a PWA.

**Rejected:** React Native / Flutter / Capacitor. All require App Store and
Play Store submissions and separate build pipelines.

**Known PWA limitations accepted:**

- iOS push notifications require "Add to Home Screen" (iOS 16.4+). Onboarding
  must prompt for this explicitly.
- Screen brightness not controllable from web APIs. Not addressed -- user
  responsibility.

---

### D9 -- Server push replaces client-side cron as primary sync

**Decision:** When `intake@rsdp.com` receives and parses a confirmation email,
the server sends a Web Push notification to the user's device. The Service
Worker wakes and syncs to IndexedDB. This is the primary sync trigger.

**Safety-net sync:** 6 AM local device time daily, in case a push was missed
(device offline overnight, push delivery failure). Park opens at 8 AM local
time; 6 AM provides margin.

**Why:** Event-driven is faster (user gets their pass minutes after forwarding)
and more reliable than polling. "1970s-era cron" retained only as fallback.

**Rejected:** Client-side cron as primary mechanism. Too slow (user waits
until next morning for new reservations to appear), unreliable on iOS (OS
kills background web processes).

---

### D10 -- Cancellation/modification emails: future work

**Decision:** v1 does not handle cancellation or modification confirmation
emails. Cancelled reservations remain in the local store until the date passes.

**Risk acknowledged:** A user who cancels a reservation will still see a
(now-invalid) barcode until it ages out. Acceptable for v1 given the small
user base.

**Future work (TODO-04):** Parse cancellation emails (distinguish by subject
line pattern), tombstone the corresponding `(confirmation_number, date)` record.

---

### D11 -- Barcode format: Code 128

**Decision:** Encode the confirmation number as a Code 128 barcode.

**Evidence:** Proven in the existing proof of concept (Google Sheets barcode
tab). Rangers at Rainbow Springs use a handheld scanner that reads Code 128.

---

### D12 -- Geo-fencing: future work

**Future work (TODO-05):** Trigger a sync and/or bring the barcode screen
forward automatically as the user drives toward the park entrance. Requires
background location permission -- significant UX and permission complexity;
deferred to a later phase.

---

## Data Model (v1)

```sql
reservations
  confirmation_number  TEXT
  date                 TEXT  (YYYY-MM-DD)
  park_name            TEXT
  space                TEXT  (e.g. "P 022")
  check_in             TEXT  (e.g. "08:00 AM")
  check_out            TEXT  (e.g. "Sunset")
  person_name          TEXT
  person_email         TEXT
  classification       TEXT  (e.g. "Annual Entrance Pass Holder")
  pass_number          TEXT  (e.g. "586415")
  status               TEXT  (e.g. "Active")
  parse_level          INT   (1, 2, or 3 -- which parser succeeded)
  raw_email_ref        TEXT  (pointer to stored raw email)
  PRIMARY KEY (confirmation_number, date)

accounts
  email                TEXT  PRIMARY KEY
  pairing_code         TEXT
  device_push_token    TEXT
  created_at           TEXT
```

---

## Address Map

| Address           | Purpose                                                                         |
|-------------------|---------------------------------------------------------------------------------|
| `intake@rsdp.com` | Receive forwarded confirmation emails; trigger registration for unknown senders |
| `info@rsdp.com`   | General questions                                                               |
| `help@rsdp.com`   | User support                                                                    |
| `bug@rsdp.com`    | Bug reports                                                                     |

---

## Open Questions

- **OQ-01:** Does Mail.app forwarding preserve the original HTML body intact,
  or wrap it in an outer HTML envelope? Level 1 parsing depends on the answer.
- **OQ-02:** Is the confirmation number the only field encoded in the Code 128
  barcode, or does the ranger scanner expect additional data?
- **OQ-03:** What is the exact FROM: address on Florida State Parks confirmation
  emails? (The server sends FROM: FL parks, but forwarded email has FROM: user --
  need to confirm Mail.app forwarding behavior preserves this correctly.)

---

## TODO

- TODO-01: Multi-email-per-account (`account_emails` join table)
- TODO-02: One-off visitor support (non-annual-pass users)
- TODO-03: Same-day multiple-reservation selection UI
- TODO-04: Cancellation/modification email handling
- TODO-05: Geo-fence sync trigger approaching park entrance
- TODO-06: Mail.app rule setup guide (forwarding works manually; rule is toil reduction)
- TODO-07: Confirm OQ-01, OQ-02, OQ-03 before building parser
