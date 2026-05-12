# Visitor Assignment App — Copilot Instructions

## Overview

Build a **fully static, client-side web app** (HTML + CSS + Vanilla JavaScript) that manages the assignment of company visitors to tour guides during an event.
The app must run with **no backend, no database, and no server** — it is hosted on **GitHub Pages**.

All data processing happens entirely in the browser. Visitor names are never transmitted to any server. The Excel file is parsed locally. The assignment data is encoded into the QR code URL itself so that visitor phones can access the full lookup table without any network calls.

---

## Core Concept

1. The **admin** uploads a pre-filled Excel file containing visitor-to-guide assignments.
2. The app parses the Excel, encodes all assignment data into a compressed URL hash, and generates a **QR code** pointing to that URL.
3. **Visitors** scan the QR code on their phones, type their name, and receive their assigned group number.
4. **Guides** wait outside the room holding up their group number on their phone.
5. Visitors match their number to the correct guide and leave with them.
6. Some visitors are designated to **stay in the room** — the app informs them of this instead of showing a number.

---

## Architecture
- **Single HTML file** (`index.html`) with embedded CSS and JavaScript.
- **No frameworks**, no build tools, no npm. Pure HTML/CSS/JS only.
- **Two libraries loaded via CDN:**
  - [SheetJS (xlsx)](https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js) — for parsing the Excel file. Use version **0.20.3 from `cdn.sheetjs.com`** (the official source). Do **not** use the `cdnjs.cloudflare.com` or npm versions — they are outdated (0.18.5) and contain known security vulnerabilities (CVE-2023-30533, CVE-2024-22363).
  - [QRCode.js](https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js) — for generating the QR code.
- **Data encoding:** The parsed assignment data is serialized to JSON, then Base64-encoded, and appended to the URL as a hash fragment (e.g. `https://yoursite.github.io/app/#<base64>`). The hash is never sent to the server.
- **Two modes** are detected automatically based on the URL:
  - If the URL has **no hash** → Admin Mode.
  - If the URL has **a hash** → Visitor Mode.

---

## Excel File Structure

The admin must prepare the Excel file (`.xlsx`) with exactly the following columns in the first sheet. The first row is a header row and is ignored.

| Column A | Column B | Column C | Column D | Column E |
|---|---|---|---|---|
| `visitor_name` | `guide_name` | `guide_email` | `group_number` | `stays` |

### Column Definitions

| Column | Type | Description |
|---|---|---|
| `visitor_name` | String | Full name of the Schnuppernde exactly as they will type it. |
| `guide_name` | String | Full name of the assigned Lernende. |
| `guide_email` | String | Email address of the Lernende. Used for the optional mailto notification. Only needs to be filled in once per Lernende — duplicate rows for the same Lernende can leave this empty. |
| `group_number` | Integer | The number shown to the Schnuppernde and held up by the Lernende. All Schnuppernde assigned to the same Lernende share the same number. |
| `stays` | String | Write `yes` if the Schnuppernde stays in the room, leave empty or write `no` otherwise. |

### Example

| visitor_name | guide_name | guide_email | group_number | stays |
|---|---|---|---|---|
| Anna Müller | Thomas Becker | t.becker@firma.ch | 1 | |
| Jonas Weber | Thomas Becker | | 1 | |
| Sara Keller | Lisa Huber | l.huber@firma.ch | 2 | |
| Marco Rossi | Lisa Huber | | 2 | |
| Petra Schmid | | | | yes |
| David Meier | | | | yes |

---

## Admin Mode

Triggered when the URL has no hash (e.g. `https://yoursite.github.io/app/`).

### Layout

```
┌────────────────────────────────────────┐
│           [App Logo / Title]           │
│        Visitor Assignment Tool         │
├────────────────────────────────────────┤
│  Step 1: Upload Excel File             │
│  [ Drop file here or click to upload ] │
├────────────────────────────────────────┤
│  Step 2: Review Assignments (table)    │
│  (visible only after successful parse) │
├────────────────────────────────────────┤
│  Step 3: Generate QR Code              │
│  ☐ Also notify Lernende via email      │
│  [ Generate QR Code ] (button)         │
│  (visible only after successful parse) │
│                                        │
│  ┌──────────────┐                      │
│  │   QR Code    │                      │
│  └──────────────┘                      │
│  [ Copy Link ] [ Download QR ]         │
│  [ Notify Lernende ] (if checked)      │
├────────────────────────────────────────┤
│  Schnuppernde Staying in Room:         │
│  (list of names — shown after parse)   │
└────────────────────────────────────────┘
```

### Behaviour

- File upload supports drag-and-drop and click-to-browse.
- After the file is uploaded, validate that columns A–E exist. Show a clear error message if the format is wrong.
- Parse all rows into an array of assignment objects.
- Display a summary table showing all assignments grouped by Lernende.
- Highlight rows where `stays = yes` separately.
- A checkbox labelled **"Also notify Lernende via email"** is shown directly above the "Generate QR Code" button. It is unchecked by default.
- On "Generate QR Code":
  - Serialize the assignments array to JSON.
  - Base64-encode it: `btoa(unescape(encodeURIComponent(JSON.stringify(data))))`.
  - Construct the full URL: `window.location.origin + window.location.pathname + '#' + encodedData`.
  - Render the QR code using QRCode.js targeting a `<div>`.
  - Show a "Copy Link" button and a "Download QR as PNG" button.
  - If the email checkbox is checked, show a **"Notify Lernende"** button below the QR code.
- On "Notify Lernende" (only visible if checkbox was checked):
  - Group the parsed data by Lernende (by `guide_name` + `guide_email`).
  - For each Lernende with a valid `guide_email`, construct a `mailto:` link with:
    - `to`: `guide_email`
    - `subject`: `Your group number for today's event`
    - `body`: A short message stating their group number and a brief instruction (e.g. "Please wait outside the room and hold up the number [X] on your phone so Schnuppernde can find you.")
  - Open each `mailto:` link sequentially with a small delay (300ms) between each to avoid the browser blocking multiple popups.
  - Show a confirmation list of which Lernende were notified and which had no email address in the Excel.
- Show the "Schnuppernde Staying in Room" section as a separate highlighted list below the QR code.

---

## Visitor Mode

Triggered when the URL contains a hash (e.g. `https://yoursite.github.io/app/#eyJ2...`).

### Layout

```
┌────────────────────────────────────────┐
│           [App Logo / Title]           │
├────────────────────────────────────────┤
│  Please enter your full name:          │
│  [ ________________________ ]          │
│  [ Find my group ]                     │
├────────────────────────────────────────┤
│  Result (shown after lookup):          │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │  Your group number is:           │  │
│  │           [ 3 ]                  │  │
│  │  Find the guide holding          │  │
│  │  the number 3 outside.           │  │
│  └──────────────────────────────────┘  │
│                                        │
│  — OR —                                │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │  Please remain seated.           │  │
│  │  You are staying in the room.    │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

### Behaviour

- On page load, decode the hash: `JSON.parse(decodeURIComponent(escape(atob(window.location.hash.slice(1)))))`.
- If decoding fails, show a friendly error: "This QR code is invalid or has expired."
- The visitor types their name and presses Enter or clicks the button.
- **Name matching must be:**
  - Case-insensitive.
  - Trim leading/trailing whitespace.
  - Fuzzy: tolerate 1–2 character differences using a simple Levenshtein distance function (to handle typos).
  - If multiple fuzzy matches exist, pick the closest one. If tied, show both and ask the visitor to clarify.
- On match:
  - If `stays = yes` → show the "remain seated" result card.
  - Otherwise → show the group number result card prominently.
- On no match → show: "Name not found. Please check your spelling and try again."
- No personal data is logged, stored in localStorage, or sent anywhere.

---

## Design Specifications

### General

- **Mobile-first.** The visitor view must be fully usable on a small phone screen (min width 320px).
- Clean, professional, minimal design. No decorative clutter.
- Font: `Inter` or `system-ui` fallback.
- Background: white (`#ffffff`).
- Primary colour: deep blue `#1a3c6e`.
- Accent colour: `#2e7df7`.
- Success/stay colour: soft green `#22c55e`.
- Error colour: `#ef4444`.
- All text must have sufficient contrast (WCAG AA minimum).

### Admin Mode

- Steps are visually separated with a light grey card/section (`#f8f9fa` background, `1px solid #e2e8f0` border, `8px` border-radius).
- Steps 2 and 3 are hidden (`display: none`) until the Excel is successfully parsed.
- The assignment summary table uses alternating row colours for readability.
- The QR code is centred and rendered at `256×256px`.
- The "Visitors Staying" section uses a distinct green-tinted card.

### Visitor Mode

- Full-height centered layout. Everything is vertically centred on screen.
- The name input is large and touch-friendly (`font-size: 1.2rem`, `padding: 12px`).
- The result card is large and bold — the group number must be displayed in a very large font (`font-size: 5rem`, `font-weight: 700`).
- The "stay" result card uses a green background tint.
- Animate the result card appearing with a simple fade-in.

---

## File & Folder Structure

```
/
├── index.html       ← entire app (HTML + CSS + JS in one file)
└── README.md        ← basic usage instructions
```

---

## Error Handling

| Situation | Message to show |
|---|---|
| Wrong Excel format | "The uploaded file does not match the expected format. Please check that columns A–E are: visitor_name, guide_name, guide_email, group_number, stays." |
| Name not found (exact) | "Name not found. Please check your spelling and try again." |
| Name not found (fuzzy failed too) | Same as above. |
| Invalid/corrupt QR hash | "This QR code is invalid or could not be read. Please ask the event coordinator for a new one." |
| Excel file is empty | "The uploaded file contains no visitor data." |
| Lernende has no email address | Show a warning in the notification confirmation list: "[Name] — no email address provided, skipped." |

---

## Security & Privacy Notes

- No data is ever sent to a server.
- The Excel file is processed entirely in the browser using the FileReader API.
- The URL hash (`#`) fragment is never included in HTTP requests by the browser — GitHub Pages never sees the encoded data.
- Do not use `localStorage` or `sessionStorage`.
- The admin page has no authentication. Security is by obscurity — only the admin has access to the base URL without a hash. This is acceptable for this low-risk internal use case.

---

## Out of Scope

- User authentication / login.
- Real-time updates between devices.
- Backend or database of any kind.
- Support for file formats other than `.xlsx`.
- Multi-language support.
