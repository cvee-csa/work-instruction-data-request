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
