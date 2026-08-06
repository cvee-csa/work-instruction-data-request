# Work Instructions: Verifying the Public CSA Staff Page Against Rippling Employment Records

**Purpose:** Confirm that everyone listed on the public "CSA Staff" page (`cloudsecurityalliance.org/about/csa-staff`) is actually a current employee, by cross-referencing each name against Rippling's People directory. This catches bios left up after someone leaves or is renamed, and the rarer case of someone publicly listed as staff who was never actually onboarded.

**Audience:** Whoever maintains the CSA website team page (Marketing/Web), or Ops handling a request to audit it.

**Tools needed:**
- Chrome browser
- Rippling admin access with People visibility — `app.rippling.com`

**✅ Validated:** Full run completed 2026-08-06 against the live staff page (47 names) and Rippling (60 active / 70 terminated / 6 new-hire records at time of run). Found 6 employment-status discrepancies and 4 title-only mismatches — see Step 4 sample output.

---

## Step 1 — Pull the public staff list

1. Go to `cloudsecurityalliance.org/about/` and click **CSA Staff** (or navigate directly to `/about/csa-staff`).
2. Read the full page text. The list has a "Select Department" filter dropdown, but it defaults to showing everyone — you don't need to touch it or paginate.
3. Record each person's name exactly as displayed, plus their listed title.

## Step 2 — Open Rippling's People directory

1. In Rippling, use the app switcher (top-left product dropdown, e.g. "Tools") → **HR** → **People**. Direct URL: `app.rippling.com/employee-list/list`.
2. Note the four tabs and their counts: **Active**, **New hires**, **Offboarding**, **Terminated**.

⚠️ **The People table is virtualized/lazy-loaded.** A single page-text capture only returns ~20 rows. Don't treat one capture as the full roster — either scroll and re-capture repeatedly (rows are sorted alphabetically by last name, which makes it easy to track gaps), or skip straight to per-name search (Step 3) once you have the list from Step 1. Per-name search is faster and less error-prone.

## Step 3 — Look up each staff-page name individually

1. Type each name from Step 1 into the People search box (top right of the table). The search applies across all four tabs simultaneously — tab counts update live. Example: searching "Raina" showed `New hires (1)` with the other three tabs at 0.
2. Record which tab the match falls into, and the exact status/title/start date shown:
   - **Active** → confirmed current employee. Compare the Rippling title against the public page's title and note any material mismatch (see Step 4).
   - **Terminated** → discrepancy. The person is on the public page but no longer employed.
   - **New hires** → check the sub-stage (Requested hire / Offer sent / **Offer accepted** / Onboarding complete). An "Offer accepted" record with a start date far in the past and status "Link to begin onboarding not sent yet" means the person was never actually onboarded, despite being publicly listed as staff.
   - **No match in any tab** → don't conclude they're missing from Rippling outright; double-check spelling and nickname variants first (see next point).
3. Minor name-format differences are expected and are **not** discrepancies by themselves: the website sometimes uses nicknames, middle initials, or credentials that Rippling doesn't carry — e.g. "Jim Reavis" (site) = "James Reavis" (Rippling); "Larry Hughes" = "Lawrence Hughes"; "Luciano (J.R.) Santos" and "Jeffrey Westcott, CPA" appear without the parenthetical/credential in Rippling.

## Step 4 — Classify and report

Split findings into two buckets, in priority order:

1. **Employment-status discrepancies** (report first — these mean the website is actively wrong, not just cosmetically outdated): person listed on the public page but **Terminated** or stuck **un-onboarded** in Rippling.
2. **Title-only discrepancies** (report as lower priority — may just mean one system hasn't been synced): person is Active in both places, but the job title differs materially between the site and Rippling.

**Sample output — 2026-08-06 run** (47 public staff names vs. Rippling):

*Employment-status discrepancies (6):*
| Name (as listed on site) | Site title | Rippling status |
|---|---|---|
| Megan Czaplinski | AVP of Growth | Terminated |
| John DiMaria | Chief of Staff | Terminated |
| Lawrence "Larry" Hughes | VP of Research & Development | Terminated (02/10/2025) |
| Paige McKenna | Account Executive | Terminated |
| Kaitlin Morgan | Cloud IT Operations Administrator | Terminated |
| Kapil Raina | Executive Advisor to the CEO | Never onboarded — offer accepted 06/30/2025, onboarding link never sent |

*Title-only discrepancies (4):*
| Name | Site title | Rippling title |
|---|---|---|
| John Yeoh | Chief Scientific Officer | VP of Research |
| Kurt Seifried | Chief Innovation Officer | AVP of Innovation |
| Carolina Ozan | AVP of Special Projects | AVP of Operations |
| Andy Ruth | Content Developer | Sr. Research Analyst |

The remaining 37 names matched cleanly on both employment status and title.

## Step 5 — Follow-up

- Employment-status discrepancies go back to whoever owns the website content (Marketing/Web) to pull the bio down or update it. Rippling is the source of truth for employment status — the website is not.
- Title-only discrepancies are worth flagging but lower urgency; confirm with the person's manager which title is current before editing anything.
- This is a **read-only audit**. Do not edit Rippling records as part of this check.

---

## ⚠️ Known issues

None observed during the 2026-08-06 run — no rate-limiting or access issues surfaced against either the CSA marketing site or Rippling.
