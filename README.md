# Combuses Boarding Log — Setup Guide

## Overview

This is a lightweight mobile web app that allows boarding agents to log which passengers board which bus. It works by scanning QR codes on tickets (or manual entry) and sends each record to a Google Sheet for reporting and revenue allocation.

---

## Quick Start (5 minutes)

### 1. Deploy the App

Host the `index.html` file anywhere:
- **Simplest**: Upload to GitHub Pages, Netlify, or Vercel (free)
- **Quick test**: Open the file directly on a phone browser
- **Pro**: Host on your own domain

The app works entirely in the browser. No server needed (except for Google Sheets sync).

### 2. Set Up Google Sheets

#### A. Create a new Google Sheet

Create a sheet with these column headers in Row 1:

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| Timestamp | Date | Ticket | Bus ID | Route | Departure | Notes | Record ID |

#### B. Add the Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**
2. Delete the default code and paste the following:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);

    // Handle test pings
    if (data.test) {
      return ContentService
        .createTextOutput(JSON.stringify({ status: 'ok', message: 'Connection successful' }))
        .setMimeType(ContentService.MimeType.JSON);
    }

    // Append boarding record
    sheet.appendRow([
      data.timestamp || new Date().toISOString(),
      data.date || Utilities.formatDate(new Date(), Session.getScriptTimeZone(), 'yyyy-MM-dd'),
      data.ticket || '',
      data.busId || '',
      data.busRoute || '',
      data.busDeparture || '',
      data.notes || '',
      data.id || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: GET endpoint to verify the script is alive
function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok', message: 'Combuses Boarding Log endpoint is active' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Deploy → New deployment**
4. Choose type: **Web app**
5. Settings:
   - Execute as: **Me**
   - Who has access: **Anyone** (so the app can send data without auth)
6. Click **Deploy** and copy the URL
7. Paste that URL into the app's Settings → Google Sheets Integration

---

## How Agents Use It

### Daily Workflow

1. Open the app on their phone
2. Select the active bus from the dropdown
3. For each passenger boarding:
   - Point phone camera at ticket QR → auto-fills ticket ID
   - OR type the ticket number manually
   - Tap **Register Boarding**
4. The counter shows how many have boarded this bus
5. Data syncs to Google Sheets in real-time

### Managing Buses

- Go to **Settings → Bus Fleet**
- Add, edit, or remove buses as the schedule changes
- Each bus has: ID/plate number, route (OD pair), and departure time

---

## Reporting

### In Google Sheets

Once data flows in, you can create pivot tables to answer:
- **How many passengers per bus?** → Pivot by Bus ID, count Tickets
- **Revenue allocation by bus**: Cross-reference ticket revenue with boarding bus
- **Utilization by route**: Pivot by Route, count unique dates and passengers
- **Peak times**: Pivot by Departure time

### CSV Export

The app also has a built-in CSV export (Log tab → Export CSV) filtered by date and bus.

---

## Offline Behavior

- The app stores all records locally on the device
- If Google Sheets is unreachable, records are still saved locally
- Records can be exported as CSV anytime as a backup
- **Important**: Local storage is per-device. If an agent switches phones, local history won't transfer (but Sheets data is safe)

---

## Technical Notes

- **QR Scanner**: Uses the device camera via `html5-qrcode` library. Works on any modern Android/iOS browser.
- **No backend needed**: The app is a single HTML file with localStorage for state.
- **Google Sheets sync**: Uses `no-cors` fetch to the Apps Script endpoint. Data flows one-way (app → sheet).
- **Security**: The Apps Script URL is the only "credential." Don't share it publicly. For production, consider adding a simple API key check in the script.

---

## Customization

### Adding Fields
To add more fields to the boarding record, edit the `registerBoarding()` function in the HTML and add corresponding columns in the Google Sheet and Apps Script.

### Branding
Update the logo, colors, and company name in the CSS variables at the top of the HTML file.

### Multiple Locations
If Combuses operates from multiple airports/terminals, you could:
- Add a "Location" selector to the app
- Deploy separate instances per location
- Or add a location field to each record
