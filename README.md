# Rainbow Springs

Tooling to make the Rainbow Springs State Park entry experience smoother.

## Problem

Florida State Parks now requires a reservation for day-use vehicle entry. The
confirmation comes by email. At the gate, rangers scan a barcode from that
email -- but finding the email, opening it, and presenting a tiny barcode is
awkward for both visitor and ranger.

## Goal

- Capture reservation data from the parks reservation site into a local data store
- Display a **full-screen barcode** on the phone at the gate
- **Auto-display today's reservation** -- nothing to look up, nothing to open

## Docs

- [UX Vision](docs/ux-vision.md) -- gate experience goals and non-goals
- [Proof of Concept](docs/proof-of-concept.md) -- what already exists
- [Data Capture](docs/data-capture.md) -- options for getting reservations into the data store
