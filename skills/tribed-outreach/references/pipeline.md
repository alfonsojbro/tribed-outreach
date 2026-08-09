# Mode 7 — Pipeline ops: LinkedIn sequences, the review queue, and the daily run

LinkedIn outreach runs on the self-hosted **LinkedIn MCP** (`linkedin` server, stickerdaniel/linkedin-mcp-server) driving Alfonso's real LinkedIn session, with the **Tribed outreach tracker** as the record of state. This replaced Gojiberry. There is no campaign engine anymore: the daily run IS the campaign engine. Requires the `linkedin` MCP (actions), the Tribed MCP (outreach lead + queue tools, community onboarding for Job D), and the Apify MCP (follower enrichment).

**The queue is the record, and approval is the ship gate.** Nothing is ever sent that was not written to the queue first: every personalized draft goes in with `queue_outreach_drafts` (Tribed MCP) before a message leaves, so the exact copy is on the record either way. A draft reaches `approved` one of two ways:

- **LinkedIn connection notes (`li_dm`) — the run may approve its own.** Under Alfonso's direct-send authorization (2026-08-06), an unattended run calls `approve_outreach_draft` on the drafts it just wrote and ships them in the SAME run, up to a hard daily cap the server enforces. Order is always queue → approve → send → `mark_outreach_draft_shipped`.
- **Everything else — a human approves in the dashboard.** IG DMs go to the VA by hand and email goes through a person, both via the NobleAdmin Review tab (admin.tribed.io, Messaging > Review). The run cannot approve those kinds; `approve_outreach_draft` refuses them.

Follow-ups stored on a lead by an approved draft were approved with it — they send on schedule without re-approval. Reply drafts (Mode 2) are never auto-approved or auto-sent. Interactive sessions where the user approves samples in chat may send directly.

## How the stack is wired (know this before touching anything)

- **The `linkedin` MCP does the actions.** It automates a logged-in browser session, so calls run sequentially through one browser and long runs are normal. The tools that matter here:
  - `connect_with_person({ linkedin_username, note })` — connection request; the `note` is where the connection note goes.
  - `send_message({ linkedin_username, message, confirm_send: true, profile_urn? })` — DM. Only works once they are a 1st-degree connection. Pass `profile_urn` (from `get_person_profile`) when you have it; it is more reliable than the username lookup.
  - `get_inbox({ limit })`, `get_conversation({ linkedin_username | thread_id })`, `search_conversations({ keywords })` — read the inbox and detect replies.
  - `get_person_profile({ linkedin_username, sections? })` — profile scrape (anchor material, connection degree, URN).
  - `search_people({ keywords, location?, network? })` — prospecting; `network: ["F"]` filters to 1st-degree (useful to confirm acceptance in bulk).
- **The Tribed tracker holds the state.** Every LinkedIn lead is a record in the outreach tracker (`upsert_outreach_leads`, accountId `"digital_university"`, `channel: "li"`). The approved copy lives in the lead's `data` bag: `data.li_note`, `data.li_followup_1`, `data.li_followup_2`, plus `data.li_username` (the publicIdentifier) and `data.li_urn` when known. Always set `externalId` to the LinkedIn publicIdentifier: upsert dedupes on `externalId` + `channel`, so re-running discovery never duplicates a person. Sequence position lives in `stage` plus `nextAction` (what to do) and `nextActionAt` (YYYY-MM-DD, when it's due). The dashboard's ops home renders a **LinkedIn paced sequences** card projected from exactly these fields (invite → follow-up 1 on acceptance → follow-up 2): what the drip will send today, what is parked and why (stale copy, missing note, no username), and which threads a human owns. It is derived, not stored, so it can never disagree with the drip — read it to see the day before it runs. Every send is recorded with `log_outreach_touch`, using `advanceTo` to move the stage. The dashboard (admin.tribed.io) is the source of truth.
- **Parking a lead for a human is a state, not a date.** When a person takes a lead back (they replied, they asked to be left alone, they are on hold until a date), record the ownership, not just the calendar: `log_outreach_touch` with `automated: false`, and set `data.li_state: "human"` on the lead. The LinkedIn drip acts only on `li_state` `to_invite` / `awaiting_accept` / `await_fu2` and skips anything else or anything carrying `automated: false`, so it never writes to a parked lead. A future `nextActionAt` alone is a snooze, not a park: it only defers the lead, and the day it comes due the drip picks it up again. Göran Hielscher was parked on 2026-07-31 and the drip overwrote his `nextAction` back to "Draft today's copy, then send" on 08-06. Handing a lead back TO the drip takes both fields: set `data.li_state` to a drip state and log a touch with `automated: true`. Reset only one and the lead stays parked.
- **The sequence map** (what replaced the Gojiberry steps):

  | Old Gojiberry step | Now |
  |---|---|
  | 0 — invitation + note | `connect_with_person` with `note` (ships from an approved draft) |
  | 2 — 1st message after connect (follow-up 1) | `send_message` on the first daily run after they accept |
  | 5 — 2nd message (follow-up 2) | `send_message` 3-4 days after follow-up 1 if no reply |
  | 1 / 4 / 6 — emails | Instantly (Mode 6), via an approved `email` draft. The LinkedIn side sends no email. |
  | 3 — profile visit | dropped |

- **Nothing sends itself.** A message goes out only when the daily run (or the user) sends it, and only after its draft is on the record and approved — by the run itself for a LinkedIn connection note, by a human for anything else. A missed daily run only delays touches. It never double-sends, because you check the lead's touch log before sending and log every send after.
- **Lead sourcing** (Gojiberry's agents are gone): leads come from the IG-first combined pipeline below, from the user handing over profiles, or from `search_people` prospecting when asked. Everything gets the same ICP gate.

Session plumbing: the server is registered as `linkedin` in NobleAdmin/.mcp.json and keeps its session in `~/.linkedin-mcp/`. If tools start failing with login or session errors, re-authenticate in a terminal with `uvx mcp-server-linkedin@latest --import-from-browser` (imports from the signed-in browser) or `--login`, then retry.

## The review queue (Tribed MCP, the unattended run's ship gate)

Five tools, all on `accountId "digital_university"` (the dashboard of record):

- `queue_outreach_drafts` — write the run's drafts. Each: `kind` (`"li_dm"` | `"ig_dm"` | `"email"`), `leadName`, `handle`, `externalId` (LinkedIn publicIdentifier / IG handle / email address; the dedupe key), `anchor` (the rule-zero detail), `language`, `messages` (`[{ slot, content }]` — slots `connection_note`, `followup_1`, `followup_2` for li_dm; `dm` + follow-ups for ig_dm; `email_1`... for email), and `shipTarget` (what shipping needs later: `{ profileUrl, profileUrn? }` / `{ owner }` / `{ campaignId, email }`). No usable anchor? Queue it anyway with `anchor` omitted — it lands as **held** and a human supplies the anchor, instead of a generic message shipping. Idempotent on (kind, externalId); an approved draft is never overwritten.
- `list_outreach_drafts` — read the queue by status: `pending` | `held` | `approved` | `rejected` | `shipped` | `error`. Auto-approved drafts carry `autoApproved: true` and `autoApprovedAt`, so you can tell the run's own approvals from a human's.
- `approve_outreach_draft` — approve `li_dm` drafts so THIS run can ship them (direct-send authorization). Pass `draftId` for one or `draftIds` for a batch, plus an optional `note` recorded on the draft. The result gives a per-draft verdict and `remainingToday`, so you always know the budget left. The server refuses anything outside the authorization and says why: a non-`li_dm` kind, a draft that is not `pending`, a missing anchor, a missing or over-long `connection_note`, or the daily cap being spent. Refused drafts are untouched and can still be approved by a human. Re-approving an already-approved draft is a no-op success, so a retried run never fails on it. A rejected draft can never be approved.
- `reject_outreach_draft` — retire a draft the run has determined can never ship, with a REQUIRED `reason` stored on the draft and shown in the Review tab (e.g. `"already has a pending LinkedIn invite from 2026-07-23, there is no second invite to send"`). Pass `draftId` for one or `draftIds` for a batch; the result reports each draft separately, so a partial failure is visible. Only `pending` | `held` | `error` can be rejected — an `approved` draft is a human decision the run must not override (reject that one in the dashboard), and `shipping`/`shipped` are final. Re-rejecting an already-rejected draft is a no-op success, so a retried run never fails on it. Use it instead of leaving dead drafts in the queue forever.
- `mark_outreach_draft_shipped` — report each ship attempt: `ok: true` with a `detail` of what happened, or `ok: false` with the `error` (the draft resurfaces in the dashboard and retries next run).

The run's self-approval is deliberately narrow, and the server enforces it rather than trusting the prompt. `approve_outreach_draft` covers LinkedIn connection notes and nothing else, only from `pending`, only with a real anchor and a real note inside LinkedIn's length limit, and only within the daily cap. Rejection stays broader on purpose: a run rejecting its own draft can only ever mean less outreach goes out. What a run can never do is override a person — it cannot resurrect a rejected draft, and it cannot un-approve or re-write one a human signed off on. If you find yourself wanting to widen the approve path (another kind, a held draft, a bigger cap), that is a decision for Alfonso, not a workaround to code around.

## Job A — personalize a lead

For each in-ICP lead:

1. Gather the anchor. Best source is the scraped LinkedIn profile via `get_person_profile` (`headline`, `about`, a named program/book, current role; add `sections: "posts"` when the main page gives nothing). IG anchors come from the profile/feed (Mode 1 / instagram.md). Apply **rule zero** — one real detail, never the niche.
2. Write, in the Tribed voice:
   - **Connection note.** Mode 4A rules: opens with their first name, under ~280 characters (confirm the count), one line on their detail, one line hinting something was built for them, no price, no features, no explicit CTA.
   - **Follow-up 1 (case study).** Mode 3 follow-up 1: how a similar coach's branded app worked, one soft question. Honest, no invented names or metrics; a [case study link] placeholder if useful.
   - **Follow-up 2 (video).** Mode 3 follow-up 2: offer a 60-second explainer video of their own demo, [video link] placeholder, lightest CTA or none.
   - (Optional) **emails** — Mode 6 (email.md), pushed to Instantly via an `email` draft, only if email is enabled for the run.
3. Rotate skeletons and CTAs across leads (see core). Never reuse a line across contacts.
4. Ship or queue, depending on who's watching:
   - **Interactive session (user in chat):** sample 3–5 leads for approval, then save to the tracker (`upsert_outreach_leads`, messages in `data.li_note` / `data.li_followup_1` / `data.li_followup_2`, `nextAction: "Send connection request"`) and send within today's budget. Verify the first of a batch with `get_outreach_lead` before doing the rest.
   - **Unattended run:** queue with `queue_outreach_drafts` (`kind "li_dm"`), then `approve_outreach_draft` and ship in the same run — writing the draft first replaces the in-chat sample approval, so the copy is on the record before the invite goes out. Nothing touches the tracker or LinkedIn until the draft is approved and ships.

## Job B — the ICP filter (run before personalizing new leads)

Filter on real audience signal, not role or industry fit:

1. Collect the batch's `li_username`s (publicIdentifiers).
2. Enrich followers with Apify actor `harvestapi/linkedin-profile-scraper`, mode `"Profile details no email ($4 per 1k)"`, passing the publicIdentifiers. It returns `followerCount`, `connectionsCount`, `creator`, `influencer`. Cost ≈ $0.004/lead. (For a handful of leads, `get_person_profile` follower counts do the job without Apify.)
3. **Keep** a lead if `followerCount >= 2000` OR `creator === true` OR `influencer === true`. Everyone else is out of ICP.
4. Out-of-ICP leads are simply not worked. If one is already on the tracker, take it out of the pipeline with `update_outreach_lead`: clear the `nextAction` and note why in `data` (e.g. `data.icp: "out - 800 followers"`).

## Job C — the daily pipeline (SHIP → DRAFT → REPORT)

Run once per day. Preflight first: one cheap `get_my_profile` call — if it fails with a session/login error, stop and tell the user to re-auth (see session plumbing above) instead of half-running. Then pull the day's due work with `list_outreach_leads({ accountId: "digital_university", channel: "li", dueBefore: <today>, activeOnly: true })` — every outreach tool call needs the explicit `accountId` (there is no default account) — the tracker's `nextActionAt` is the scheduler. That reply is one page: it carries `total` and `nextOffset`, so if `hasMore` is true, page through with `offset` before treating the day's due work as covered. Order matters: replies first, then advancing existing leads, then approved new invites, then drafting.

### 1. SHIP

1. **Replies.** `get_inbox({ limit: 50 })` and match conversation names against open LinkedIn leads in the tracker. Any reply: stop that lead's sequence with `log_outreach_touch` (`advanceTo: "replied"`, or `"interested"` when the reply shows real interest; `nextAction: "Handle reply (Mode 2)"`), read the thread with `get_conversation`, store the thread id in `data.li_thread_id` for reliable lookups later, and draft a Mode 2 reply for the user. Replies are never auto-sent. If the reply shows interest and the lead has no demo yet, build the demo first (Job D) so the Mode 2 draft has something real behind it.
2. **Acceptances → follow-up 1.** For due leads awaiting acceptance: check degree with `get_person_profile` (or one `search_people` name query with `network: ["F"]`). Accepted: send `data.li_followup_1` via `send_message` (with `profile_urn` if stored), then `log_outreach_touch` with `nextAction: "Send follow-up 2"` and `nextActionAt` 3-4 days out. Not accepted yet: bump `nextActionAt` a day. If `send_message` fails because they are not messageable, they have not accepted; treat as not accepted. Pending 21+ days: stop waiting, mark `stage: "lost"` and note it (the MCP has no invite-withdraw tool — flag stale invites in the report so the user can withdraw them by hand; LinkedIn throttles accounts with too many pending invites).
3. **Due follow-up 2.** Leads due today with `nextAction: "Send follow-up 2"` and still no reply (verify against the thread with `get_conversation` before sending — the touch log can lag a crashed run): send `data.li_followup_2`, log it with `nextAction: "Archive if still quiet"` and `nextActionAt` 14 days out. Due again and still quiet: mark `stage: "lost"`.
4. **Approved drafts.** `list_outreach_drafts({ status: "approved" })` (plus `status: "error"` for retries), per kind, within today's budget:
   - **li_dm:** `connect_with_person(<li_username from shipTarget/externalId>, <connection_note>)`, then upsert the lead to the tracker (`channel "li"`, `externalId` = publicIdentifier, messages into `data.li_note` / `data.li_followup_1` / `data.li_followup_2`, `data.li_username`, `data.li_urn` if known) and `log_outreach_touch` (`advanceTo: "reached"`, `nextAction: "Check acceptance"`, `nextActionAt` tomorrow). Approved drafts beyond today's invite budget stay approved and ship on a later run.
   - **ig_dm:** upsert the lead with `channel "ig"`, `owner` from `shipTarget.owner` (e.g. "Martin"), messages in `data` (`data.ig_dm`, `data.ig_followup_1`, `data.ig_followup_2`), `nextAction "Send IG DM"`, `stage "top"` — the VA sends by hand.
   - **email:** push to the Instantly campaign from `shipTarget` (Instantly auto-sends its sequence).
   After each attempt: `mark_outreach_draft_shipped({ draftId, ok, detail | error })`.
   Draft that can never ship rather than failed to ship — the prospect already has a pending invite from an earlier run (`data.li_invited_at` set, `li_state: "awaiting_accept"`, and there is no second invite to send), the profile is gone, or they already replied on another channel: retire it with `reject_outreach_draft({ draftIds: [...], reason: "<the concrete reason>" })` instead of leaving it in the queue for a human to clear by hand. Never reject an `approved` draft; that one is the human's call.

### 2. DRAFT

The day's new prospects (from the IG-first pipeline below, `search_people` on rotated queries, or handed over by the user): skip-check each candidate first (`list_outreach_leads({ search: "<handle>" })` — never re-work someone already in the pipeline), ICP filter (Job B), personalize the survivors (Job A), and queue everything with `queue_outreach_drafts`. Anchor-less prospects get queued with no anchor (→ held), never written generic. Rotate discovery searches via `get_outreach_state({ key: "query_rotation" })` (oldest `last_used` first, nulls first) and stamp them with `set_outreach_state` after.

Then ship the day's LinkedIn first touches, within whatever invite budget phase 1 left:

1. `approve_outreach_draft({ draftIds: [<the li_dm drafts just queued>], accountId: "digital_university" })`. Read `remainingToday` from the result — that is the real budget, not your own count. Anything refused comes back with its reason; act on it rather than retrying (a `held_needs_anchor` draft needs a real anchor, a `daily_cap_reached` one simply waits for tomorrow) and never try to route around a refusal.
2. Ship each approved one exactly as in phase 1's `li_dm` step: `connect_with_person`, verify `note_sent`, upsert + `log_outreach_touch`, then `mark_outreach_draft_shipped`.

`ig_dm` and `email` drafts are NOT approved here — they stay pending for the dashboard, and the digest nudges the human to the Review tab. Nothing else in this phase sends or touches the tracker.

### 3. REPORT

1. `set_outreach_state({ key: "daily_run", value: { at, perChannel: { discovered, kept, removed, drafted, held, shipped, shipErrors } } })` so the dashboard's Review tab shows the last run.
2. **Snapshot the funnel.** Get the stage counts from `get_outreach_lead_counts({ accountId: "digital_university" })` — one call, every lead counted, broken down per channel per stage with no page cap. Never tally `list_outreach_leads` pages for this: that tool returns a page, so a tally of it reports a floor as if it were a total (that is how the 2026-08-09 snapshot came out short — the account had 708 `li` and 513 `ig` leads behind a 500 cap). Map each channel's `byStage` onto the schema-v1 normalized stages and push with `upsert_outreach_metrics` (account, generatedAt, channels[]; `accountId: "digital_university"`). Idempotent on the snapshot date, so re-running the same day is safe.
3. `send_admin_report` digest: replies found, acceptances messaged, follow-up 2s sent, new invites sent, drafts queued and held (nudge to the dashboard's Review tab), drafts rejected with their reasons, kept vs removed by ICP, demos built, stale invites to withdraw, ship errors.

### Volume + account safety (hard limits)

This is browser automation on Alfonso's real LinkedIn account, which LinkedIn's terms prohibit. Volume discipline is what keeps the account alive:

- Max **15 connection requests** and **25 messages** per day. One daily run.
- Calls queue through one browser session; never try to parallelize, and expect long runs.
- If a tool errors with anything that smells like a checkpoint, captcha, or verification wall, STOP the run immediately and tell the user. Do not retry.
- Sends happen only inside the daily run or on an explicit ask.
- Other channels: max 25 ig_dm handoffs and 50 emails per run. Approved drafts over a cap wait.

### Combined cross-channel daily pipeline (IG-first)

The daily job targets one person across all three channels. Start on Instagram and fan out. All of this happens in the DRAFT phase — every message rides the queue:

1. **Discover on Instagram** (WebSearch + Apify `figue/instagram-profile-scraper`): qualified coaches 10k-500k followers with a bio link. Capture `external_url` (bio link), `business_email`, bio, captions.
2. **IG copy → queue.** Draft a gift-first DM + two follow-ups (Mode 1 + Mode 3). Queue as `kind "ig_dm"`, `externalId` = IG handle, `shipTarget: { owner: "Martin" }`. IG is never auto-sent — after approval the lead lands on the VA's list and the VA sends by hand.
3. **High-confidence match to LinkedIn.** The reliable bridge is the **IG bio link**: if it's a `linkedin.com/in/` URL use it directly; if it's a personal site/Linktree, fetch it and read the LinkedIn link off it; else search by exact name + a corroborating signal (same company / same website domain). If nothing confidently matches, keep the lead **IG-only** — never guess, a wrong match messages the wrong person.
4. **Matched → queue for the LinkedIn sequence.** ICP-gate (Job B), personalize (Job A), queue as `kind "li_dm"`.
5. **Email → queue.** Enrich the email (IG business_email, bio, or LinkedIn contact_info via `get_person_profile`), write a cold email (Mode 6), queue as `kind "email"` with the Instantly campaign in `shipTarget`.
6. **Notify Martin** (WhatsApp) when shipped IG drafts land on his list, and report per channel.

Canonical routing: IG → digital_university pipeline (VA sends); LinkedIn → tracker + `linkedin` MCP (the daily run sends); email → Instantly. The digital_university pipeline (admin.tribed.io) is the dashboard of record — its Review tab is where drafts get approved.

## Job D — build the demo from the conversation

The gift-first promise is that the demo already exists. When a prospect's reply shows interest, make it true before the reply goes back: read the conversation, build their real demo community, and put the link on the lead.

1. **Trigger.** A reply classified as interested (asks to see it, "send it over", a curious question about the app) AND the lead has no `data.demo_url` yet. One demo per prospect, ever: re-read the lead with `get_outreach_lead` and check `list_communities` for an existing community before creating anything. Never recreate or duplicate; demos are real communities in the shared Firebase project and deleting one is a manual decision.
2. **Gather the material.** Three sources, in order of weight:
   - The conversation itself (`get_conversation`): what they call their program, what they reacted to, the words they use for their people. What earned the reply shapes the demo's front door.
   - The LinkedIn profile (`get_person_profile` with `sections: "posts,experience,contact_info"`): voice, method, program names, credentials.
   - Their site or bio link if one exists (fetch it): pricing, program structure, brand colors, testimonials.
3. **Build it with the Tribed MCP.** Call `get_onboarding_guide` FIRST and follow it exactly: extract the author, brand, habits, a starter plan, and questionnaire from the gathered material, then `onboard_community` plus the content tools the guide prescribes. Name and theme it as THEIR brand, not Tribed's. Where the material runs thin, build the smallest demo that still feels like theirs (a tight demo beats an empty shell) and note the gaps in `data.demo_notes`.
4. **Record it.** Save `data.demo_handle` and `data.demo_url` (app.tribed.io/[handle]) on the lead, then `log_outreach_touch` with `advanceTo: "interested"` and `nextAction: "Send demo link (Mode 2)"`. The Mode 2 draft still uses the `[demo link]` placeholder per the hard rules; the user pastes `data.demo_url` when sending.
5. **Cap and report.** Max 5 demos per daily run; queue the rest for tomorrow. Report every demo created (lead, community id, demo URL) in the daily report.

## Guardrails

- Never work an out-of-ICP lead.
- Never ship a message that fails rule zero — queue it as held (no anchor) instead.
- Unattended runs never send anything that was not queued as a draft first; the queue is the only path. The run may approve its own LinkedIn connection notes within the server-enforced bounds, and only those — IG DMs and email still wait on a person in the dashboard.
- Reply-check before every send; never message someone who already replied — advance their stage instead.
- Sample-then-bulk on any first interactive run against a new batch.
- The connection note has no CTA and no price; pre-demo follow-ups never ask for a call.
- Respect the daily caps. When in doubt, send less.
