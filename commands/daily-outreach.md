---
name: daily-outreach
description: Run the daily Tribed IG + LinkedIn pipeline — handle replies, advance LinkedIn sequences through the LinkedIn MCP, and ICP-filter + personalize the day's new leads.
---

Run the Tribed daily outreach pipeline. Follow references/linkedin-ops.md (Job C) exactly. Preflight the LinkedIn session (`get_my_profile`; abort and tell the user on auth errors), then pull the day's due work with `list_outreach_leads({ channel: "li", dueBefore: today, activeOnly: true })`. Order matters: replies first, then advancing existing leads, then new invites. Always set `externalId` (the LinkedIn publicIdentifier) on upserts and schedule everything forward with `nextActionAt`.

1. **Replies.** `get_inbox` (limit 50), match conversations to open LinkedIn leads in the tracker. Any reply: stop that lead's sequence, log the touch, and draft a Mode 2 reply for the user. Replies are never auto-sent.
   - Interested reply with no demo yet: build the demo first (Job D). Read the thread + profile + their site, then create the community with the Tribed MCP (`get_onboarding_guide` FIRST, then `onboard_community` + content tools), themed as their brand. Save `data.demo_handle` / `data.demo_url` on the lead, then draft the reply. Max 5 demos per run, never a duplicate.
2. **Acceptances.** Leads awaiting acceptance that are now 1st-degree: send `data.li_followup_1` with `send_message`, log the touch, set the next action.
3. **Due follow-up 2.** Follow-up 1 sent 3-4+ days ago with no reply: send `data.li_followup_2` and log it. Archive leads 14 days quiet after follow-up 2.
4. **New leads** (IG-first discovery + anything queued with "Send connection request"):
   - ICP filter (Job B): enrich via Apify `harvestapi/linkedin-profile-scraper` and keep only `followerCount >= 2000` OR `creator` OR `influencer`.
   - Personalize the survivors (Job A) in the Tribed voice, anchored on one real detail (rule zero): connection note (<280 chars, no CTA, no price), follow-up 1 (light bump), follow-up 2 (new angle / soft close). Save to the tracker `data` bag.
   - Send connection requests with `connect_with_person` within today's invite budget.
5. **Instagram.** Draft DM + two follow-ups onto the tracker for the VA (unchanged; the VA sends by hand).
6. **Report** per channel: replies, follow-ups sent, invites sent, kept vs removed by ICP, and held-for-review.

Hard limits: max 15 connection requests and 25 messages per day, one run per day. Do not enroll out-of-ICP leads. Do not send a message that fails rule zero — hold it for manual review. If LinkedIn throws a checkpoint, captcha, or verification wall, stop immediately and tell the user.
