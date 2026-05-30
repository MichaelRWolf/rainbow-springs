# App Spec -- Rainbow Springs Reservation Barcode App (v2)

> **Note:** This is the v2 spec for the shared PWA built on email ingestion.
> The v1 proof of concept (Google Sheets + Claude skill) is documented in
> [proof-of-concept.md](proof-of-concept.md) and is kept as a working personal tool. These are
> separate things with separate audiences.

## What This App Does

A PWA for Michael and a small group of friends who each have a Florida State
Parks annual pass and make their own day-use reservations at Rainbow Springs
State Park. The app solves one problem: at the entry gate, rangers need to scan a
barcode from your reservation. Finding that barcode in a confirmation email is
slow and awkward. This app shows the right barcode, full-screen, automatically.

## Users

- Michael and a few friends, each with their own account on the Florida State
  Parks reservation site (`reserve.floridastateparks.org`)
- Each user has their own reservations, independent of the others
- Personal use only -- no public sign-ups
- Annual pass holders only (v1); one-off visitors are out of scope

## Phase 1 -- Get Reservations into the App

Each user forwards a Florida State Parks confirmation email to `intake@rsdp.com`.

- First forward from a new address: server replies with a pairing code to
  register the app
- User installs the PWA and enters the pairing code (or scans QR)
- Subsequent forwards: parsed automatically, pushed to device

Each reservation has:

- **Date** -- the day of the visit (e.g. `05/01/2026`)
- **Confirmation Number** -- the booking ID (e.g. `22569340`)
- **Parking Space** -- assigned space (e.g. `P 057`)
- **Purchased** -- date the reservation was made

No credentials required. No scraping. Manual forward is the minimum; a
Mail.app rule automates it for power users.

## Phase 2 -- Show the Barcode at the Gate

When the user opens the app on their phone at the entry gate:

- If today has a reservation, show its barcode **immediately, full-screen**
- The barcode encodes the confirmation number
- No tapping, no searching, no scrolling
- Must work **offline** -- the barcode cannot depend on a network call to render

A list of upcoming reservations should also be accessible one tap away.

## What the App Should Feel Like

- Opens on a phone and the barcode is just *there*
- Clean, minimal -- a barcode and a date, nothing else on the main screen
- Fast enough to use at a gate with a ranger waiting

## What the App Does NOT Do

- Does not make or cancel reservations
- Does not replace the official Florida State Parks app
- Does not need to work for strangers -- just this group of friends
