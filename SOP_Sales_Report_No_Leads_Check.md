# Work Instructions: Checking Whether a Missing/Empty Sales Report Really Means "No Leads"

**Purpose:** Answer tickets like "We didn't receive the sales report today — were there really no leads?" This is a different question from a publication's download count or a report's opt-in export: it's about the *internal* lead pipeline (`opt_in_marketing_internal`), not downloads or a specific report's external opt-ins.

**Audience:** IT Ops / Zendesk ticket handlers with access to the ops dashboard (Grafana).

**Tools needed:**
- Ops dashboard (Grafana) — `ops-dashboard.cloudsecurityalliance.org`

**⚠️ Not covered by the Business Activity dashboard.** `ops-dashboard.cloudsecurityalliance.org/d/business/business-activity` only has panels for signups (today/week/month), total users, active sessions, and articles published/downloads. It has no lead or `personal_data_submissions` panel. This check always requires a manual Explore query — there is no dashboard shortcut.

**✅ Validated:** Dry-run against ticket #122913 ("Sales report 06-10-2025"), where the query returned zero matching rows — confirming there genuinely were no leads that day. Re-run against ticket #155788 ("Sales report 08/06/2026") using the identical filter pattern returned 5 rows — meaning the "no leads" theory was wrong; there was unexported lead data sitting in the table, pointing to the export/report job itself not having run. Both outcomes are valid and mean different things — see Step 3.

---

## Step 1 — Open Grafana Explore

1. Go to `ops-dashboard.cloudsecurityalliance.org/explore`.
2. Confirm the datasource is **Main Site PostgreSQL** and the dataset is **csa_production**.
3. Confirm the query editor is in **Builder** mode, not **Code**.

## Step 2 — Build the query using the Builder UI only

Table: `personal_data_submissions`. Build the entire query before clicking **Run Query** — see the Cloudflare WAF note in the download-count SOP, which applies identically here.

1. **Table:** select `personal_data_submissions`.
2. For a **count-only check** (usually all that's needed to answer the ticket): set Data operations to `COUNT` on column `id`.
3. **Filter**, three rows joined with AND:
   - `opt_in_marketing_internal` `==` `Yes` (renders as a Yes/No radio button — it's boolean)
   - `exported_for_sales_at` `Is null`
   - `created_at` `>=` the start of the report's window (typically the report day's midnight, e.g. `2026-08-06 00:00:00` — adjust to match whatever period the requester says was missing)
4. Click **Run Query** once.

This is the exact filter pattern from the SQL Ryan Richardson gave in #122913:
```sql
SELECT COUNT(id)
FROM personal_data_submissions
WHERE opt_in_marketing_internal = true
  AND exported_for_sales_at IS NULL
  AND created_at >= '{start of report window}'
```

## Step 3 — Interpret the result

- **Zero rows:** the "no leads" explanation is correct — there's genuinely nothing unexported for that window. Report this back as confirmed (as in #122913).
- **One or more rows:** the "no leads" explanation is wrong. There *is* unexported lead data — the export/report job itself didn't run or failed. Per Ryan's explanation in #122913, unexported records just roll into the next successful run rather than being lost, so nothing is missing, but the job failure itself should be flagged to whoever owns that job (Ryan Richardson handled it previously) rather than closed out as a non-issue.

## Step 4 — If the actual lead records are needed (not just the count)

Only pull full records if the requester specifically needs the underlying data (e.g. to confirm which leads were affected), not by default.

1. Remove the `COUNT` data operation and add plain columns instead: `id`, `name_given`, `name_family`, `email`, `organization_name`, `organization_title`, `created_at`.
2. Keep the same three filters from Step 2.
3. Run once.
4. **This is PII.** Export to CSV and attach as a **private** ticket comment only — same rule as `SOP_ThirdParty_OptIn_Leads.md` Step 4. Do not post publicly or email outside the ticket. Name the file descriptively, e.g. `Ticket{#}_Unexported-Sales-Leads_{date}.csv`.

## Step 5 — Sanity-check and report

- State clearly which of the two outcomes in Step 3 applies, and don't conflate "nothing is lost" with "nothing is wrong" — a job failure is worth flagging even though the data itself is safe.
- **Timezone display note:** the `created_at` values shown in Grafana's Explore table may render a few hours earlier than the filter boundary you entered (e.g. filtering `>= 2026-08-06 00:00:00` can show result rows timestamped late on 2026-08-05). This has been observed to be a display/timezone rendering quirk, not a filter error — the row count matches independently-run COUNT-only queries using the same filter. Don't take the displayed date at face value as evidence the filter is wrong; if in doubt, re-run the COUNT-only version of the query to cross-check the row count.

---

## ⚠️ Known issue: Cloudflare WAF blocks

Same underlying ops-dashboard/Grafana workflow as the other SOPs, and the same mitigation applies — see `SOP_Publication_Download_Counts.md` for the full avoid/recover procedure. In short: Builder UI only, build the entire query before running, one execution per attempt, 60–90s cooldown and a clean `/explore` reload if blocked.

---

## Reference query (for context only — do not paste this into the Code editor)

```sql
-- Count-only check
SELECT COUNT(id)
FROM personal_data_submissions
WHERE opt_in_marketing_internal = true
  AND exported_for_sales_at IS NULL
  AND created_at >= '{start of report window}';

-- Full record pull (PII — private ticket attachment only)
SELECT id, name_given, name_family, email, organization_name, organization_title, created_at
FROM personal_data_submissions
WHERE opt_in_marketing_internal = true
  AND exported_for_sales_at IS NULL
  AND created_at >= '{start of report window}';
```
