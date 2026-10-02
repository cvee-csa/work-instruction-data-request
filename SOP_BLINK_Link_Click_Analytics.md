# Work Instructions: Pulling Click Analytics for a csaurl.org Short Link (BL.INK)

**Purpose:** Answer requests like "Can we get the analytics for [csaurl.org short link]?" — e.g., a department wants to know how much traffic a short link gets, where it comes from, and whether it's growing. Also covers what to do when BL.INK warns that a link is nearing its click-tracking limit.

**Audience:** IT Ops / Zendesk ticket handlers with access to BL.INK (csaurl.org domain).

**Tools needed:**
- BL.INK — `app.bl.ink`, with **csaurl.org** as the active domain (see Note 1)
- Python 3 with `pandas` and `openpyxl` (for summarizing the export — local machine or Claude)
- The short link's slug (from the request, e.g. `csaurl.org/csa-public-slack` → `csa-public-slack`)

**✅ Validated:** Run end-to-end against ticket **#159331** (`csaurl.org/csa-public-slack`, requested by Research via Marina Bregkou) on 2026-09-30. The BL.INK dashboard showed 13,894 total / 6,590 unique clicks all-time and 1,267 clicks in the last 30 days. The one-year export returned 7,835 click rows (2025-09-30 → 2026-09-30, 3,128 distinct IPs); its September 2026 total (1,266) matched the dashboard's last-30-days figure (1,267) to within one click, confirming the export is complete and consistent with the dashboard.

---

## Step 1 — Find the link in BL.INK

1. Go to `app.bl.ink` and confirm the domain selector (top left) reads **csaurl.org**. If not, switch it (see Note 1).
2. Type the slug into the **search** box (top right) and press Enter.
3. **Change the Link Owner filter from your own name to "All Users"** and click **Apply Filters** (funnel icon). The search defaults to links *you* own — if someone else created the link (e.g., `csa-public-slack` is owned by Kurt Seifried), it will show "0 of 0 Links" until you do this.
4. Click the link name. The detail page URL is `app.bl.ink/manage/blinks/{LINK_ID}/display` — note the ID (e.g. `930161` for `csa-public-slack`).

## Step 2 — Record the dashboard totals

The **Overview** tab under Click Analytics shows a small table. Record all of it — it's the only source for data older than 12 months:

| Row | Unique | Total |
|---|---|---|
| All (since link created) | | |
| Last 24 Hours | | |
| Last 7 Days | | |
| Last 30 Days | | |

Also note the **Created on** date (right-hand panel). "Unique" is BL.INK's de-duplicated count; "Total" counts every click.

## Step 3 — Export the click-level data

1. On the link's detail page, click the small **download icon** to the right of the Overview / Device / Referrers / Location tabs. Hovering shows the tooltip **"Export Last 1 Year of Clicks."**
2. The CSV downloads silently — **there is no on-page confirmation**. Check the browser's Downloads for a file named `clicks_{slug}.csv`.
3. The file contains one row per click, with columns: `Click Date, Mobile Device, Referrer, IP, IP Host, User Agent, Country, Region, City`.

**This is the richest source available — use it rather than the options below:**
- The **Device / Referrers / Location** tabs are charts only (no numbers in the page text) and cover the **last 30 days** only.
- **Links → Export** (left sidebar) exports lists of *links*, not clicks — at most daily counts for the last 90 days.
- **Analytics → Overview** has a Date Range picker, but it would not accept a start date earlier than the default (neither typed input nor form entry stuck). Don't spend time fighting it.
- Click-level data **older than 12 months cannot be exported** from the UI. For older periods, only the all-time totals from Step 2 are available (BL.INK support or API would be the only route to more).

## Step 4 — Treat the raw export as PII

The export contains **IP addresses, full user-agent strings, and city-level location**. Do **not** forward the raw CSV to the requester or attach it to a public ticket reply. Everything shared outside IT Ops should be aggregated counts only (Step 5). If the raw file must be kept, attach it as a private/internal note only.

## Step 5 — Summarize the export

Build these aggregates (the reference script at the bottom does all of them):

- **Monthly:** total clicks and distinct IPs per month (plus a line chart).
- **Daily:** total clicks and distinct IPs per day — useful for spotting spikes.
- **Referrers:** clicks by referring domain. Blank referrer = "(direct / none)" — typically email clients, chat apps, bookmarks. Relabel `android-app://com.google.android.gm/` as "Gmail app (Android)" and `android-app://com.slack/` as "Slack app (Android)". Drop or anonymize obviously unrelated/spam referrers.
- **Countries:** clicks, distinct IPs, and share of clicks by ISO country code.
- **Devices:** use `Mobile Device` when filled (iphone/android/ipad); otherwise fall back to the user agent (`android` → Android, `iphone`/`ipad` → iOS), else "Desktop / other".
- **Summary tab:** 12-month totals, distinct IPs, clicks per visitor, countries represented, top-country share, Google-search share, no-referrer share, mobile share, busiest month, plus the all-time dashboard totals from Step 2.

Deliver as an `.xlsx` (use formulas for totals/shares so it recalculates) with **no IP, user-agent, or city columns**.

## Step 6 — Sanity-check before reporting (required)

1. **Completeness check:** the export's total for the most recent ~30 days should match the dashboard's "Last 30 Days → Total" from Step 2 (±a few clicks for timing). A large gap means the export is truncated or you exported the wrong link — re-export before going further.
2. **Repeat-IP concentration:** count IPs with 20+ clicks and the share of clicks they account for. In the #159331 run, 76 IPs accounted for ~2,474 of 7,835 clicks (~32%) — consistent with shared corporate networks and email link-scanners, not a single bot. If *one* IP accounts for a large share, flag it as likely automated before reporting.
3. **Bot user agents:** check for `bot|crawl|spider|curl|python|headless` etc. in the user agent. (Only 2 of 7,835 in the validated run — BL.INK appears to filter most bots already.)
4. **Spikes:** for the top few days, check whether traffic is spread across many IPs and mostly direct/email referrers (consistent with a newsletter or event promotion) vs. concentrated in a few IPs.

## Step 7 — Report with these caveats

Always state, in the report or the reply:
- **Clicks are not conversions.** For a redirect like `csa-public-slack`, BL.INK cannot tell whether the person actually joined — only that they clicked. (Slack's own workspace analytics would be needed for joins.)
- **"Unique visitors" = distinct IP addresses** — an approximation. Shared networks undercount people; people on multiple networks overcount.
- **Detailed data covers the last 12 months only;** earlier periods are all-time totals from the dashboard.
- **No personal data is included** in the shared summary.

---

## ⚠️ Known issue: BL.INK click-tracking cap

BL.INK warns when a link approaches its plan's click-tracking limit (15,000 clicks for `csa-public-slack` as of 2026-09). Per BL.INK: **the link keeps redirecting** after the cap — only analytics stop (clicks are no longer tracked, processed, or stored). Options BL.INK offers: do nothing (lose tracking), reset the click count (restarts tracking but **clears history**), buy wildcard link tokens (unlimited tracking on specific links, sold in bundles of ten), or upgrade the plan tier.

**Before anyone resets the count or the cap is hit, run Steps 2–3 to export the history** — a reset or cap makes it unrecoverable. Estimate remaining headroom from the *last 30 days* rate, not a single week: in #159331, one week showed 81 clicks but the last 30 days showed 1,267, which cut the estimate from ~4 months to under a month.

## Note 1 — bl.ink domain access

bl.ink organizes links under separate domain workspaces (csachapter.io, csaqr.org, csaurl.org). If a direct `app.bl.ink/manage/blinks/{ID}/...` URL returns "Access Denied," the active domain (top-left selector) is set to something other than csaurl.org — switch it and retry. (Same as the note in `csa-public-slack-invite-rotation.md`.)

---

## Reference script (summarize a BL.INK click export)

```python
import pandas as pd, re

SRC = "clicks_csa-public-slack.csv"   # BL.INK export
d = pd.read_csv(SRC, skipinitialspace=True, dtype=str).fillna("")
d.columns = [c.strip() for c in d.columns]
d["dt"] = pd.to_datetime(d["Click Date"].str.strip(), format="%Y-%m-%d %I:%M:%S%p")

def ref_domain(r):
    if not r: return "(direct / none)"
    m = re.match(r"https?://([^/]+)", r)
    return (m.group(1) if m else r).removeprefix("www.")

def device(row):
    ua = row["User Agent"].lower()
    if row["Mobile Device"]: return row["Mobile Device"]
    if "android" in ua: return "android"
    if "iphone" in ua or "ipad" in ua: return "ios"
    return "desktop/other"

d["ref"] = d["Referrer"].map(ref_domain)
d["dev"] = d.apply(device, axis=1)

monthly  = d.groupby(d.dt.dt.to_period("M")).agg(clicks=("IP","size"), unique_ips=("IP","nunique"))
daily    = d.groupby(d.dt.dt.date).agg(clicks=("IP","size"), unique_ips=("IP","nunique"))
refs     = d["ref"].value_counts()
countries= d["Country"].replace("", "(unknown)").value_counts()
devices  = d["dev"].value_counts()

# Sanity checks (Step 6)
ip_counts = d["IP"].value_counts()
print("rows", len(d), "distinct IPs", d.IP.nunique(), d.dt.min(), "->", d.dt.max())
print("IPs with 20+ clicks:", (ip_counts >= 20).sum(), "clicks from them:", ip_counts[ip_counts >= 20].sum())
print("last 30 days:", (d.dt >= d.dt.max() - pd.Timedelta(days=30)).sum(), "(compare to dashboard)")
bot = d["User Agent"].str.lower().str.contains("bot|crawl|spider|curl|python|headless|wget")
print("bot-like user agents:", bot.sum())
```

Write the aggregates to an `.xlsx` (one tab each + a Summary tab) — never include the `IP`, `IP Host`, `User Agent`, `City`, or `Region` columns in anything shared.
