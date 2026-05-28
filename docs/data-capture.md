# Data Capture -- Getting Reservations into the Data Store

## The Auth Problem

The reservation page requires login:

```text
GET https://reserve.floridastateparks.org/Web/Customers/CustomerReservations.aspx
→ 403 Forbidden (unauthenticated)
```

Any scraping approach must operate inside an authenticated browser session --
there is no public API.

## Options

| Approach                                | Trigger                   | Friction                                 | Complexity |
|-----------------------------------------|---------------------------|------------------------------------------|------------|
| **Screenshot → Claude skill** (current) | Manual                    | Medium -- switch to Claude, invoke skill | Low        |
| **Bookmarklet**                         | One click while logged in | Low                                      | Medium     |
| **Playwright / browser automation**     | Scheduled or on-demand    | Near-zero after setup                    | High       |
| **iOS Shortcut**                        | One tap in Safari         | Low on phone                             | Medium     |

## Recommendation: Bookmarklet

A bookmarklet runs JavaScript in the browser tab while the user is already
logged in and viewing the reservations page. One click:

1. Scrapes reservation rows from the DOM
2. Formats as JSON or CSV
3. POSTs to a local server, writes to a file, or copies to clipboard

**Why bookmarklet over Playwright**: no credential storage, no scheduled
process to maintain, works on both desktop and mobile Safari.

**Why bookmarklet over iOS Shortcut**: also works on desktop, and the DOM
parsing logic lives in one place rather than being split across platforms.

## Open Questions

- [ ] Where does the bookmarklet send the data? Options:
  - POST to a local server (requires Mac to be running)
  - Copy to clipboard → paste into a file or sheet
  - Write directly to a cloud store (Gist, Sheets API, iCloud)
- [ ] What platform does Michael use when making reservations -- phone or Mac?
      (Answer changes whether Safari mobile bookmarklet or desktop bookmarklet
      is the primary path)
- [ ] What is the HTML structure of the reservations page? (Need to inspect DOM
      to write the scraper -- requires a logged-in session)
- [ ] Should capture run automatically on page load, or require a manual trigger?

## Data Shape

Fields available from the Payment Summary page (confirmed via current skill):

| Field               | Example      |
|---------------------|--------------|
| Date                | `05/01/2026` |
| Confirmation Number | `22569340`   |
| Parking Space       | `P 057`      |
| Purchased           | `04/29/2026` |

## TODO

- [ ] Inspect DOM of the reservations page while logged in to identify CSS
      selectors for scraping
- [ ] Prototype bookmarklet that copies reservation data to clipboard
- [ ] Decide on data store (Google Sheets via API, local JSON file, iCloud,
      other)
- [ ] Define sync strategy: overwrite all, append + deduplicate, or diff-based
