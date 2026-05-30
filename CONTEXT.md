# Rainbow Springs App -- Project Context

## Goal

PWA (Progressive Web App) that receives Florida State Parks reservation
confirmation emails via forwarding, stores reservation data, and displays
Code 128 barcodes full-screen at the park gate -- working offline, no
email-hunting, no pinching to zoom.

**Why:** Park gates scan a Code 128 barcode; cell service is unreliable at
the gate; finding the confirmation in email is slow and awkward.

## Architecture (current design)

### Ingestion

User forwards a Florida State Parks confirmation email to `intake@rsdp.com`.

- First-time sender: server replies with an 8-digit pairing code + QR image
  to register the app
- Known sender: email is parsed and pushed to the user's device

Three-level parsing pipeline (see `docs/architecture-decisions.md` D5):

1. HTML/CSS class targeting (USeDirect template classes)
2. Plain text + LLM structured extraction
3. Raw email fallback (surfaced in app, no data loss)

### Sync

- **Primary:** Web Push notification on email receipt → Service Worker syncs
  to IndexedDB
- **Safety net:** 6 AM local device time daily

### Display

PWA installed to home screen. Opens directly to today's barcode (full-screen,
no scroll, no zoom). One tap to reservation list.

---

## Target Users

Annual pass holders at Rainbow Springs State Park who make their own day-use
reservations. Small group. Not public. One-off visitors are out of scope (v1).

---

## Key Constraints

- Barcode display must work offline at the gate
- No App Store maintenance -- PWA only
- No API available from Tyler Technologies (Recreation Dynamics / USeDirect)
- Code 128 format confirmed by proof of concept
- Single email address per account (v1)

---

## Data Model

Composite primary key: `(confirmation_number, date)` -- handles multi-day
reservations on a single confirmation number as separate display records.

Fields: confirmation_number, date, park_name, space, check_in, check_out,
person_name, person_email, classification, pass_number, status,
parse_level, raw_email_ref.

---

## Proof of Concept (superseded for data capture)

Google Sheets + Claude skill approach documented in `docs/proof-of-concept.md`.
Code 128 barcode generation and display confirmed working. Data capture via
screenshot + Claude skill is replaced by email ingestion in the new architecture.

---

## Docs

- `docs/architecture-decisions.md` -- decisions, rejected options, open questions, TODO list
- `docs/ux-vision.md` -- gate experience goals and non-goals
- `docs/proof-of-concept.md` -- existing skill + Sheet + Apps Script
- `docs/data-capture.md` -- prior scraping options (superseded; kept for context)
- `docs/lovable-spec.md` -- Lovable app spec (Phase 1/2 still valid for display layer)
