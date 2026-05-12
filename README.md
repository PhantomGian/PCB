# Visitor Assignment Tool

A fully static, client-side web app that manages the assignment of company visitors to tour guides during an event. Runs entirely in the browser — no backend, no database, no server required. Hosted on GitHub Pages.

## How It Works

1. **Admin** uploads a pre-filled Excel file (`.xlsx`) containing visitor-to-guide assignments.
2. The app parses the file locally, encodes all data into a QR code URL.
3. **Visitors** scan the QR code, enter their name, and see their assigned group number.
4. **Guides** hold up their group number on their phone outside the room.
5. Visitors match their number to the correct guide and leave with them.

## Excel File Format

Prepare a `.xlsx` file with the following columns in the first sheet (first row is a header):

| Column A | Column B | Column C | Column D |
|---|---|---|---|
| `visitor_name` | `guide_name` | `group_number` | `stays` |

- **visitor_name** — Full name of the visitor (as they will type it).
- **guide_name** — Name of the assigned guide.
- **group_number** — The group number shown to the visitor and held by the guide.
- **stays** — Write `yes` if the visitor stays in the room; leave empty otherwise.

### Example

| visitor_name | guide_name | group_number | stays |
|---|---|---|---|
| Anna Müller | Thomas Becker | 1 | |
| Jonas Weber | Thomas Becker | 1 | |
| Sara Keller | Lisa Huber | 2 | |
| Petra Schmid | | | yes |

## Usage

### Admin

1. Open `index.html` in a browser (or the GitHub Pages URL).
2. Upload your `.xlsx` file by dragging it onto the drop zone or clicking to browse.
3. Review the parsed assignments in the table.
4. Click **Generate QR Code**.
5. Use **Copy Link** or **Download QR** to share the QR code with visitors.

### Visitors

1. Scan the QR code with your phone camera.
2. Enter your full name and tap **Find my group**.
3. Your assigned group number (or instruction to stay) will appear on screen.

## Privacy

- No data is sent to any server. All processing happens in the browser.
- The Excel file is parsed locally using the FileReader API.
- The URL hash fragment is never transmitted to the server.
- No cookies, localStorage, or sessionStorage are used.

## Deployment

Simply host `index.html` on GitHub Pages or any static file server.
