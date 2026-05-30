# Test Strategy -- Component-First, No Vibe Coding

Decisions and test cases from the May 2026 design/grill session.
Supersedes any vague "test it when it's built" assumptions.

---

## Architectural Clarifications (from grilling session)

### Types

| Type             | Description                                                                                                                                                                                                |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `EmailMessage`   | Can contain another `EmailMessage` (forwarded wrapper case)                                                                                                                                                |
| `DayReservation` | Full storage record. Mandatory: `confirmation_number`, `date`. All other fields optional enrichment. Carries `source: EmailMessage`.                                                                       |
| `DisplayRecord`  | Display-layer view of a reservation. Adapter pattern translates from `DayReservation`. Fields TBD but at minimum: `confirmation_number`, `date`. May include `person_name`, `person_email`, `pass_number`. |

### Parsing -- Constructor Model (not Level 1/2/3)

Each constructor either returns a fully-populated `DayReservation` (mandatory fields present) or raises. No partial results.

| Constructor                     | Input              | Extraction                  | Status                             |
|---------------------------------|--------------------|-----------------------------|------------------------------------|
| `DayReservationFromEmailHTML`   | email HTML body    | rule-based CSS selectors    | implement                          |
| `DayReservationFromEmailText`   | email → plain text | AI/LLM                      | implement                          |
| `DayReservationFromEmailRaw`    | email message      | none -- email IS the record | implement (fallback, not a parser) |
| `DayReservationFromPasteBuffer` | clipboard text     | TBD                         | stub                               |
| `DayReservationFromScreenshot`  | image              | AI vision                   | stub                               |

`DayReservationFromEmailRaw` is not a parser -- it's a constructor. One member: the `EmailMessage`. Valid domain object; `source` is the only populated field.

### Sync / Pub-Sub Model

- **Server DB (Supabase/PostgreSQL):** primary/authoritative
- **Phone IndexedDB:** read-only mirror (browser API, phone-only)
- **Sync = DB replication** via pub/sub push -- well-known pattern, not invented here
- **Pairing = auth + pub/sub subscription:** device registers → subscribes to account's reservation updates
- **All paired devices on an account** receive push on every server write
- **Safety-net push:** 7 AM local device time daily (park opens 8 AM)

### Pairing / Registration

- Unknown sender → server generates pairing code → replies to sender's email
- Pairing code: no expiry, multi-use (family plan -- partner texts code to spouse)
- If you have the code, you're in. No invitation request flow.
- One primary email address per account (v1). Multiple device tokens per account.

---

## Test Cases

### Email Parsing

1. `can detect whether email is wrapped` (EmailMessage → bool)
2. `can unwrap forwarded confirmation email` (EmailMessage → EmailMessage)
3. `parser returns multiple DayReservations for multi-day reservation` (same source, different dates)
4. `DayReservationFromEmailRaw returns valid DayReservation with source pointer`
5. `DayReservationFromPasteBuffer raises NotImplementedError`
6. `DayReservationFromScreenshot raises NotImplementedError`

### Registration / Pairing

1. `unknown sender triggers pairing code generation`
2. `valid pairing code links device push token to account`
3. `invalid pairing code is rejected`
4. `multiple device tokens stored per account`
5. `pairing code can be used by multiple devices` (family plan)

### Sync

1. `all devices on same account receive push when reservation arrives`

---

## Open Questions (to resolve before building)

- OQ-01: Does Mail.app forwarding preserve original HTML body intact, or wrap it?
  Capture raw MIME at intake address from a real forwarded email -- use as Level 1 fixture.
- OQ-02: Is confirmation number the only field in the Code 128 barcode?
- OQ-03: Exact FROM: address on FL parks confirmation emails after Mail.app forward?

---

## Next (Q10 -- pick up here)

Grilling continues: Service Worker sync mechanics, barcode display correctness,
DisplayRecord adapter, and pipeline runner tests.
