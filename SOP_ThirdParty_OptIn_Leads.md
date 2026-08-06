# Work Instructions: Pulling Third-Party Opt-In Leads for a Report/Publication

**Purpose:** Answer requests like "Can I get the third-party opt-ins and lead data for [report]?" — typically from Research/Marketing staff who need to share lead data with a sponsor (e.g., a survey report co-branded with a security vendor).

**Audience:** IT Ops / Zendesk ticket handlers with access to the ops dashboard (Grafana).

**Tools needed:**
- Ops dashboard (Grafana) — `ops-dashboard.cloudsecurityalliance.org`
- The report's URL slug (from the request itself, e.g. `cloudsecurityalliance.org/artifacts/{slug}`)

**✅ Validated:** Dry-run against ticket #153521 ("2026 State of Modern Application and AI Security" / Miggo Security survey report), using the exact slug and filters from that ticket's internal note. Query ran cleanly through the Grafana Builder UI on the first attempt (no Cloudflare block) and returned 120 current opt-in leads — up from the smaller set originally delivered on 2026-07-07, consistent with organic lead growth over the following three weeks.

---

## Step 1 — Get the report's slug

The request will usually include the report's public URL, e.g.:
`cloudsecurityalliance.org/artifacts/2026-state-of-modern-application-and-ai-security`

The slug is the last path segment: `2026-state-of-modern-application-and-ai-security`. This is used directly as `context_id` in the query below — no CMS lookup needed (unlike the download-count SOP, this table keys off the slug, not a numeric CMS ID).

## Step 2 — Open Grafana Explore

1. Go to `ops-dashboard.cloudsecurityalliance.org/explore`.
2. Confirm the datasource is **Main Site PostgreSQL** and the dataset is **csa_production**.
3. Confirm the query editor is in **Builder** mode, not **Code**.

## Step 3 — Build the query using the Builder UI only

Table: `personal_data_submissions`. Build the entire query below before clicking **Run Query** — see the Cloudflare WAF note further down.

1. **Table:** select `personal_data_submissions`.
2. **Filter**, three rows joined with AND:
   - `context` `==` `artifact request`
   - `context_id` `==` `{slug from Step 1}`
   - `opt_in_marketing_external` `==` `Yes` (this column renders as a Yes/No radio button in the Builder, since it's boolean)
3. For a quick **count only** (to sanity-check before pulling full data): set Data operations to `COUNT` on column `id`.
4. For the **full lead export** (what the requester actually needs), instead add these columns individually with no aggregation: `id`, `name_given`, `name_family`, `email`, `organization_name`, `organization_title`, `job_level`, `region`, `opt_in_marketing_external`, `created_at`.
5. Turn on **Order** and sort by `created_at` ascending, to match the delivery format used previously.
6. Click **Run Query** once.

## Step 4 — Export and deliver

- Use Grafana's table result to export/copy the rows into a CSV.
- Name the file consistently with prior tickets, e.g. `Ticket{#}_{Sponsor}_{Report-Slug}_Opt-in-Leads.csv`.
- Attach the CSV as a **private** ticket comment (this is PII — do not post publicly, and do not email it outside the ticket).
- Leave an internal note documenting the exact filters used (context, context_id, opt_in_marketing_external), so future tickets for the same or similar reports can reuse it directly, same as the precedent this SOP was built from.

## Step 5 — Sanity-check and report

- If a similar leads list has been pulled before for the same report, compare row counts — an increase is expected and fine; a decrease would indicate something's wrong with the filter or the data.
- Note the pull date, since this number changes over time as more leads opt in.

---

## ⚠️ Known issue: Cloudflare WAF blocks

Same underlying ops-dashboard/Grafana workflow as the publication-download SOP, and the same mitigation applies:
- Never use the Code/raw-SQL editor or navigate to a URL with SQL in the query string.
- Build the entire query (table, columns, all three filters, ordering) before clicking **Run Query** once — this dry run needed only a single execution and went through cleanly.
- If blocked: wait 30–60 seconds, navigate to a clean `ops-dashboard.cloudsecurityalliance.org/explore` URL with no parameters, confirm it loads normally, then rebuild and retry.

---

## Reference query (for context only — do not paste this into the Code editor)

```sql
SELECT id, name_given, name_family, email, organization_name, organization_title,
       job_level, region, opt_in_marketing_external, created_at
FROM personal_data_submissions
WHERE context = 'artifact request'
  AND context_id = '{slug}'
  AND opt_in_marketing_external = true
ORDER BY created_at
```
