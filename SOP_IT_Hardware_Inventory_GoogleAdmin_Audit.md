# Work Instructions: Auditing the IT Hardware Inventory Against Google Admin Workspace

**Purpose:** Verify that the IT Hardware Inventory spreadsheet (`Sheet1`) accurately reflects what Google Admin Workspace actually shows for each employee's device — correct serial numbers, models, and hostnames — and promote the highest-confidence matches into the spreadsheet's `Confirmed` tab. This catches stale/incorrect serials, missing device records, and devices that exist in one system but not the other.

**Audience:** IT Ops maintaining the IT Hardware Inventory spreadsheet, or whoever handles a periodic request to reconcile it against Google Admin.

**Tools needed:**
- Google Admin Console with Devices visibility — `admin.google.com` → **Devices > Mobile & endpoints > Devices**
- The current `IT Hardware Inventory.xlsx` file
- Python 3 with `pandas` and `openpyxl` (for anything beyond a handful of rows — do this by hand only for small, one-off spot checks)

**✅ Validated:** Full run completed 2026-08-17 against a 75-row inventory and a 337-row Google Admin export. Result: 14 rows Confirmed, 44 Likely, 17 Needs review. 12 of the 14 Confirmed rows were new additions to the `Confirmed` tab (2 were already present under different sheet rows).

---

## Step 1 — Export the Google Admin device list (don't scrape it manually)

1. Go to **Devices > Mobile & endpoints > Devices** in the Admin console.
2. Before exporting, add these columns to the list view if not already shown: **Serial Number**, **Model**, **Host Name**, **Last Sync**, **Status**, **Ownership**, **Email**, **Device ID**, **OS/Type**.
3. Use the console's **Download** / export feature to get a CSV of the full device list. This is dramatically faster and less error-prone than paging through the UI and transcribing rows by hand — a manual scrape of ~330 devices took 7 pages and was fully superseded once a clean CSV export was available.
4. Confirm the CSV has one row per device with at least: Device Name, Name, Email, Ownership, Type, Model, First Sync, Last Sync, Status, Device ID, Serial Number, OS, Host Name.

## Step 2 — Load and normalize both sources

1. Load the inventory spreadsheet's `Sheet1` rows into a working structure (row number, employee name, processor, machine type, serial, model, memory, updated date — **check for extra columns past the ones you expect**; a "Chip" column with real Apple chip data existed past the columns used for matching and was easy to miss on a first read).
2. Load the Google Admin CSV.
3. Filter out placeholder/junk values before treating a field as "real data" on either side:
   - **Junk serials:** `DEFAULT STRING`, `SYSTEM SERIAL NUMBER`, `NAN`, empty string, and manually-entered placeholders like `????` — treat these as "no serial," not as a value to carry forward.
   - **Generic models:** `MAC OS`, `WINDOWS`, `LINUX`, `UNKNOWN`, `NAN`, empty string.
   - Note: many Windows-machine "serials" already in the spreadsheet are actually **Windows installation GUIDs**, not hardware serials — this is why they will never match Google Admin's true hardware Serial Number field, and isn't itself a discrepancy worth flagging.
4. **If using pandas:** `.astype(str)` does **not** reliably stringify `NaN` on newer pandas versions — it can leave real `NaN` floats in place, which then evaluate as "truthy" and silently pass an `is this real data` check. Always `.fillna('')` before `.astype(str)`, and/or use explicit `.notna()` checks when deciding whether a field counts as present.

## Step 3 — Match each spreadsheet row to a Google Admin record

Apply this hierarchy, stopping at the first tier that produces a match:

1. **Exact serial number match.**
2. **Hostname / machine name match** (normalize both sides to alphanumeric-only before comparing).
3. **Name/email match** — generate candidate email local-parts from the employee's name (e.g. first-initial + last name) and search for a substring match against last name. Maintain a small aliases list for known nickname/spelling mismatches between the spreadsheet and Google Admin identities (e.g. a shortened first name, a maiden/married name, a different preferred spelling) — this list will need occasional additions as new mismatches surface; don't assume "no match" means "not in Google Admin" without checking it first.

When a person has **multiple** Google Admin device records, prefer the one with, in order: a real (non-placeholder) serial, then a real (non-generic) model, then the most recent Last Sync. This avoids picking a generic placeholder record over the one with actual device detail.

## Step 4 — Classify confidence

- **Confirmed:** serial matches AND assigned user matches.
- **Likely:** serial is missing/unusable, but hostname or name signals line up consistently.
- **Needs review:** duplicate serials, conflicting identifiers, or a device with no reasonable match on either side.

Also flag, independent of the row-level confidence: duplicate serials (whether duplicated *within* the spreadsheet itself, or one Google Admin serial mapped to two different people), devices present in Google Admin but missing from the spreadsheet, devices in the spreadsheet with no footprint in Google Admin at all, and blank/junk serials.

## Step 5 — Update the inventory spreadsheet

For **Confirmed** and **Likely** rows only:
1. Update Serial Number/Device ID and Model Name/Number to the Google Admin values.
2. Fill Type of Machine only if it was previously blank (don't overwrite an existing value).
3. Append an audit note to Updated Date, e.g. `GAdmin {confidence} ({date}, last sync {admin_last_sync}) — {flags}`.
4. Highlight changed cells (e.g. light green fill) and attach a cell comment recording the previous value, so the edit is traceable and reversible.

For **Needs review** rows: highlight (e.g. light red/yellow) and note "NEEDS REVIEW" plus the specific flag — **do not change the underlying data** on these rows. They need a human decision, not an automatic overwrite.

## Step 6 — Promote high-confidence matches into the `Confirmed` tab

1. Take every row classified **Confirmed** in Step 4.
2. Check whether that person already has a row in the `Confirmed` tab (match by Google Admin email local-part, not by name string, since the two tabs may spell names differently) — skip anyone already present rather than duplicating them.
3. Map fields across the schema difference between `Sheet1` and `Confirmed`: MachineName/Type/Model/Updated Date come from Google Admin (source of truth); Processor and Memory aren't tracked in Google Admin, so pull those from the original spreadsheet row; MAC Address 1/2 have no Google Admin equivalent in a standard device export, so leave them blank rather than guessing.
4. Highlight newly-added rows so they're easy to spot in review.

## Step 7 — Regression check before delivering (required)

Before treating any update as final:
1. Diff **every column**, not just the ones you intended to touch, between the original file and the updated file, for every row that should be unchanged. Two real bugs were caught this way in past runs: a pandas NaN-handling bug that silently blanked out real serial numbers on unrelated rows, and a new column being written into a column that already held unrelated real data (a "Chip" column) because it hadn't been checked for existing content first.
2. Specifically re-verify: any row/column you did *not* intend to touch is byte-for-byte identical to the source; any "empty" column you're about to reuse for new data is actually empty across all rows before you write into it.
3. Only after a clean regression check should the file be delivered or treated as the new source of truth.

---

## Known issues

- **pandas `.astype(str)` + NaN:** see Step 2, item 4. Always test this in isolation against your pandas version before trusting it on real data (`pd.Series([np.nan, 'ABC123', None]).astype(str)`).
- **Reusing "empty" spreadsheet columns:** always re-check the target column across the *entire* column (not just the rows/columns you've already parsed into a working structure) before writing new data into it — a column read early on may have real data further down, or in a header row, that a partial read missed.
- **Windows serial vs. installation GUID:** expect most Windows rows to never match on serial; hostname/name matching is the normal path for them, not a fallback for a broken process.
