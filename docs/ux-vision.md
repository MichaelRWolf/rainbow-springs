# UX Vision — Gate Entry Experience

## The Moment That Matters

Michael pulls up to the Rainbow Springs entry gate. The ranger needs to scan a
barcode. The experience should be:

1. Phone is already showing the barcode — or one tap away
2. Barcode fills the screen — easy for the ranger to scan
3. No hunting for an email, no pinching to zoom, no awkward wait

## Goals

- **Zero-friction display**: today's reservation barcode is visible without searching
- **Full-screen barcode**: large enough for a handheld scanner at arm's length
- **Auto-date awareness**: the right reservation surfaces based on today's date
- **Works offline**: barcode display must not require a network call at the gate

## Non-Goals

- Does not replace the official Florida State Parks app or website
- Does not handle check-out or cancellations
- Does not need to work for anyone other than Michael

## Open Questions

- [ ] What triggers the barcode display — a home screen widget, a dedicated app, a
      web page bookmark, or a Shortcut?
- [ ] What happens when there is no reservation for today?
- [ ] Should upcoming reservations (next 7 days) also be visible?
- [ ] iOS only, or also usable on Android / desktop?
