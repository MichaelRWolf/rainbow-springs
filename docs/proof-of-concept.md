# Proof of Concept

The working proof of concept lives in the
[`claude-tools`](https://github.com/MichaelRWolf/claude-tools) repo under
`skills/extract-rainbow-springs-reservation/`.

## What Exists

### Claude Skill -- `extract-rainbow-springs-reservation`

A Claude Code skill that accepts one or more screenshots of the Florida State
Parks "Payment Summary" page and:

1. Reads reservation fields visually (date, confirmation number, parking space, purchase date)
2. Formats them as CSV
3. Copies CSV to the clipboard
4. Opens the Google Sheet so the user can paste

Handles multiple reservations stacked in a single screenshot. Safe to run
twice -- deduplication happens downstream.

**Skill file**: `skills/extract-rainbow-springs-reservation/SKILL.md`

### Google Sheet -- Reservations + Barcode Tabs

Destination sheet:
`https://docs.google.com/spreadsheets/d/1wLHPw0uB7pe7hw_NJ9hPQWoQ2mt4uyv7_2YaQUt8-UU`

**Reservations tab** -- four columns: Date, Confirmation Number, Parking Space, Purchased

**Barcode tab** -- highlights today's reservation row and passes the confirmation
number to a URL-based barcode generator; the resulting image is displayed
full-size in the sheet.

### Google Apps Script -- `reservations.js`

Runs automatically when the sheet opens (`onOpen` trigger). It:

- Formats dates as `yyyy-mm-dd (ddd)` (e.g. `2026-05-01 (Fri)`)
- Left-aligns all columns
- Removes duplicate rows (keyed on confirmation number)
- Sorts descending by date

Installed by copy-pasting into **Extensions → Apps Script** in the sheet. No
`clasp`, no deploy step needed.

## Limitations of the Current Approach

- **Manual screenshot step** -- user must take a screenshot, switch to Claude, invoke the skill
- **Google Sheets barcode display** -- opening a spreadsheet at the gate is not a smooth experience
- **No offline support** -- barcode URL requires a network call to render

## TODO

- [ ] Document which barcode generator URL is used in the Barcode tab formula
- [ ] Document what data is encoded in the Code 128 barcode (confirmation number only, or more?)
- [ ] Decide whether to keep Google Sheets as the data store long-term or migrate
