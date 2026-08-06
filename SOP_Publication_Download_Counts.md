# Work Instructions: Looking Up Publication Download Counts

**Purpose:** Answer requests like "How many downloads does [publication] have?" using the Main Site PostgreSQL data behind the ops dashboard.

**Audience:** IT Ops / Zendesk ticket handlers with access to the CSA Admin CMS and the ops dashboard (Grafana).

**Tools needed:**
- CSA Admin CMS — `cloudsecurityalliance.org/csa-admin/publication`
- Ops dashboard (Grafana) — `ops-dashboard.cloudsecurityalliance.org`

**✅ Validated:** This procedure was dry-run end-to-end against "Disaster Recovery as a Service" (CMS ID 624) and cross-checked against a known-good figure from a previously answered ticket (#153721, confirmed 4,128 downloads on 2026-07-15). Following these exact steps returned 4,157 (2,342 under `Artifact` + 1,815 under `Publication`) on 2026-07-29 — a 0.7% difference consistent with two weeks of organic growth. This also confirmed the "sum across types" step is not an edge case — for this publication the count was genuinely split across both `Artifact` and `Publication` rows, and using only one type would have been badly wrong.

**✅ Second validation (anomalous-count case):** Re-run against the "Hugging Face Incident Initial Post-Mortem" (CMS ID 2572) on 2026-07-30 to answer "how many downloads today?" The count had jumped from 26,669 (2026-07-29) to 33,199 — a 24% one-day increase, unusually large for a 3-day-old document. Re-running the identical query twice on separate clean (non-blocked) sessions returned 33,173 and then 33,199 minutes apart — small, consistent, incremental growth, confirming the number was real and not a query fluke. A follow-up sanity check (see Step 5 below) confirmed the growth was organic, not bot traffic. Lesson: a big or surprising number is not, by itself, a reason to distrust the query — but it is a reason to run Step 5.

---

## Step 1 — Find the publication's CMS ID

Downloads are tracked against a CMS record ID, not the public URL slug, so this has to be looked up first.

1. Go to `cloudsecurityalliance.org/csa-admin/publication`.
2. Type the publication name into the **Filter** text box near the top of the list and click **Refresh** — this reliably narrows the list to matching titles (confirmed working; faster than scrolling the full list).
3. Open the record. The CMS ID is in the edit URL: `/csa-admin/publication/{ID}/edit`.
4. Note the **Status** and **Created** date while you're there — useful context if the download number looks unusual (e.g., a just-published document with a huge count is worth a sanity check).

Do **not** try to find the publication in the `articles` Postgres table — that table only holds Press Releases and Blog posts, not Publications/Artifacts/Whitepapers.

## Step 2 — Open Grafana Explore

1. Go to `ops-dashboard.cloudsecurityalliance.org/explore`.
2. Confirm the datasource is **Main Site PostgreSQL** and the dataset is **csa_production**.
3. Make sure the query editor is in **Builder** mode (top right toggle), not **Code**. See the warning below — this matters.

## Step 3 — Build the query using the Builder UI only

Do this entirely with point-and-click controls. Do not type raw SQL into the Code editor, and do not navigate to a URL containing SQL text (see warning below).

1. **Table:** select `interactions`.
2. **Data operations / Column:** add `COUNT` on column `id`.
3. Click **+** under Data operations to add a second column, plain (no aggregation): `interactable_type`. This lets you confirm which type the rows fall under (usually `Artifact` or `Publication` depending on content type) instead of guessing.
4. Turn on **Filter** and add three rows (all joined with AND):
   - `interactable_id` `==` `{CMS ID from Step 1}`
   - `controller` `Is empty`
   - `action` `Is empty`
   (The controller/action = empty condition excludes RailsAdmin edit-log noise — rows created by internal edits rather than actual downloads.)
5. Turn on **Group by column** and set it to `interactable_type`.
6. Build out every step above **before** clicking Run Query — see the throttling note below on why this matters.
7. Click **Run Query** once.

Expected result: a small table with one row per `interactable_type`, each with a `count`. **Do not assume only one type will appear** — in testing, one publication returned a single `Publication` row while another returned both `Artifact` and `Publication` rows that needed to be added together for the true total. Always sum every row returned.

## Step 4 — Sanity-check and report

- Compare the count to the publication's age and to comparable publications if available. A brand-new document with an outsized count, or a years-old document with a near-zero count, is worth a second look before it goes in a ticket reply.
- If the number looks surprising (unusually high, or a large jump since the last time it was checked), don't stop here — run Step 5 before reporting it as fact.
- Before reporting any number, complete Step 6 (confirmation) below — it's required, not optional, even when the number doesn't look surprising.

## Step 5 — Optional: verify a surprising number isn't bot/scripted traffic

If a count looks anomalous (e.g., a big single-day jump, or a total that's high relative to the document's age), check whether the growth is spread across many visitors or concentrated in a few sources before trusting it.

1. In the same query, change the second column and the **Group by column** from `interactable_type` to `ip_address` (keep the same three filters: `interactable_id`, `controller Is empty`, `action Is empty`).
2. Turn on **Order** and sort by `COUNT(id)` **descending**, so the busiest IPs surface first.
3. Run once (per the throttling rules below).
4. Read the top rows: if the #1 IP accounts for a large share of the total (e.g., thousands of the total downloads while everything else is near-zero), that's a red flag for a bot, crawler, or internal test script — flag it before reporting the number. If instead the top IPs each account for a small, steadily-declining share (a long-tail distribution, e.g., a handful of IPs in the 50-200 range out of a total in the tens of thousands), that's consistent with organic traffic, often reflecting corporate NAT/VPN gateways shared by many real readers rather than any single source.

This does **not** tell you *why* traffic is high (referral source, campaign, etc.) — the `interactions` table has no referrer data — only whether it's plausible.

## Step 6 — Confirm the number before reporting (required)

Neither Step 4 nor Step 5 actually verifies that the count is *correct* — Step 4 is a plausibility check and Step 5 rules out bot traffic. Before any download count goes into a ticket reply or report, confirm it two ways:

1. **Re-run in a second clean session.** Wait out the WAF cool-down (see below), navigate to a fresh Explore URL, rebuild the query from scratch, and run it once. The two totals should either match exactly or differ only by an amount consistent with organic growth over the elapsed time (minutes-to-hours of typical daily volume for that publication). A large or unexplained difference between the two runs means one of them is wrong — often a WAF block silently returning bad data (see the silent-block trap below) — and should be treated as unresolved, not averaged or waved through.
2. **Cross-check against an independent source, if one exists.** In order of reliability:
   - A previously reported figure for the same publication from an earlier ticket or report, adjusted for elapsed time (see the "Second validation" note above for a worked example).
   - Any count surfaced elsewhere for the same publication — e.g., a views/downloads field on the CMS admin edit page, if the CMS displays one.
   - Google Analytics or another analytics tool, if accessible, for the same publication and date range. GA and the `interactions` table may use different definitions (unique visitors vs. total download events), so expect the numbers to land in the same ballpark rather than match exactly.

   If no independent source is available, say so explicitly when reporting the number (e.g., "confirmed via two independent Grafana queries; no external source was available for cross-check") instead of silently treating the double-run alone as full confirmation.

Only report the number once both checks pass, or once you've explicitly noted that an independent source wasn't available. A number that hasn't gone through this step is a draft, not a fact to report.

---

## ⚠️ Known issue: Cloudflare WAF blocks

The ops-dashboard domain sits behind Cloudflare, and its WAF will intermittently block requests tied to this workflow. Two triggers have been confirmed:

1. **Navigating directly to a URL with raw SQL in the query string** (e.g., pasting a full Explore URL with `rawSql=...` into the address bar, or using the Code editor and running it). This reliably triggers a hard block ("Sorry, you have been blocked").
2. **Running several queries back-to-back in quick succession**, even through the Builder UI. This has triggered blocks even when the query itself was clean and had run successfully moments before — it appears to be a rate/session-based trigger, not purely content-based. In practice, even *one* extra run after a successful query — for example, re-running the same query a few minutes later to double-check a number — is enough to trigger it. Treat every re-run as its own attempt requiring the full avoid/recover cycle below, not as a quick follow-up.

**⚠️ Silent-block trap:** A block does not always show an obvious error. Sometimes Explore just renders **"No data"** in the results panel with no visible warning, indistinguishable at a glance from a real empty result. If a count comes back as "No data" or zero when you expected real numbers, don't take it at face value — open **Query inspector → Refresh** (or the **Error** tab if one appears) and check whether the response is actually a Cloudflare "Attention Required" HTML page rather than a real query result. If so, treat it as a block, not as a genuine finding, and go through the recovery steps below.

**How to avoid it:**
- Always use the Builder UI (point-and-click), never the Code/raw-SQL editor, and never navigate to a URL with SQL embedded in it.
- Build the *entire* query (all columns, filters, and grouping) before clicking **Run Query**, so you only need one execution per attempt.

**How to recover if blocked:**
- Wait roughly 30–60 seconds (in practice, closer to 60-90 seconds has been more reliable — shorter waits have still resulted in an immediate re-block).
- Navigate to a clean URL with no query parameters: `ops-dashboard.cloudsecurityalliance.org/explore`.
- Confirm the block has cleared (page loads normally, no Cloudflare "Attention Required" page) before rebuilding and re-running the query.
- If it blocks again immediately, wait longer and retry — avoid repeated rapid attempts, which seems to make it worse.

---

## Reference query (for context only — do not paste this into the Code editor)

```sql
SELECT COUNT(id), interactable_type
FROM interactions
WHERE interactable_id = {CMS_ID}
  AND controller IS NULL
  AND action IS NULL
GROUP BY interactable_type
```

Reference query for Step 5 (IP-concentration sanity check):

```sql
SELECT COUNT(id), ip_address
FROM interactions
WHERE interactable_id = {CMS_ID}
  AND controller IS NULL
  AND action IS NULL
GROUP BY ip_address
ORDER BY COUNT(id) DESC
```
