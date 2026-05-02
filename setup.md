# The Hub — Setup Guide
## Private Google Sheet + Restricted API Key

No backend. No public CSV. Your sheet stays completely private.
The trick: an API key locked to your server's IP, so only your site can use it.

---

## How it works

```
Browser → checks your server IP matches office WiFi IP → 
  → calls Google Sheets API v4 with your API key →
    → Google checks: is this request from the allowed IP? →
      → yes: returns sheet data  /  no: rejects the key
```

Your sheet is **never published**. It's shared only with a service account email.
Even if someone finds your API key in the HTML, they can't use it from their laptop —
Google will reject it because their IP doesn't match.

---

## Step 1 — Google Cloud Project + Sheets API

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g. "Hub Intranet")
3. Go to **APIs & Services → Library**
4. Search **Google Sheets API** → Enable it

---

## Step 2 — Create a restricted API Key

1. Go to **APIs & Services → Credentials**
2. Click **+ Create Credentials → API Key**
3. Click **Edit API Key** on the key that appears
4. Under **Application restrictions** → select **IP addresses**
5. Add your server's public IP (the IP of the machine hosting index.html)
   - SSH into your server and run: `curl ifconfig.me`
6. Under **API restrictions** → select **Restrict key** → pick **Google Sheets API**
7. Save → copy the API key

> ⚠️ The IP restriction applies to the machine making the API call.
> Since the browser (visitor) is making the call, you need your **office WiFi's public IP** here,
> not your server's IP. Add both if needed:
> - Your server's IP (if you ever call from server-side)  
> - Your office WiFi's public IP (for browser-side calls from members)

**Add your office WiFi IP to the allowed list:**
From inside the office WiFi, visit [whatismyip.com](https://whatismyip.com) and add that IP.

---

## Step 3 — Create a Service Account (to share the sheet privately)

1. Go to **APIs & Services → Credentials → + Create Credentials → Service Account**
2. Give it any name (e.g. "hub-reader")
3. Skip optional steps → Done
4. Click the service account → **Keys tab → Add Key → JSON** → download it
5. Note the **service account email** (looks like `hub-reader@your-project.iam.gserviceaccount.com`)

> You won't need the JSON key file for this setup (no backend).
> The service account email is just used to share the sheet.

---

## Step 4 — Set up your Google Sheet

Create a new Google Sheet. Add **3 tabs** with these exact names (or update CONFIG in index.html):

### Tab: Employees
| Name | DOB | Gender | Joined Date |
|------|-----|--------|-------------|
| Priya Sharma | 15/04/1996 | Female | 01/06/2023 |
| Arjun Mehta | 22/05/1993 | Male | 15/03/2022 |

- DOB format: `DD/MM/YYYY`

### Tab: Holidays
| Date | Holiday Name | Type |
|------|-------------|------|
| 01/05/2025 | Labour Day | Public |
| 15/08/2025 | Independence Day | Public |
| 02/10/2025 | Gandhi Jayanti | Optional |

- Date format: `DD/MM/YYYY`
- Type: `Public`, `Optional`, or `Restricted`

### Tab: Announcements
| Title | Body | Emoji | Date |
|-------|------|-------|------|
| Rooftop mixer Friday! | Join us at 6 PM 🍻 | 🎉 | 2025-04-28 |
| Printer down | 2nd floor being serviced | 🖨️ | 2025-04-25 |

- Date format: `YYYY-MM-DD`

### Share the sheet
Click **Share** → paste the service account email → set to **Viewer** → Share.

The sheet is now private. Only that service account (and your API key) can read it.

---

## Step 5 — Get the Spreadsheet ID

From your sheet URL:
```
https://docs.google.com/spreadsheets/d/  1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms  /edit
                                          ↑ this is your SPREADSHEET_ID
```

---

## Step 6 — Edit index.html

Open `index.html` and fill in the CONFIG block:

```js
const CONFIG = {
  OFFICE_IP:      "103.45.67.89",        // your office WiFi public IP
  SPREADSHEET_ID: "1BxiMVs0XRA5n...",   // from sheet URL
  API_KEY:        "AIzaSyD...",          // restricted API key from Step 2

  SHEETS: {
    employees:     "Employees",           // must match your tab names exactly
    holidays:      "Holidays",
    announcements: "Announcements",
  },

  SPACE_NAME: "The Hub",
};
```

---

## Step 7 — Deploy

Upload `index.html` to your server. That's the only file.

```bash
scp index.html user@yourserver:/var/www/html/intranet/index.html
```

Visit it from inside your office WiFi — you should see the full site.
Visit from outside — you'll see the lock screen.

---

## Security summary

| Threat | Protection |
|--------|-----------|
| Random person finds the URL | IP check — lock screen shown |
| Someone finds the API key in HTML | Key restricted to office WiFi IP — useless elsewhere |
| Sheet accidentally exposed | Sheet is private, shared only with service account email |
| Someone on office WiFi snoops | They're already a co-working member — acceptable |

---

## Updating content

Everything lives in the Google Sheet — just edit rows.
The site fetches fresh data on every page load. No redeploy needed.

| Action | Where |
|--------|-------|
| Add/edit a birthday | Employees tab |
| Add a holiday | Holidays tab |
| Post announcement | Announcements tab (newest date = top) |
| Change office IP | `OFFICE_IP` in index.html CONFIG |
| Rename the space | `SPACE_NAME` in index.html CONFIG |
