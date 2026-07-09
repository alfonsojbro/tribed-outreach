# Mode 7 — LinkedIn ops: personalization + daily pipeline

LinkedIn outreach runs on the self-hosted **LinkedIn MCP** (`linkedin` server, stickerdaniel/linkedin-mcp-server) driving Alfonso's real LinkedIn session, with the **Tribed outreach tracker** as the record of state. This replaced Gojiberry. There is no campaign engine anymore: the daily run IS the campaign engine. Requires the `linkedin` MCP (actions), the Tribed MCP (outreach lead tools + community onboarding for demo creation, Job D), and the Apify MCP (follower enrichment).

## How the stack is wired (know this before touching anything)

- **The `linkedin` MCP does the actions.** It automates a logged-in browser session, so calls run sequentially through one browser and long runs are normal. The tools that matter here:
  - `connect_with_person({ linkedin_username, note })` — connection request; the `note` is where the connection note goes.
  - `send_message({ linkedin_username, message, confirm_send: true, profile_urn? })` — DM. Only works once they are a 1st-degree connection. Pass `profile_urn` (from `get_person_profile`) when you have it; it is more reliable than the username lookup.
  - `get_inbox({ limit })`, `get_conversation({ linkedin_username | thread_id })`, `search_conversations({ keywords })` — read the inbox and detect replies.
  - `get_person_profile({ linkedin_username, sections? })` — profile scrape (anchor material, connection degree, URN).
  - `search_people({ keywords, location?, network? })` — prospecting; `network: ["F"]` filters to 1st-degree (useful to confirm acceptance in bulk).
- **The Tribed tracker holds the state.** Every LinkedIn lead is a record in the outreach tracker (`upsert_outreach_leads`, accountId `"digital_university"`, `channel: "li"`). The drafted copy lives in the lead's `data` bag: `data.li_note`, `data.li_followup_1`, `data.li_followup_2`, plus `data.li_username` (the publicIdentifier) and `data.li_urn` when known. `nextAction` says where the lead is in the sequence, and every send is recorded with `log_outreach_touch`. The dashboard (admin.tribed.io) is the source of truth.
- **The sequence map** (what replaced the Gojiberry steps):

  | Old Gojiberry step | Now |
  |---|---|
  | 0 — invitation + note | `connect_with_person` with `note` |
  | 2 — 1st message after connect (follow-up 1) | `send_message` on the first daily run after they accept |
  | 5 — 2nd message (follow-up 2) | `send_message` 3-4 days after follow-up 1 if no reply |
  | 1 / 4 / 6 — emails | Instantly (Mode 6). The LinkedIn side sends no email. |
  | 3 — profile visit | dropped |

- **Nothing sends itself.** Gojiberry fired steps on its own schedule; now a message goes out only when the daily run (or the user) sends it. A missed daily run only delays touches. It never double-sends, because you check the lead's touch log before sending and log every send after.
- **Lead sourcing** (Gojiberry's agents are gone): leads come from the IG-first combined pipeline below, from the user handing over profiles, or from `search_people` prospecting when asked. Everything gets the same ICP gate.

Session plumbing: the server is registered as `linkedin` in NobleAdmin/.mcp.json and keeps its session in `~/.linkedin-mcp/`. If tools start failing with login or session errors, re-authenticate in a terminal with `uvx mcp-server-linkedin@latest --import-from-browser` (imports from the signed-in browser) or `--login`, then retry.

## Job A — write personalized messages onto leads

For each in-ICP lead:

1. Gather the anchor. Best source is the scraped LinkedIn profile via `get_person_profile` (`headline`, `about`, a named program/book, current role; add `sections: "posts"` when the main page gives nothing). Apply **rule zero** — one real detail, never the niche.
2. Write, in the Tribed voice:
   - **Connection note.** Mode 4A rules: under ~280 characters (confirm the count), one line on their detail, one line hinting something was built for them, no price, no features, no explicit CTA.
   - **Follow-up 1 (light bump).** Mode 3 Touch 1: one or two sentences, the thing exists and is theirs, door open. One soft question.
   - **Follow-up 2 (new angle / soft close).** Mode 3 Touch 2 or 3: a fresh reframe or the takeaway, lightest CTA or none.
   - (Optional) **emails** — Mode 6 (email.md), pushed to Instantly, only if email is enabled for the run.
3. Rotate skeletons and CTAs across leads (see core). Never reuse a line across contacts.
4. Save it to the tracker: `upsert_outreach_leads` with `channel: "li"`, the messages in `data.li_note` / `data.li_followup_1` / `data.li_followup_2`, `data.li_username`, and `nextAction: "Send connection request"`.
5. Verify on the first lead of a batch by re-reading with `get_outreach_lead` before doing the rest.

Always sample 3–5 leads for the user's approval before writing a whole list.

## Job B — the ICP filter (run before personalizing new leads)

Filter on real audience signal, not role or industry fit:

1. Collect the batch's `li_username`s (publicIdentifiers).
2. Enrich followers with Apify actor `harvestapi/linkedin-profile-scraper`, mode `"Profile details no email ($4 per 1k)"`, passing the publicIdentifiers. It returns `followerCount`, `connectionsCount`, `creator`, `influencer`. Cost ≈ $0.004/lead.
3. **Keep** a lead if `followerCount >= 2000` OR `creator === true` OR `influencer === true`. Everyone else is out of ICP.
4. Out-of-ICP leads are simply not enrolled. If one is already on the tracker, take it out of the pipeline with `update_outreach_lead`: clear the `nextAction` and note why in `data` (e.g. `data.icp: "out - 800 followers"`).

## Job C — the daily pipeline

Run once per day. Order matters: replies first, then advancing existing leads, then new invites.

1. **Replies.** `get_inbox({ limit: 50 })` and match conversation names against open LinkedIn leads in the tracker. Any reply: stop that lead's sequence (`nextAction: "Handle reply (Mode 2)"`), `log_outreach_touch`, read the thread with `get_conversation`, and draft a Mode 2 reply for the user. Replies are never auto-sent. If the reply shows interest and the lead has no demo yet, build the demo first (Job D) so the Mode 2 draft has something real behind it.
2. **Acceptances → follow-up 1.** For leads with `nextAction: "Awaiting acceptance"`: check degree with `get_person_profile` (or one `search_people` name query with `network: ["F"]`). Accepted: send `data.li_followup_1` via `send_message` (with `profile_urn` if stored), log the touch, set `nextAction: "Follow-up 2 in 3-4 days"`. If `send_message` fails because they are not messageable, they have not accepted; leave them for the next run.
3. **Due follow-up 2.** Follow-up 1 sent 3-4+ days ago (check the touch log) and no reply: send `data.li_followup_2`, log it, `nextAction: "Awaiting reply"`. No reply 14 days after follow-up 2: archive the lead.
4. **New leads.** The day's new prospects (from the IG-first pipeline or queued on the tracker): ICP filter (Job B), personalize the survivors (Job A), then `connect_with_person` with the note for as many as today's invite budget allows. Log each touch, set `nextAction: "Awaiting acceptance"`. The rest keep `nextAction: "Send connection request"` for tomorrow.
5. **Report:** replies found, acceptances messaged, follow-up 2s sent, new invites sent, kept vs removed by ICP, and leads held for manual review (no usable anchor). Per channel.

### Volume + account safety (hard limits)

This is browser automation on Alfonso's real LinkedIn account, which LinkedIn's terms prohibit. Volume discipline is what keeps the account alive:

- Max **15 connection requests** and **25 messages** per day. One daily run.
- Calls queue through one browser session; never try to parallelize, and expect long runs.
- If a tool errors with anything that smells like a checkpoint, captcha, or verification wall, STOP the run immediately and tell the user. Do not retry.
- Sends happen only inside the daily run or on an explicit ask.

### Channel split — LinkedIn vs Instagram

- **LinkedIn → tracker + `linkedin` MCP.** Jobs A–C above; Claude sends during the daily run.
- **Instagram → tracker for the VA.** Unchanged: IG DMs are never auto-sent. IG leads live in the same tracker (`channel: "ig"`, accountId per the pipeline below), the VA sends by hand. For each IG lead:
  1. Anchor comes from the profile/feed (Mode 1 / instagram.md), not scraped LinkedIn fields. IG has no connection-note step — start at the DM.
  2. Draft in the Tribed voice: a cold DM plus two follow-ups (Mode 3). Write them into the lead's `data` bag, e.g. `data.ig_dm`, `data.ig_followup_1`, `data.ig_followup_2`.
  3. Set `channel: "ig"`, `owner` to the VA handling IG (e.g. "Martin"), a clear `nextAction` ("Send IG DM"), and `stage: "top"` so it surfaces on the VA's list. Upsert with `upsert_outreach_leads`.
  4. If a lead has no usable anchor, leave the message fields empty and flag it — don't write a generic DM.

### Combined cross-channel daily pipeline (IG-first)

The daily job targets one person across all three channels. Start on Instagram and fan out:

1. **Discover on Instagram** (WebSearch + Apify `figue/instagram-profile-scraper`): qualified coaches 10k-60k followers with a bio link. Capture `external_url` (bio link), `business_email`, bio, captions.
2. **IG copy → the VA.** Draft a gift-first DM + two follow-ups (Mode 1 + Mode 3). Upsert to the Tribed pipeline with `accountId "digital_university"` (this is the record; NOT "tribed"), `channel "ig"`, `owner "Martin"`, messages in `data`. IG is never auto-sent — the VA sends by hand.
3. **High-confidence match to LinkedIn.** The reliable bridge is the **IG bio link**: if it's a `linkedin.com/in/` URL use it directly; if it's a personal site/Linktree, fetch it and read the LinkedIn link off it; else search by exact name + a corroborating signal (same company / same website domain). If nothing confidently matches, keep the lead **IG-only** — never guess, a wrong match messages the wrong person.
4. **Matched → LinkedIn sequence.** ICP-gate (2,000+ or creator), personalize (Job A: note + two follow-ups into `data`), and send the connection request via `connect_with_person` if today's invite budget allows — otherwise leave `nextAction: "Send connection request"` for the next daily run.
5. **Email → Instantly.** Enrich the email (IG business_email, bio, or LinkedIn contact_info via `get_person_profile`), write a cold email (Mode 6), and push to the Instantly campaign (auto-send). Instantly is the email sender.
6. **Notify Martin** (WhatsApp) and report per channel.

Canonical routing: IG → digital_university pipeline (VA sends); LinkedIn → tracker + `linkedin` MCP (Claude sends in the daily run); email → Instantly. The digital_university pipeline (admin.tribed.io) is the dashboard of record.

## Job D — build the demo from the conversation

The gift-first promise is that the demo already exists. When a prospect's reply shows interest, make it true before the reply goes back: read the conversation, build their real demo community, and put the link on the lead.

1. **Trigger.** A reply classified as interested (asks to see it, "send it over", a curious question about the app) AND the lead has no `data.demo_url` yet. One demo per prospect, ever: re-read the lead with `get_outreach_lead` and check `list_communities` for an existing community before creating anything. Never recreate or duplicate; demos are real communities in the shared Firebase project and deleting one is a manual decision.
2. **Gather the material.** Three sources, in order of weight:
   - The conversation itself (`get_conversation`): what they call their program, what they reacted to, the words they use for their people. What earned the reply shapes the demo's front door.
   - The LinkedIn profile (`get_person_profile` with `sections: "posts,experience,contact_info"`): voice, method, program names, credentials.
   - Their site or bio link if one exists (fetch it): pricing, program structure, brand colors, testimonials.
3. **Build it with the Tribed MCP.** Call `get_onboarding_guide` FIRST and follow it exactly: extract the author, brand, habits, a starter plan, and questionnaire from the gathered material, then `onboard_community` plus the content tools the guide prescribes. Name and theme it as THEIR brand, not Tribed's. Where the material runs thin, build the smallest demo that still feels like theirs (a tight demo beats an empty shell) and note the gaps in `data.demo_notes`.
4. **Record it.** Save `data.demo_handle` and `data.demo_url` (app.tribed.io/[handle]) on the lead, `log_outreach_touch`, set `nextAction: "Send demo link (Mode 2)"`. The Mode 2 draft still uses the `[demo link]` placeholder per the hard rules; the user pastes `data.demo_url` when sending.
5. **Cap and report.** Max 5 demos per daily run; queue the rest for tomorrow. Report every demo created (lead, community id, demo URL) in the daily report.

## Guardrails

- Never enroll an out-of-ICP lead.
- Never ship a message that fails rule zero — hold it for manual review instead.
- Sample-then-bulk on any first run against a new batch.
- The connection note has no CTA and no price; pre-demo follow-ups never ask for a call.
- Respect the daily caps. When in doubt, send less.
