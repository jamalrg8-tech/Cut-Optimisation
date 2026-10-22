# Eurolux Doors and Windows, Cut Optimization

A single-page, installable PWA for optimising SCHUCO aluminium profile cutting
lists — paste, type, or import a parts list, it groups parts by Article No.
automatically and runs a cutting-stock optimisation per profile, then lets you
export the result as Excel or a PDF report. All data stays in the browser
(localStorage) — nothing is sent anywhere.

## What's in this folder

```
index.html      the whole app (HTML + CSS + JS, all inline)
manifest.json   PWA manifest (name, icons, colours)
sw.js           service worker — caches the app shell for offline use
icons/          app icons (192, 512, maskable, apple-touch, favicon)
vercel.json     sets correct no-cache headers for sw.js / manifest.json on Vercel
```

`index.html` loads a few small resources from public CDNs on first run:
three libraries for Excel import/export, PDF export, and reading embedded
Excel images (SheetJS, jsPDF and JSZip, all from cdnjs) and a Google Fonts
stylesheet (Barlow Semi Condensed / IBM Plex Sans / IBM Plex Mono). The
service worker caches all of them, so the app keeps working offline afterwards
— export included — once it's been opened online at least once.

## App layout

- **Dashboard** — the home screen: start a new optimisation, jump to job
  settings or the stock register, and see every saved job with Open / Rename
  / Duplicate / Delete actions (click anywhere on a job's row to open it).
- **Parts List** — build or import the cutting list for the current job. The
  "Optimise cutting layout" button appears both above and below the list.
- **Results** — the optimised cutting layout per profile, with an Export
  Excel / Export PDF report toolbar. When a job's parts came from an Excel
  workbook with more than one sheet/tab, a **Whole Job / By Sheet** switch
  appears above the results: *Whole Job* is the combined optimisation
  across every sheet (same-profile parts from different sheets can share a
  bar, which is usually the most efficient view); *By Sheet* optimises
  each source sheet completely on its own, as if it were being cut
  separately — useful when each sheet is its own door/window that needs
  its own cut list, even though it can end up using a little more stock in
  total than cutting everything together. Both views are included
  automatically in the Excel and PDF exports.
- **Stock** — the shared bars-on-hand register (see below).

## Importing from Excel

Click **Import Excel** on the Parts List tab and pick a workbook. The import:

- **Reads every sheet in the workbook**, not just the first one — if a
  workbook has several sheets (e.g. one per door/window elevation, all part
  of the same system), every sheet that looks like a cutting list is
  imported and combined into one parts list. Each imported row is tagged
  with the sheet it came from (visible in the Excel export's "Parts Input"
  sheet). A sheet that doesn't look like a raw cutting list — a "Summary"/
  "Totals"/"Report" tab, or any sheet whose columns look like an aggregate
  report (Yield %, Bars Required, Waste, etc.) rather than raw parts — is
  skipped automatically rather than imported as garbage data. The import
  toast tells you how many sheets were used and how many were skipped.
- **Imports the Position column** when the workbook has one (next to
  Article No. / Description, or alongside the per-item columns). Values are
  normalised to "Vertical" or "Horizontal" (abbreviations like "V"/"H" or
  "Vert"/"Horiz" are recognised too); anything else is kept as typed rather
  than discarded. If the column header is in an unusual spot (e.g. a merged
  header cell left the label one column off from the actual data), the
  import also scans the data itself for Vertical/Horizontal-looking values
  and uses that column instead, so Position still comes in correctly even
  then. Position shows up as an editable dropdown on each row in the Parts
  List, and as its own column in the Results tab's per-bar parts breakdown
  and in the Excel export — it's for your reference on the cut list and
  doesn't affect how the optimiser packs the bars.
- **Picks up an "Application" reference image** per row when the workbook
  has a small diagram/icon embedded in an Application column (this is a
  best-effort extraction straight from the Excel file's internal structure,
  so unusual templates may not carry an image over — in that case, or for
  any row, you can attach or replace one yourself with the **+ Image**
  button on that row in the Parts List).

You can also paste rows (Job menu → Parts List → Paste rows) using
`Article No, Description, Length(mm), Qty, Position` — Description, Qty and
Position are all optional. (Pasted rows don't carry an image; add one with
the row's **+ Image** button if needed.)

## Parts List columns

Article No., Description, Length, Qty and Position are editable directly
in the table. **Application** shows a thumbnail of that row's reference
image when one is set (from import or upload) — click the small **×** on
the thumbnail to remove it, or **+ Image** to attach/replace one from your
computer. (The old "Tag" column, used internally to tell apart multiple
pieces cut from the same Article No. on a tailored multi-item cut-list, is
no longer shown — it's still tracked in the background and still appears
in the Results breakdown and Excel export.)

## Stock register

A running count of bars on hand per Article No., shared across every job
(it's physical inventory, not something tied to one job). The first time a
profile is used anywhere — typed into a part row, imported from Excel, or
added manually from **Stock → Add profile** — it's registered automatically
at **100 bars**. Edit any quantity directly in the Stock tab, or from a
Results card click **"Record cut — deduct N bars from stock"** after running
an optimisation to subtract the bars that job actually used. If a job needs
more bars than are on hand for a profile, the Results tab shows a shortfall
warning naming the affected Article No.s.

## Saving jobs / choosing where files go

**Job actions → Save job as file (.json)**, and the Excel/PDF export buttons,
all go through the same save routine, which picks the best option your
browser supports:

1. **Desktop Chrome or Edge**: a native "Save As" dialog opens so you can
   choose the folder and filename yourself.
2. **Other browsers** (Safari, Firefox, mobile): the file downloads straight
   to your browser's normal Downloads location — those browsers don't yet
   support letting a web page open a native save dialog.

**Job actions → Load job from file (.json)** brings a saved job back in on
any device/browser.

## Installing it as an app

Once it's hosted on HTTPS (Vercel/GitHub Pages/any static host), open the URL:

- **Desktop Chrome/Edge**: an install icon appears in the address bar, or
  Menu → "Install Eurolux Doors and Windows, Cut Optimization…".
- **Android Chrome**: Menu → "Add to Home screen" / an install banner.
- **iPhone/iPad Safari**: Share button → "Add to Home Screen".

After installing, it opens in its own window/icon and works offline.

## Deploy to Vercel

1. Push this folder to a GitHub repo (or drag-and-drop it into the Vercel
   dashboard — "Add New… → Project → Deploy" supports a folder upload).
2. In Vercel, "Add New Project", import the repo, framework preset
   **Other** / static — no build command needed, output directory is the
   repo root.
3. Deploy. Vercel serves static files with HTTPS automatically, which is all
   a PWA needs to be installable.

Or with the Vercel CLI, from inside this folder:

```
npm i -g vercel
vercel --prod
```

## Deploy to GitHub Pages

1. Create a new repo and push the contents of this folder to it (commit
   `index.html`, `manifest.json`, `sw.js`, `icons/` at the repo root).
2. Repo → Settings → Pages → Source: "Deploy from a branch" → branch `main`,
   folder `/ (root)`.
3. Wait a minute for the first deploy, then open the `https://<you>.github.io/<repo>/`
   URL GitHub gives you.

## Updating the app after you make changes

Browsers cache the service worker aggressively by nature, so after editing
`index.html` bump the version string in `sw.js`:

```js
var CACHE_NAME = 'eurolux-cut-optimization-v6';   // was v5
```

That forces every visitor's browser to fetch the new files next time they
open the app (it activates on the *next* load after the update is fetched).

## Notes on the cutting-stock algorithm

- Kerf (saw-blade width — 10mm by default, editable in Job settings) is
  charged once per cut — i.e. every piece placed on a bar consumes
  `piece length + kerf` of that bar, matching how most shop-floor cutting
  reports (including opticutter-style reports) account for waste. The
  Results tab and PDF report mark each cut's kerf allowance with a small
  dark tick between segments on the bar diagram, rather than lumping all of
  a bar's kerf waste into one block at the end.
- The optimiser uses a best-fit-decreasing heuristic across whatever stock
  lengths you make available (globally, or per Article No. override) — it's
  a strong, fast heuristic, not a guaranteed mathematically optimal solver,
  same approach used by most commercial cut-list tools.
- Each Article No. is optimised completely separately, since different
  profiles are physically different stock. Position (Vertical/Horizontal)
  is carried through as reference metadata only — it does not change which
  bars a piece can be cut from.

## Data & privacy

Jobs, and the stock register, are saved in the browser's `localStorage`
only — nothing is uploaded anywhere. Use **Job actions → Save job as file
(.json)** to back a job up or move it to another device/browser, and **Load
job from file (.json)** to bring it back in.
