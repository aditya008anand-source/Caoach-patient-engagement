# Coach Engagement & Plan Expiry Dashboard

A single self-contained page (`index.html`) for health coaches to see which patients'
plans are expiring in a given month, and which coach has been most engaged with each
patient during their plan period.

It reads your two Google Sheet tabs live, every time the page is opened (or when you
click **Refresh data**). Nothing needs to be re-built or re-uploaded when you add new
rows to the sheet — just publish and refresh.

## What it does

- Joins the **patient data** sheet and the **session data** sheet by phone number.
- For every plan enrollment (a patient can appear more than once if they renew), it
  works out the plan window (assigned date → assigned date + duration + extension) and
  finds the coach with the most **completed** sessions inside that exact window.
- If nobody has completed a session yet but a Nutritionist has been booked, it shows
  that Nutritionist's name flagged as "booked, not completed."
- Excludes all MAMILY plans. Maps any 1-month plan to "Smart CGM plan."
- Groups everything by expiry month, with cascading filters: Month → Coach → Plan.
- Export button downloads the currently filtered view as CSV.
- If the live fetch fails (rare — network/CORS hiccup), an upload panel appears so you
  can drop in exported CSV files instead.

## One-time setup: publish your Google Sheet tabs

For each of the two tabs (patient data, session data):

1. In Google Sheets: **File → Share → Publish to web**.
2. Choose the specific sheet/tab, format **Comma-separated values (.csv)**, click **Publish**.
3. Copy the generated URL — it looks like:
   `https://docs.google.com/spreadsheets/d/e/XXXXX/pub?gid=NNNNNN&single=true&output=csv`

The two links you already gave me are baked in as the defaults in `index.html`. If the
sheet URL ever changes, open the dashboard and click **Data source settings** in the
header — it lets you paste new URLs without touching any code, and remembers them in
the browser.

> Note: a published sheet is technically viewable by anyone with the link, since that's
> what "publish to web" means. If this data shouldn't be publicly reachable, keep the
> published link unlisted (don't post it anywhere public) or use the manual CSV upload
> mode instead of live fetch — worth a quick internal call on your end.

## Deploying to Vercel via GitHub

1. Create a new GitHub repository (can be private).
2. Add `index.html`, `.gitignore`, and this `README.md` to the repo root and push:
   ```
   git init
   git add index.html .gitignore README.md
   git commit -m "Coach engagement dashboard"
   git branch -M main
   git remote add origin <your-repo-url>
   git push -u origin main
   ```
3. Go to [vercel.com](https://vercel.com) → **Add New Project** → import that GitHub repo.
4. Vercel will auto-detect it as a static site — no build command needed. Click **Deploy**.
5. You'll get a public URL (e.g. `your-project.vercel.app`) you can share with any coach.
   Anyone with the link can open it; no login required.

Every time you push a change to `index.html`, Vercel redeploys automatically. You will
**not** need to redeploy just because the sheet data changed — the page always fetches
fresh data on load.

## Monthly workflow

1. At month-end, add new patient rows to the patient sheet and let session data keep
   flowing into the session sheet as usual.
2. Open the dashboard (or click **Refresh data**) — it re-reads both sheets live.
3. Use the Month filter to jump to the month you care about (defaults to the current
   month on load).

## Known data-quality notes worth a look on your end

- About half of patient rows don't have a phone number that matches any row in the
  session sheet — for those, the coach/engagement columns will be blank (no session
  data found), which is expected, not a bug.
- A handful of plan names look like data-entry slips (e.g. "Docto Essential" instead of
  "Doctor Essential", one row with a person's name instead of a plan name) — these show
  up as their own entries in the Plan filter since the tool doesn't guess corrections.
- Some `Plan Duration` values are non-numeric ("No", "-", blank) — those rows show
  patient details but no expiry date or engagement, and appear under an "Unspecified"
  bucket in the Month filter, per your instruction to leave them blank rather than guess.
