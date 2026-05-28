# Rainbow Springs App -- Project Context

## Goal

Offline-first local/mobile app that scrapes the Florida State Parks reservation portal,
stores data in SQLite, and displays Code 128 barcodes at the park gate with zero cell service.

**Why:** Park gates scan a Code 128 barcode from the confirmation; cell service is unreliable
at the gate; app must work fully offline once data is loaded.

## Scope (current)

- Step 1: Scrape existing reservations from portal → store in SQLite
- Step 2: Display layer -- date-aware sorting, Code 128 barcode rendering, offline view

**Skipped for now:** Automated reservation/checkout ("clicky-clicky game")

---

## Step 1: Ingestion

### Flow

1. User enters Florida State Parks portal credentials
2. App opens headless browser session (Playwright) or direct HTTPS → logs into reserve.floridastateparks.org
3. Navigates to "Current Reservations" / "History" tab
4. Scrapes HTML for reservation records

### What the scraper extracts (per reservation)

- **Park name** -- e.g., `Rainbow Springs State Park`
- **Reservation date** -- e.g., `2026-06-15`
- **Confirmation number** -- the booking ID encoded in the Code 128 barcode, e.g., `22569340`

---

## SQLite Schema: `local_reservations`

| Column              | Type         | Purpose                            | Example                    |
|---------------------|--------------|------------------------------------|----------------------------|
| id                  | Integer (PK) | Auto-increment unique ID           | 1                          |
| park_name           | Text         | State park location name           | Rainbow Springs State Park |
| reservation_date    | Text/Date    | Date pass is valid                 | 2026-06-15                 |
| confirmation_number | Text         | Booking ID; encoded in barcode     | 22569340                   |
| is_used             | Boolean      | Visual toggle: already used today? | 0 (False)                  |

---

## Step 2: Display (next)

- Date-aware sorting of upcoming reservations
- Generate Code 128 barcode lines rendered on screen
- Offline view management (no network required after sync)
