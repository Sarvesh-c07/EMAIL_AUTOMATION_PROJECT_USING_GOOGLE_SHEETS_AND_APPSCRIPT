# 📧 Email Automation Suite

A Google Apps Script web app that lets you send bulk, templated business emails (e.g. Balance Confirmation requests) directly from a Google Sheet — complete with a password-protected login page, a live dashboard (Sent / Pending / Total), and a recipient status tracker.

---

## 🧩 Query Decomposition — "How do I deploy this on my own Google Sheet?"

This README breaks the deployment task into sub-queries, each solved as a discrete step. Follow them in order.

---

### Q1. What do I need before I start?

**Sub-tasks:**
- [ ] A Google account with access to Google Sheets
- [ ] Gmail enabled on that account (used to send emails via `GmailApp`)
- [ ] The 5 files from this repo: `Code.gs`, `Login.html`, `Index.html`, `Styles.html`, `Scripts.html`

---

### Q2. How do I set up the Google Sheet that holds my recipient data?

**Sub-tasks:**
1. Go to [sheets.google.com](https://sheets.google.com) → create a **New Spreadsheet**.
2. Rename it (e.g. `Balance Confirmation FY 2025-26`).
3. Create one sheet (tab) per recipient group — e.g. `3S Pharma`, `3S Corp`. You can have multiple sheets; the app auto-detects all of them in a dropdown.
4. In each sheet, set up columns **exactly** in this order (Row 1 = headers):

| Column | A | B | C | D |
|--------|---|---|---|---|
| Header | Name | Email | Status | Date Sent |

5. Fill in recipient `Name` and `Email` starting from Row 2. Leave `Status` and `Date Sent` blank — the script fills these automatically after sending.

> ⚠️ Column order matters — the script reads columns by position (A=Name, B=Email, C=Status, D=Date).

---

### Q3. How do I attach the Apps Script project to this Sheet?

**Sub-tasks:**
1. In your Sheet, go to **Extensions → Apps Script**.
2. This opens a new Apps Script project bound to your spreadsheet.
3. Delete the default boilerplate `Code.gs` content.

---

### Q4. How do I add the project files into Apps Script?

**Sub-tasks (repeat per file):**
1. **Code.gs** — paste the contents of this repo's `Code.gs` into the existing `Code.gs` file.
2. **Login.html** — click `+` next to "Files" → **HTML** → name it exactly `Login` → paste contents of `Login.html`.
3. **Index.html** — click `+` → **HTML** → name it exactly `Index` → paste contents of `Index.html`.
4. **Styles.html** — click `+` → **HTML** → name it exactly `Styles` → paste contents of `Styles.html`.
5. **Scripts.html** — click `+` → **HTML** → name it exactly `Scripts` → paste contents of `Scripts.html`.
6. Click the 💾 **Save** icon (or `Ctrl+S` / `Cmd+S`).

> ⚠️ File names must match exactly (`Login`, `Index`, `Styles`, `Scripts`) — they're referenced by name in `Code.gs` via `HtmlService.createTemplateFromFile(...)` and `include(...)`.

---

### Q5. How do I set my own login password?

**Sub-tasks:**
1. Open `Code.gs` in the Apps Script editor.
2. Find this line near the top:
   ```js
   const APP_PASSWORD = "hariom00@";
   ```
3. Replace `"hariom00@"` with your own password.
4. Open `Login.html`, find the matching line inside the `<script>` block:
   ```js
   const CORRECT_PASSWORD = "hariom00@";
   ```
5. Replace it with the **same** password you set in Step 2.

> 🔒 **Security note:** This is a basic client+server password check, not enterprise-grade auth. Don't reuse a sensitive password, and don't make the underlying script/repo public with the real password still in it.

---

### Q6. How do I customize the email template?

**Sub-tasks:**
1. In `Code.gs`, locate the `sendEmails()` function.
2. Edit the `htmlBody` template string — update the company name, signatory, subject line, and email body to match your business.
3. Update the sender display name in the `GmailApp.sendEmail(...)` options object (`name: "..."`).

---

### Q7. How do I deploy this as a web app?

**Sub-tasks:**
1. In the Apps Script editor, click **Deploy → New deployment**.
2. Click the ⚙️ gear icon next to "Select type" → choose **Web app**.
3. Fill in:
   - **Description:** e.g. `Email Automation Suite v1`
   - **Execute as:** `Me (your email)`
   - **Who has access:** `Only myself` (recommended) or `Anyone with the link` if others need access
4. Click **Deploy**.
5. **Authorize** the app when prompted — Google will warn "This app isn't verified" since it's your personal script. Click **Advanced → Go to (project name) → Allow**. This grants Gmail + Sheets permissions.
6. Copy the **Web app URL** shown after deployment.

---

### Q8. How do I access and use the app?

**Sub-tasks:**
1. Open the **Web app URL** in your browser → this loads the password-protected **Login** page.
2. Enter the password you set in Q5.
3. You'll land on the **Dashboard** (`Index` page) showing:
   - A sheet selector dropdown (auto-populated from your spreadsheet's tabs)
   - Sent / Pending / Total cards
   - A recipient table with live status
4. Click **Simulate App** to dry-run a sheet (no emails sent — just shows counts).
5. Click **Send Emails**, enter your password again when prompted, and confirm. The script will:
   - Skip rows already marked `Sent`
   - Send the templated email to each pending recipient
   - Update `Status` → `Sent` and `Date Sent` → current timestamp in the sheet

---

### Q9. How do I update the app after making code changes?

**Sub-tasks:**
1. Make your edits in `Code.gs` / any `.html` file.
2. Save (`Ctrl+S`).
3. Go to **Deploy → Manage deployments**.
4. Click the ✏️ pencil icon on your active deployment.
5. Under **Version**, select **New version** → **Deploy**.

> ℹ️ Editing code without creating a new version will **not** update the live web app URL.

---

### Q10. Troubleshooting — common issues

| Issue | Likely Cause | Fix |
|---|---|---|
| Blank page after login | HTML file names don't match (`Index`, `Login`, etc.) | Re-check exact spelling/casing in Apps Script files |
| "Sheet not found" error | Dropdown sheet name doesn't match an actual tab | Refresh the page; verify sheet tab names |
| Emails not sending | Gmail daily quota exceeded (100/day for free Gmail, 1500 for Workspace) | Wait 24h or use a Workspace account |
| "Authorization required" loop | First-time script authorization wasn't completed | Redo Deploy → Authorize flow (Q7, step 5) |
| Wrong password always rejected | Password mismatch between `Code.gs` and `Login.html` | Ensure both constants are identical |

---

## 📁 File Reference

| File | Purpose |
|---|---|
| `Code.gs` | Server-side logic: auth, sheet read/write, email sending |
| `Login.html` | Password-gated entry page with lockout/attempt tracking |
| `Index.html` | Main dashboard UI shell |
| `Styles.html` | Shared CSS for dashboard |
| `Scripts.html` | Client-side JS: loads sheets, renders table, triggers send/simulate |

---

## ⚠️ Disclaimer

This tool sends real emails via your Gmail account when "Send Emails" is used. Always run **Simulate App** first to verify recipient counts before sending live.
