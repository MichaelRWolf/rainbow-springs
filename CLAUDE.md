# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A PWA (Progressive Web App) called **rsdp.com** (Rainbow Springs Day Pass) for Michael
and a small group of friends with Florida State Parks annual passes. At the entry gate,
rangers scan a Code 128 barcode from your reservation. This app shows the right barcode
full-screen, automatically, with no email-hunting and no pinching to zoom.

## Current State

No application code exists yet. The repo contains design docs and a working proof of
concept that lives in a separate repo.

**v1 Proof of concept** (personal use only, not the architecture for v2) -- `MichaelRWolf/claude-tools`,
skill `extract-rainbow-springs-reservation`:

- Claude skill: Payment Summary screenshots → CSV → Google Sheet
- Google Sheet: `1wLHPw0uB7pe7hw_NJ9hPQWoQ2mt4uyv7_2YaQUt8-UU` (Reservations tab + Barcode tab)
- `reservations.js`: Google Apps Script deduplicate/sort/format, runs `onOpen`

## Architecture (v2 -- Email Ingestion + PWA)

### Data Capture -- Email Forwarding (NOT scraping)

**The scraping/bookmarklet approach was explicitly rejected.** The portal
(`reserve.floridastateparks.org`) runs Tyler Technologies "Recreation Dynamics"
(ASP.NET WebForms, no public API, all state in `__VIEWSTATE`). Scraping requires
stored credentials and breaks on UI changes; it is fragile and ToS-risky. The
confirmation email the portal already sends is the durable, ToS-safe transport.

Users forward a Florida State Parks confirmation email to `intake@rsdp.com`.

- Unknown sender → server replies with 8-digit pairing code + QR image (registration)
- Known sender → email is parsed, data pushed to device

Three-level parsing pipeline (D5 in `docs/architecture-decisions.md`):

1. HTML/CSS class targeting (USeDirect template class names) -- fast, brittle
2. HTML → plain text → LLM structured extraction -- slower, robust
3. Raw email fallback -- stored and surfaced in app; no data loss

### Sync

- **Primary:** Web Push notification on email receipt → Service Worker → IndexedDB
- **Safety net:** 6 AM local device time daily (park opens at 8 AM)

### Display

PWA installed to home screen (no App Store). Opens directly to today's barcode
full-screen. One tap to reservation list. Offline-capable -- barcode renders
from local IndexedDB, no network call.

Barcode format: **Code 128**, confirmed by the v1 proof of concept.

## Key Constraints

- Offline barcode display at the gate (Service Worker + IndexedDB, no network call to render)
- No App Store -- PWA only, single codebase for iOS and Android
- No public API from Tyler Technologies -- email is the only safe data transport
- Code 128 format confirmed by proof of concept
- Single email address per account (v1); multi-email is a documented future TODO

## Data Model

Composite primary key `(confirmation_number, date)` handles multi-day reservations
on one confirmation number as separate display records.

See `docs/architecture-decisions.md` for full SQL schema.

## Build Platform

Likely **Lovable** (lovable.dev) generating React + Supabase. Lovable project:
`https://lovable.dev/projects/9a86f7bf-6301-4532-b405-4d245786f833`

## Development

```bash
make setup-hooks   # install pre-commit hooks (run once after clone)
```

Pre-commit hooks: markdown-table-formatter, markdownlint-fix, ligature/smartquote/dash fixers.

## Docs

- `docs/architecture-decisions.md` -- **authoritative**: decisions D1--D12, data model,
  address map, open questions, TODO list
- `docs/lovable-spec.md` -- v2 app spec (Phase 1/2, display layer)
- `docs/proof-of-concept.md` -- v1 Google Sheets + Claude skill (personal use, not v2 architecture)
- `docs/data-capture.md` -- prior scraping/bookmarklet options (**superseded and rejected**)
- `docs/art-gemini-notes.md` -- Playwright scraping notes (**rejected approach**)
- `CONTEXT.md` -- high-level project summary
