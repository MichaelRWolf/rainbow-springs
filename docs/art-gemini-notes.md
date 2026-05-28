# Notes from Art's Conversation with Gemini

## Step 1: Extracting Existing Reservations

Since we are skipping automated browser checkout, we need to pull data from where it currently lands. There are two primary avenues to harvest this data from the "clicky-clicky game" outputs: **Web Scraping** (parsing the account history page) or **Email Parsing** (extracting data from the confirmation emails).

For a local app, **Web Scraping** the user's portal profile is the most direct way to get a clean batch of upcoming dates.

### The Ingestion Flow

1. **User Login:** The user opens your app and enters their Florida State Parks portal credentials.
2. **Session Authentication:** The app opens an invisible background window (a headless browser session using a lightweight tool like Playwright or a direct HTTPS request) to log into reserve.floridastateparks.org.
3. **Targeting the History Page:** The script navigates directly to the user's "Current Reservations" or "History" tab.
4. **Data Extraction (The Scrape):** The script searches the HTML code for specific data markers (tags) that contain the reservation details.

### What the Scraper Looks For

The scraper parses the raw webpage HTML to pull out a clean text string for each reservation. It isolates three critical pieces of data per day:

- **The Park Name:** (e.g., "Rainbow Springs State Park")
- **The Target Date:** (e.g., "Valid: June 15, 2026")
- **The Alphanumeric Confirmation String:** This text string is the exact data embedded inside that Code 128 barcode. The scanner at the gate reads this text when it scans the lines.

## Storing the Data Locally (SQLite)

Once the scraper grabs that data, it instantly writes it to a lightweight, local database built right into the app (typically **SQLite** for mobile devices). This is what ensures the app requires **zero network access** once the data is pulled.

### `local_reservations` Table

| Column Name          | Data Type            | Purpose                                               | Example Data               |
|----------------------|----------------------|-------------------------------------------------------|----------------------------|
| id                   | Integer (Primary Key) | Auto-incrementing unique identifier.                 | 1                          |
| park_name            | Text                 | Name of the state park location.                      | Rainbow Springs State Park |
| reservation_date     | Text/Date            | The specific day the pass is valid.                   | 2026-06-15                 |
| barcode_data_string  | Text                 | The raw alphanumeric text used to draw the barcode.   | FLSP-987654321-X           |
| is_used              | Boolean              | A toggle to visually mark if they already used it today. | 0 (False)               |

## Next Steps

Move to **Step 2: Display** — date-aware sorting, generating the physical Code 128 barcode lines on the screen, and managing the offline view.
