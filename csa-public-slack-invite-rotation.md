# Work Instructions: Rotate the csa-public Slack Invite Link

*Browser-executable version, verified live in Chrome (Claude in Chrome) on August 6, 2026. Version 2.1 — supersedes the June 2022 doc "Update csa-public Slack public invite link."*

---

## Purpose

Slack public invite links expire after 30 days by default (or after 400 uses); that expiry can also be extended or set to never expire. Cloud Security Alliance publishes a stable short link, csaurl.org/csa-public-slack, that always points at the current Slack invite link. Because the underlying Slack link expires (by design — see Note 2), it needs to be rotated periodically — roughly every 2 weeks as practice, and always before the current link's own 30-day expiration date.

## Preconditions

- Logged into Slack in the browser as a user with admin rights on the csa-public workspace.
- Logged into bl.ink in the browser, with access to the **csaurl.org** branded domain — not just csachapter.io. See Note 1 below.
- Chrome, driven via Claude in Chrome (or performed manually using the same steps).

## Part A — Deactivate the current Slack invite link and create a new one

1. Go to `https://csa-public.slack.com/admin/invites` and click the **"Invite Links"** tab. This tab lists every active/expired invite link by creator, with join counts and expiration dates — it replaces the old single-link view.
2. Find the row for your own account (it will show your name as "Link creator"). Click **Deactivate** next to it, then confirm **"Deactivate Link"** in the dialog that appears. This is the row that csaurl.org currently points to, so deactivating it breaks the old public link immediately — that's expected, because Part B replaces it right away.
3. Click **"Invite People"** (top-right of the Invitations page). In the dialog, click the small chevron next to **"Copy Invite Link"**, then click **Copy Invite Link** itself. This generates a brand-new invite link for the workspace and copies it to your clipboard. A toast appears confirming "Link copied — expires in 30 days" — that's the setting we want, so nothing further to change here.
4. Optional confirmation: click the chevron again and choose **"Edit settings"** (labeled "Edit settings," not "Edit link settings", in the current UI). In the "Invitation settings" dialog, confirm the **"Invite set to expire after…"** dropdown reads **"30 days."** This is the current Slack default for a freshly generated link, so no change is needed — just verify it wasn't left on "Never expires" or another value from a prior rotation, then click **Save** (or Cancel/Back if no change was made).
5. Go to the **"Invite Links"** tab and find the new row under your name (it will show 0 joiners and today's creation date). Click the invite code text itself (it looks like `zt-XXXXXXXXX-XXXXXXXXXXXXXXXXXXXXXX`) to open it, or copy its link target directly — this is the full join URL, e.g. `https://join.slack.com/t/csa-public/shared_invite/zt-XXXXXXXXX-XXXXXXXXXXXXXXXXXXXXXX`. Note the exact URL; it's needed in Part B. (The dialog's own "Copy Invite Link" button copies to the OS clipboard, which is not reliably readable back by browser-automation tools — reading the link straight off the Invite Links table avoids that problem.)

## Part B — Point the csaurl.org short link at the new invite link

6. Go to `https://app.bl.ink/manage/blinks/930161/edit`. If you land on an "Access Denied" page, your bl.ink account is defaulted to a different domain workspace (e.g. csachapter.io). Click the domain selector at the top left and switch to **csaurl.org**, then reload the edit URL.
7. In the top field (the destination URL), select all existing text and replace it with the new Slack invite URL copied in Part A, step 5. **Caution:** some automated fill methods (select-all + type) have been observed to insert the new text without deleting the old text first, producing a broken concatenated URL. After entering the new URL, re-read the field's actual contents (not just what's visually shown) to confirm it contains only the new URL before saving.
8. Confirm the short link itself is unchanged: `csaurl.org / csa-public-slack`.
9. Confirm the redirect-type dropdown on the right is set to **"Temporary Redirect (307)"** — this was already correctly set in the current record, but verify it on every rotation so the link stays easy to update again later.
10. Click **Update**. A blue "Link Updated!" banner confirms the save, and the link's detail view will show the new destination URL and "Modified on" timestamp.

## Part C — Verify

11. Open `https://csaurl.org/csa-public-slack` in a fresh tab. It should redirect through `join.slack.com` to the new invite — if you are already a workspace member, Slack will skip the join screen and go straight to "Launching csa-public," which is itself confirmation the redirect chain resolved correctly. If you are not a member, you should land on the normal Slack invite/join page instead of an error page.

## Cadence

Rotate this link every 2 weeks as a matter of practice (per the original guidance), and always before the current link's 30-day expiration date. As of this rotation (August 6, 2026), the live link expires September 5, 2026 — the next rotation should happen well before that date.

## Notes — what changed since the original (2022) instructions

- The Invitations page now has three tabs — Pending, Accepted, and **Invite Links** — instead of a single link. The Invite Links tab shows every link ever created for the workspace, by creator, with per-link Extend/Deactivate/Renew controls.
- The per-link settings menu item is now labeled "Edit settings," not "Edit link settings."
- Slack's own copy now states each link/QR code works for up to 400 people (matches the original doc) and defaults new links to 30 days (the dialog showed "15 days" on the pre-rotation link and "30 days" on the freshly generated one — Slack seems to default to 30 days for a brand-new link).
- The "Notify me whenever someone joins using the link or QR code" checkbox exists and defaults to checked on new links; the deactivated link had it unchecked. Decide per your own notification preferences.
- Clicking "Invite People" → "Copy Invite Link" manages **your own** personal invite link (tied to the logged-in admin account), not a workspace-wide link independent of any user. Whoever runs this rotation becomes the new "Link creator" shown in the Invite Links table.
- bl.ink now organizes links under separate domain workspaces (csachapter.io, csaqr.org, csaurl.org, plus the raw b.link account); the direct blinks/930161/edit URL only works once csaurl.org is the active domain in the top-left switcher.
- The redirect type on the existing csaurl.org/csa-public-slack link was already "Temporary Redirect (307)" as instructed — no change needed there this rotation.

## Note 1 — bl.ink domain access

If `https://app.bl.ink/manage/blinks/930161/edit` returns "Access Denied," it is almost always because the account's active domain (shown top-left) is set to something other than csaurl.org. Switch domains from that dropdown, then retry the edit URL — no separate permission grant is needed if your bl.ink account already has csaurl.org listed under "My Domains."

## Note 2 — why we keep the 30-day expiry (not "Never expires")

New Slack invite links default to a 30-day expiration, and that's the setting this SOP keeps — matching the original 2022 guidance, which called for updating the link every few weeks specifically because it expires in 30 days. An earlier draft of this rotation briefly set the link to "Never expires" for security-hygiene reasons, but that broke the tie between "link rotated" and "link still valid" that the original process relied on, so it was reverted back to the 30-day default on this pass. If a future rotation is missed, the link now fails safe (expires) rather than staying live indefinitely.
