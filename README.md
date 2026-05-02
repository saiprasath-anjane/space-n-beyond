# 🏢 Space N Beyond — Co-working Space Intranet

A lightweight, single-file intranet website for co-working spaces.  
Pulls live data from a private Google Sheet. No backend. No database. Just one HTML file.

---

## ✨ Features

- 🎂 **Birthdays this month** — shows who's celebrating, with a "Today!" badge on the actual day
- 📅 **Holiday calendar** — full year view, past holidays auto-fade
- 📢 **Announcements board** — newest first, shows how many days ago each was posted
- 🔒 **IP-based access lock** — only visible on your office WiFi, everyone else sees a lock screen

---

## 🔐 Security Model

Two layers of protection:

```
Visitor arrives
      │
      ▼
Layer 1: IP Check (JavaScript)
  Wrong IP → Lock screen shown, nothing loads
  Right IP ↓
      │
      ▼
Layer 2: API Key (Google-side)
  Wrong referrer → Google rejects the request
  Right referrer ↓
      │
      ▼
  Sheet data loads ✅
```

| Threat | Protection |
|--------|-----------|
| Random person finds the URL | IP lock — they see a lock screen |
| Someone copies the API key from HTML | Key restricted to your domain — useless elsewhere |
| Someone guesses the Sheet ID | Sheet is "anyone with link" Viewer only — not indexed, not searchable |

---

## 🗂️ Google Sheet Structure

Create one Google Sheet with **3 tabs**:

### Tab 1: `Employees`
| Name | DOB | Gender | Joined Date |
|------|-----|--------|-------------|
| Priya Sharma | 15/04/1996 | Female | 01/06/2023 |
| Arjun Mehta | 22/05/1993 | Male | 15/03/2022 |

- `DOB` format: `DD/MM/YYYY`
- `Gender` and `Joined Date` are optional

### Tab 2: `Holidays`
| Date | Holiday Name | Type |
|------|-------------|------|
| 01/05/2025 | Labour Day | Public |
| 15/08/2025 | Independence Day | Public |
| 02/10/2025 | Gandhi Jayanti | Optional |

- `Date` format: `DD/MM/YYYY`
- `Type` options: `Public`, `Optional`, `Restricted`

### Tab 3: `Announcements`
| Title | Body | Emoji | Date |
|-------|------|-------|------|
| Rooftop mixer Friday! | Join us at 6 PM 🍻 | 🎉 | 2025-04-28 |
| Printer maintenance | 2nd floor being serviced | 🖨️ | 2025-04-25 |

- `Date` format: `YYYY-MM-DD`
- `Emoji`: any single emoji, shows as the announcement icon

---

## ⚙️ Setup

### 1. Google Cloud — Enable Sheets API
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a project (e.g. "Hub Intranet")
3. **APIs & Services → Library → Google Sheets API → Enable**

### 2. Create an API Key
1. **APIs & Services → Credentials → + Create Credentials → API Key**
2. Click **Edit** on the new key
3. Under **API restrictions** → select **Google Sheets API**
4. Under **Application restrictions** → select **Websites**
5. Add your site URL: `https://yourdomain.com/*`
6. Save and **copy the key**

### 3. Share your Google Sheet
1. Open your Google Sheet
2. Click **Share**
3. Change access to **"Anyone with the link" → Viewer**
4. Do NOT publish — just share with link

### 4. Get your Spreadsheet ID
From your sheet's URL:
```
https://docs.google.com/spreadsheets/d/  1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms  /edit
                                          ↑ this part is your SPREADSHEET_ID
```

### 5. Get your office WiFi's public IP
Connect to the office WiFi and visit [whatismyip.com](https://whatismyip.com).  
Copy the IP shown (e.g. `103.45.67.89`).

### 6. Edit `index.html`
Find the `CONFIG` block near the top of the `<script>` tag:

```js
const CONFIG = {
  OFFICE_IP:      "103.45.67.89",              // your office WiFi public IP
  SPREADSHEET_ID: "1BxiMVs0XRA5nFMd...",      // from your sheet URL
  API_KEY:        "AIzaSyD...",                // restricted API key

  SHEETS: {
    employees:     "Employees",                // must match tab names exactly
    holidays:      "Holidays",
    announcements: "Announcements",
  },

  SPACE_NAME: "Space N Beyond",                       // your space's name
};
```

### 7. Deploy
Upload `index.html` to your server:

```bash
scp index.html user@yourserver:/var/www/html/intranet/index.html
```

Visit from office WiFi → full site loads.  
Visit from anywhere else → lock screen.

---

## 🔄 Updating Content

Everything is managed from the Google Sheet — no redeployment needed.

| What to do | Where |
|------------|-------|
| Add a new member's birthday | Add row in `Employees` tab |
| Add / edit a holiday | Add row in `Holidays` tab |
| Post an announcement | Add row in `Announcements` tab |
| Remove an announcement | Delete that row |
| Change office IP (if WiFi changes) | Edit `OFFICE_IP` in `index.html` |
| Rename the space | Edit `SPACE_NAME` in `index.html` |

---

## 📁 File Structure

```
/
└── index.html      ← the entire app, one file
└── README.md       ← this file
```

---

## 🛠️ Tech Stack

| Layer | What |
|-------|------|
| Frontend | HTML + Tailwind CSS (CDN) + Vanilla JS |
| Fonts | DM Serif Display + DM Sans (Google Fonts) |
| Data | Google Sheets API v4 |
| Auth | API Key (domain-restricted) + IP lock |
| Hosting | Any server (Apache, Nginx, etc.) |

No npm. No build step. No framework. Open the file and it works.

---

## 🚀 Optional Upgrades

- **Work anniversary tracker** — `Joined Date` is already in the Employees tab, easy to add a section
- **Birthday email alerts** — use Google Apps Script to auto-email the space on someone's birthday morning
- **QR code at entrance** — print a QR linking to the intranet; members scan it while on WiFi
- **Stronger auth** — add HTTP Basic Auth in Nginx/Apache for a second password layer
- **Dark mode** — CSS variables are already set up, a toggle is straightforward to add

---

## ⚠️ Known Limitations

- IP check is client-side — a tech-savvy person could bypass the lock screen in DevTools. The API key restriction on Google's side is the real security backstop.
- If your office WiFi IP changes (ISP reassigns it), update `OFFICE_IP` in `index.html` and redeploy.
- Google Sheets API has a free quota of **500 requests/100 seconds** — more than enough for a co-working space.

---

*Built for Space N Beyond · Single file · No backend · Just edit the sheet*
