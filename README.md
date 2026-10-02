# work-instruction-data-request

Work instructions (SOPs) for recurring IT Ops / Zendesk ticket types.

## Files

- **SOP_Publication_Download_Counts.md** — How to look up a publication's download count via Grafana Explore (`interactions` table), including a bot-traffic sanity check and a required cross-session confirmation step before reporting any number.
  Sample ticket: **#153721** ("Disaster Recovery as a Service" download count, 4,128 downloads).
- **SOP_ThirdParty_OptIn_Leads.md** — How to pull the list of third-party opt-in leads for a specific report/publication (e.g. for a sponsor), via the `personal_data_submissions` table filtered by report slug. Output is PII — private ticket attachment only.
  Sample ticket: **#153521** ("2026 State of Modern Application and AI Security" / Miggo Security survey report opt-in leads).
- **SOP_Sales_Report_No_Leads_Check.md** — How to check whether a missing/empty daily sales report really means "no leads that day," or whether leads exist but the export job failed. Also covers pulling the underlying records (PII) if needed.
  Sample tickets: **#122913** (confirmed genuinely zero leads that day) and **#155788** (leads existed but the export job hadn't run).
- **csa-public-slack-invite-rotation.md** — How to rotate the public Slack invite link for the csa-public workspace and update the csaurl.org/csa-public-slack redirect, roughly every 2 weeks or before the current link's 30-day expiry.
  No specific ticket on file — this one runs on a recurring schedule rather than in response to a request.
- **SOP_CSA_Staff_Page_Rippling_Verification.md** — How to cross-reference the public CSA Staff page (`/about/csa-staff`) against Rippling's People directory to confirm everyone listed is actually a current employee, and how to classify employment-status vs. title-only discrepancies. Browser-based (no Grafana/SQL) — steps are all navigating the CSA website and the Rippling web app.
  Sample run: **2026-08-06** (47 public staff names checked; found 6 employment-status discrepancies — 5 terminated, 1 never onboarded — and 4 title-only mismatches).
- **SOP_IT_Hardware_Inventory_GoogleAdmin_Audit.md** — How to reconcile the IT Hardware Inventory spreadsheet against a Google Admin Workspace device export: export the device CSV, match rows by serial → hostname → name/email, classify confidence (Confirmed/Likely/Needs review), update the spreadsheet in place with tracked changes, and promote Confirmed matches into the `Confirmed` tab. Includes a required full-column regression check before delivering, plus known pandas/openpyxl pitfalls to avoid.
  Sample run: **2026-08-17** (75-row inventory vs. 337-row Google Admin export; 14 Confirmed, 44 Likely, 17 Needs review; 12 new rows promoted to `Confirmed`).
- **SOP_BLINK_Link_Click_Analytics.md** — How to pull click analytics for a csaurl.org short link in BL.INK: find the link (owner filter → All Users), record the dashboard totals, export the last year of click-level data, and summarize it into a PII-free spreadsheet (monthly/daily trend, referrers, countries, devices). Includes a required completeness check against the dashboard, a repeat-IP concentration check, and what to do before a link hits BL.INK's click-tracking cap.
  Sample ticket: **#159331** (`csaurl.org/csa-public-slack` analytics for Research; 7,835 clicks / 3,128 distinct IPs over 12 months, 13,894 clicks all-time).
