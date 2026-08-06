# work-instruction-data-request

Work instructions (SOPs) for recurring IT Ops / Zendesk ticket types.

## Files

- **SOP_Publication_Download_Counts.md** — How to look up a publication's download count via Grafana Explore (`interactions` table), including a bot-traffic sanity check and a required cross-session confirmation step before reporting any number.
- **SOP_ThirdParty_OptIn_Leads.md** — How to pull the list of third-party opt-in leads for a specific report/publication (e.g. for a sponsor), via the `personal_data_submissions` table filtered by report slug. Output is PII — private ticket attachment only.
- **SOP_Sales_Report_No_Leads_Check.md** — How to check whether a missing/empty daily sales report really means "no leads that day," or whether leads exist but the export job failed. Also covers pulling the underlying records (PII) if needed.
- **csa-public-slack-invite-rotation.md** — How to rotate the public Slack invite link for the csa-public workspace and update the csaurl.org/csa-public-slack redirect, roughly every 2 weeks or before the current link's 30-day expiry.
