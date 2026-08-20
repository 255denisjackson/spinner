# CREDORA — Setup Guide

CREDORA is a two-piece system:

1. **The website** (`index.html`) — the dashboard you open in a browser.
2. **The backend** (`Code.gs`) — a free Google Apps Script attached to a Google Sheet, which acts as your database + API.

No paid hosting, no server, no Node.js install required. Total setup time: ~5 minutes.

---

## Step 1 — Create the Google Sheet backend

1. Go to [sheets.google.com](https://sheets.google.com) and create a **new blank spreadsheet**. Name it "CREDORA Database".
2. In the menu, go to **Extensions → Apps Script**. A new tab opens with a code editor.
3. Delete everything in the default `Code.gs` file that opens.
4. Open the `Code.gs` file I generated for you, copy **all of it**, and paste it into the Apps Script editor.
5. Click the **Save** icon (or Ctrl/Cmd+S).
6. At the top of the Apps Script editor, find the function dropdown (it may say `doGet`). Change it to **`setup`**.
7. Click **Run** (▶). The first time, Google will ask you to authorize the script — click through **Review permissions → (your account) → Advanced → Go to CREDORA (unsafe) → Allow**. This warning appears because it's your own personal script, not a published app — it's safe to allow.
8. Once it finishes, go back to your Google Sheet — you'll see new tabs: `Users`, `Customers`, `LoanApplications`, `KYC`, `CreditReports`, `RiskAssessments`, `AuditLogs`, `LoanProducts`, `Notifications`. These ARE your database tables.

## Step 2 — Deploy it as a web app (this gives you the API URL)

1. Back in the Apps Script editor, click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Fill in:
   - **Description**: CREDORA API v1
   - **Execute as**: **Me** (your account)
   - **Who has access**: **Anyone** (this lets your website call it — the URL itself is your only "secret," so don't publish it publicly)
4. Click **Deploy**. Authorize again if asked.
5. Copy the **Web app URL** shown — it looks like:
   `https://script.google.com/macros/s/AKfycb.../exec`

Keep this tab open — you'll need this URL in Step 3.

> **Updating the backend later:** if you ever edit `Code.gs` again, you must click **Deploy → Manage deployments → ✏️ Edit → New version → Deploy** for changes to take effect. Saving alone is not enough.

## Step 3 — Connect the website to your Sheet

1. Open `index.html` (double-click it, or drag it into a Chrome tab).
2. On the login screen, paste your Web app URL into the **"API URL"** field at the bottom and click **Save API URL**.
3. Log in with the demo credentials seeded by `setup()`:
   - `admin@credora.demo` / `admin123` (Super Administrator)
   - `officer@credora.demo` / `officer123` (Loan Officer)
4. **Change these passwords** by editing the `Users` tab directly in your Google Sheet (find the row, edit the `password` column, save).

You're in. Every button click in the website reads and writes rows in your Google Sheet in real time — open the Sheet in another tab and watch rows appear as you use the app.

---

## Hosting the website itself (optional)

Opening `index.html` directly from your computer works fine for solo use and testing. If you want a shareable link for your team:

- **Easiest**: upload `index.html` to Google Drive → right-click → "Publish to the web" won't render HTML directly, so instead use a static host:
- **GitHub Pages** (free): create a GitHub repo, upload `index.html`, enable Pages in repo settings — you'll get a `https://yourname.github.io/repo/` link.
- **Netlify Drop** (free, no account needed for a quick test): go to [app.netlify.com/drop](https://app.netlify.com/drop) and drag `index.html` in — you get an instant public link.

Anyone with the link can then use the same Sheet as the shared database (as long as they also have your API URL saved — it's baked into their browser's local storage after they enter it once).

---

## Important limits of the Google Sheets approach (be aware of these)

- **Not encrypted at rest** the way a real database would be — anyone with edit access to the Sheet can see all customer data in plain text. Fine for a demo/pilot; not appropriate for real customer PII/production lending without a proper database and encryption.
- **No real authentication security** — passwords are stored in plain text in the `Users` tab. Do not use this for real customer-facing production use without adding proper hashing (this would require moving off Apps Script to a real backend).
- **Apps Script quotas** — free Google accounts get ~20,000 URL Fetch calls/day and similar limits on script runtime; fine for a pilot with a handful of loan officers, not built for high transaction volume.
- **All KYC and credit bureau results are MOCK/DEMO DATA** — clearly labeled in the UI. To go live you'd replace `runKYC`/`runCreditCheck` in `Code.gs` with real `UrlFetchApp.fetch()` calls to a licensed provider.

Treat this build as a **working prototype/pilot tool** — it demonstrates the full CREDORA workflow end-to-end and is genuinely usable for a small team, but a real production deployment (bank-grade security, real bureau integrations, real database) is a larger engineering project.

---

## What's included in this first version

- Login / role display
- Dashboard with live stats + charts
- Customer registration & profile (KYC status, credit score, risk level)
- Loan application creation
- Mock KYC verification
- Mock credit bureau check
- Financial statement entry → affordability calculation
- Configurable-weight credit scoring / risk engine (0–1000, "CREDORA Internal Risk Score")
- Basic fraud heuristic flags
- Rule-based decision engine (Approve / Review / Reject) with explanation
- Human review screen with mandatory override reason
- Reports (CSV export)
- Immutable audit log
- Dark / light mode, mobile-responsive layout

**Not yet built** (from the original 50-section spec — ask if you'd like any of these next): document upload & storage, multi-level approval workflows by loan amount, collateral module, webhook system, API key management dashboard, notification delivery (email/SMS), organization/multi-tenant management, customer self-service portal.
