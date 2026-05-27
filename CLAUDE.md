# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A personal tool to make entry at Rainbow Springs State Park smoother. Florida
State Parks requires a reservation; the confirmation email has a barcode rangers
scan at the gate. The goal is to display that barcode full-screen on a phone —
auto-selected for today's date — with no email-hunting or pinching to zoom.

## Current State

No application code yet. The repo contains design docs and a working proof of
concept that lives in a separate repo.

**Proof of concept** — `MichaelRWolf/claude-tools`, skill `extract-rainbow-springs-reservation`:
- Claude skill: takes Payment Summary screenshots → extracts CSV → pastes into Google Sheet
- Google Sheet: `1wLHPw0uB7pe7hw_NJ9hPQWoQ2mt4uyv7_2YaQUt8-UU`
  - Reservations tab: Date, Confirmation Number, Parking Space, Purchased
  - Barcode tab: highlights today's row, generates barcode from confirmation number via URL
- `reservations.js`: Google Apps Script (copy-paste into Extensions → Apps Script), runs `onOpen` to deduplicate, sort, and format dates

## Architecture Intent

The system has three layers being designed:

1. **Data capture** — extract reservation data from the authenticated parks site into a data store
2. **Data store** — currently Google Sheets; long-term destination TBD
3. **Barcode display** — mobile-first, full-screen, offline-capable, auto-filters to today

The **preferred data-capture approach is a bookmarklet** — JavaScript that runs
in the browser while the user is logged in. The reservations page
(`reserve.floridastateparks.org`) returns 403 unauthenticated; there is no API.

The **preferred app-builder platform is Glide** (glideapps.com) because it
connects natively to Google Sheets and renders image URLs full-screen on mobile.
Lovable (lovable.dev) is the fallback if Glide's image sizing proves too
constrained.

## Key Design Constraints

- Barcode display **must work offline** at the gate — no network call to render
- Only needs to work for one user (Michael)
- Does not handle cancellations or check-out

## Docs

- `docs/ux-vision.md` — gate experience goals, non-goals, open UX questions
- `docs/proof-of-concept.md` — detailed description of existing skill + Sheet + Apps Script
- `docs/data-capture.md` — scraping options, bookmarklet recommendation, data shape, open questions
