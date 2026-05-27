# App Spec — Rainbow Springs Reservation Barcode App

## What This App Does

A small web app for a group of ~10 friends who each have a Florida State Parks
annual pass and make their own day-use reservations at Rainbow Springs State
Park. The app solves one problem: at the entry gate, rangers need to scan a
barcode from your reservation. Finding that barcode in a confirmation email is
slow and awkward. This app shows the right barcode, full-screen, automatically.

## Users

- ~10 people, each with their own account on the Florida State Parks reservation
  site (`reserve.floridastateparks.org`)
- Each user has their own reservations, independent of the others
- Personal use only — no public sign-ups

## Phase 1 — Get Reservations into the App

Each user logs into the app and provides their Florida State Parks credentials.
The app logs into `https://reserve.floridastateparks.org/Web/Customers/CustomerReservations.aspx`
on their behalf, scrapes their upcoming reservations, and saves them to the
app's database.

Each reservation has:
- **Date** — the day of the visit (e.g. `05/01/2026`)
- **Confirmation Number** — the booking ID (e.g. `22569340`)
- **Parking Space** — assigned space (e.g. `P 057`)
- **Purchased** — date the reservation was made

The user can trigger a sync manually (a "Refresh Reservations" button). No
reservations need to be entered by hand.

## Phase 2 — Show the Barcode at the Gate

When the user opens the app on their phone at the entry gate:

- If today has a reservation, show its barcode **immediately, full-screen**
- The barcode encodes the confirmation number
- No tapping, no searching, no scrolling
- Must work **offline** — the barcode cannot depend on a network call to render

A list of upcoming reservations should also be accessible one tap away.

## What the App Should Feel Like

- Opens on a phone and the barcode is just *there*
- Clean, minimal — a barcode and a date, nothing else on the main screen
- Fast enough to use at a gate with a ranger waiting

## What the App Does NOT Do

- Does not make or cancel reservations
- Does not replace the official Florida State Parks app
- Does not need to work for strangers — just this group of friends
