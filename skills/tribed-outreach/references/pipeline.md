# Mode 7 — Pipeline ops: LinkedIn sequences, the review queue, and the daily run

LinkedIn outreach runs on the self-hosted **LinkedIn MCP** (`linkedin` server, stickerdaniel/linkedin-mcp-server) driving Alfonso's real LinkedIn session, with the **Tribed outreach tracker** as the record of state. This replaced Gojiberry. There is no campaign engine anymore: the daily run IS the campaign engine. Requires the `linkedin` MCP (actions), the Tribed MCP (outreach lead + queue tools, community onboarding for Job D), and the Apify MCP (follower enrichment).

**The queue is the record, and approval is the ship gate.** Nothing is ever sent that was not written to the queue first: every personalized draft goes in with `queue_outreach_drafts` (Tribed MCP) before a message leaves, so the exact copy is on the record either way. A draft reaches `approved` one of two ways:

- **LinkedIn connection notes (`li_dm`) — the run may approve its own.** Under Alfonso's direct-send authorization (2026-08-06), an unattended run calls `approve_outreach_draft` on the drafts it just wrote and ships them in the SAME run, up to a hard daily cap the server enforces. Order is always queue → approve → send → `mark_outreach_draft_shipped`.
- **Everything else — a human approves in the dashboard.** IG DMs go to the VA by hand and email goes through a person, both via the NobleAdmin Review tab (admin.tribed.io, Messaging > Review). The run cannot approve those kinds; `approve_outreach_draft` refuses them.

Follow-ups stored on a lead by an approved draft were approved with it — they send on schedule without re-approval. Reply drafts (Mode 2) are never auto-approved or auto-sent. Interactive sessions where the user approves samples in chat may send directly.

## How the stack is wired (know this before touching anything)

- **The `linkedin` MCP does the actions.** It automates a logged-in browser session, so calls run sequentially through one browser and long runs are normal. The tools that matter here:
  - `connect_with_person({ linkedin_username, note })` — connection request; the `note` is where the connection note goes. **`connect_unavailable` is almost never "closed to requests."** On creator profiles (most of this ICP) Follow is the primary button and Connect sits under the More menu; the engine opens More and the payload reports `observed_state`, `connect_route` and `more_menu_opened` so you can see which path it took. A `connect_unavailable` (or a bare no-payload error) means Follow-primary until the payload proves otherwise — never reject a draft or mark a lead uninvitable on it alone; retry next run and only give up when `observed_state` actually shows no Connect anywhere.
  - `send_message({ linkedin_username, message, confirm_send: true, profile_urn? })` — DM. Only works once they are a 1st-degree connection. **Send by `linkedin_username` alone.** `profile_urn` is not a sturdier username: it builds the compose URL directly and skips the profile lookup entirely, so a URN belonging to somebody else opens a composer addressed to that person and the send comes back successful. Only pass a `profile_urn` you read from `get_person_profile` in the same run, for that same person. A URN out of the tracker is only as good as the run that wrote it, and runs before 2026-08-15 wrote some wrong ones (see the URN rule below). **A real send can take ~20 minutes to appear in the thread** (measured 2026-08-17): "not in the thread" minutes after `sent: true` is NOT a phantom send. Never resend because a fresh thread read came back empty — poll `data.li_thread_id` after a few minutes if you must know today, otherwise just recheck next run.
  - `get_inbox({ limit })`, `get_conversation({ linkedin_username | thread_id })`, `search_conversations({ keywords })` — read threads and detect replies. **None of them paginate.** `get_inbox` returns the 10-20 most recent threads and never clicks "Load more conversations"; `search_conversations` returns 10 rows and ignores any `limit` above that. Reply coverage therefore comes from looking leads up by name one at a time, not from listing the mailbox — see Job C phase 1 and "Why the reply sweep has three passes".
  - `get_person_profile({ linkedin_username, sections? })` — profile scrape (anchor material, connection degree, URN).
  - `search_people({ keywords, location?, network? })` — prospecting; `network: ["F"]` filters to 1st-degree (useful to confirm acceptance in bulk).
- **The Tribed tracker holds the state.** Every LinkedIn lead is a record in the outreach tracker (`upsert_outreach_leads`, accountId `"digital_university"`, `channel: "li"`). The approved copy lives in the lead's `data` bag: `data.li_note`, `data.li_followup_1`, `data.li_followup_2`, plus `data.li_username` (the publicIdentifier) and `data.li_urn` when known. Always set `externalId` to the LinkedIn publicIdentifier: upsert dedupes on `externalId` + `channel`, so re-running discovery never duplicates a person. Sequence position lives in `stage` plus `nextAction` (what to do) and `nextActionAt` (YYYY-MM-DD, when it's due). The dashboard's ops home renders a **LinkedIn paced sequences** card projected from exactly these fields (invite → follow-up 1 on acceptance → follow-up 2): what the drip will send today, what is parked and why (stale copy, missing note, no username), and which threads a human owns. It is derived, not stored, so it can never disagree with the drip — read it to see the day before it runs. Every send is recorded with `log_outreach_touch`, using `advanceTo` to move the stage. The dashboard (admin.tribed.io) is the source of truth.
- **A URN is a person's identity, so it only ever comes from their own profile.** `data.li_urn` may be written from exactly one source: the top-level `profile_urn` field that `get_person_profile({ linkedin_username })` returns for **that** person. Never from a `references` block, a sidebar or "people also viewed" card, a search-result row, a thread participant, or any other `ACoAA…` string that happened to be on the page — those are other people's URNs, and a profile page is full of them. If `get_person_profile` returns no `profile_urn`, the lead has no URN: write nothing. A missing URN costs nothing (the send falls back to the username lookup); a wrong one addresses a stranger. The engine used to read the first compose anchor anywhere in `<main>`, which on any not-yet-connected profile is a sidebar stranger's — that put four wrong URNs on the tracker from the 2026-08-10 run alone (fixed 2026-08-15 in `linkedin-worker/patch_engine.py`, patches `subject-own-urn` and `urn-contradiction-guard`). Leads corrected by hand keep the old value in `data.li_urn_previous_wrong`.
- **Parking a lead for a human is a state, not a date.** When a person takes a lead back (they replied, they asked to be left alone, they are on hold until a date), record the ownership, not just the calendar: `log_outreach_touch` with `automated: false`, and set `data.li_state: "human"` on the lead. The LinkedIn drip acts only on `li_state` `to_invite` / `awaiting_accept` / `await_fu2` and skips anything else or anything carrying `automated: false`, so it never writes to a parked lead. A future `nextActionAt` alone is a snooze, not a park: it only defers the lead, and the day it comes due the drip picks it up again. Göran Hielscher was parked on 2026-07-31 and the drip overwrote his `nextAction` back to "Draft today's copy, then send" on 08-06. Handing a lead back TO the drip takes both fields: set `data.li_state` to a drip state and log a touch with `automated: true`. Reset only one and the lead stays parked.
- **`dedupe_outreach_leads` hard-deletes and its matcher misses real duplicates — never run it with `apply: true`.** On 2026-08-13 the tracker held 39 real duplicate groups (91 records) and the tool found 0 of them; whatever it deletes is gone, not merged. Duplicates are found by hand: match on normalized publicIdentifier UNION normalized name across `li` leads, pick the canonical record, park the loser (`data.li_state: "human"` AND `automated: false` — both, per the parking rule) and stamp it `data.duplicateOf: "<canonical externalId>"` (the human-written field; a `duplicate_of` written by tooling is a different, untrusted field). Prevention beats cleanup: always upsert with `externalId` = publicIdentifier so discovery reruns dedupe on write.
- **Acceptance is READ, never inferred. This is the one rule to get right.** A LinkedIn invite counts as accepted only when you have read a connection degree of **"· 1st"** off the profile top card (`get_person_profile`) or seen the person come back from `search_people` with `network: ["F"]`. Nothing else is evidence:

  | Signal | What it proves about the connection |
  |---|---|
  | The message sent / delivered | Nothing. LinkedIn **Open Profile** messaging works at 2nd degree with no connection at all, and this ICP (large creator and Premium profiles) is full of open profiles. |
  | The composer opened / a composer probe succeeded | Nothing. Every Open Profile 2nd-degree shows a working composer. |
  | `composer_unavailable`, or a send that failed | Nothing, in **either** direction. It is inconclusive about degree — never "they have not accepted", and never a reason to stop re-checking. |
  | They **replied** | Nothing. Steve Dennis and Steven Jordan both replied and were still "· 2nd" on 2026-08-14. |
  | The profile top card reads "· 1st" | They accepted. |
  | `search_people` returns them under `network: ["F"]` | They accepted. |

  Whenever you record an acceptance, record what proved it in the same write: `data.li_degree` (`"1st"`), `data.li_degree_checked_at` (ISO), `data.li_degree_source` (`"profile_top_card"` or `"search_people_network_f"`). **This is on you, not on the tools** (verified 2026-08-17): `li_degree` appears nowhere in the MCP source, so `update_outreach_lead` / `upsert_outreach_leads` accept an `li_accepted_at` or an `await_fu2` with no proof attached and no complaint. Earlier text here claimed the MCP refuses those writes. It does not. Nothing will stop you writing an acceptance you cannot show, which is exactly why the discipline has to be yours: go read the degree, and write the triple in the same call.

  This exists because two runs got it wrong and messaged people who had never connected. The 2026-08-04 run wrote the touch note "invite accepted (confirmed by a working composer), sent follow-up 1" for a cohort; the 2026-08-13 run advanced leads to `await_fu2` reasoning that a message delivered on 08-04 "is only possible at 1st degree". Both are false. Of the six leads given composer-inference acceptance, the five that could be read were **all** "· 2nd" (Eva Steortz, Tina Lewis, Lion Fludd, Steve Dennis, Steven Jordan). The one lead that verified as "· 1st" (Jeffrey Magee) got there by a different path. Two more (Gerald Joseph, Halle Eavelyn) carried `li_accepted_at` stamps their own `li_state_note` already called "known false".

  Repairing one of those: read the real degree and write it with `li_accepted_at: ""` in the SAME call. Nothing refuses a write that records a non-1st degree while leaving the acceptance stamp in place (verified against the MCP source 2026-08-18, same as above) — which is exactly why the clear has to travel in the same call: a false stamp left behind is one the next run will believe.

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

1. **Replies.** Three passes, in this order. **The inbox is a recent-window peek, not the sweep** — read "Why the reply sweep has three passes" below before changing any of this.

   **Pass A — the recent window.** `get_inbox({ limit: 50 })`, match conversation names against open LinkedIn leads. This catches today's replies and anyone who wrote in who is not on the tracker at all. It returns only the threads LinkedIn first-paints (10-20, measured 17 on 2026-08-14) and stops at "Load more conversations". Never treat an absence here as "no reply".

   **Pass B — the tracker-driven sweep. This is the one that actually finds replies.** Build the watch list from the tracker, not from the inbox: every `li` lead at stage `replied`, `interested` or `demo_request`, plus every `li` lead carrying a `data.demo_url` with no send logged after the demo was built (the link was never delivered), plus anything already due today. Order it `demo_request` → `interested` → demo-built-but-never-sent → `replied`, and check each lead's thread:
   - **`data.li_thread_id` stored:** `get_conversation({ thread_id })`. One direct navigation, no sidebar enumeration, nothing else marked read. Always the cheapest path, which is why the id gets stored the first time it is ever seen.
   - **No stored thread id:** `search_conversations({ keywords: "<lead display name>", limit: 2 })`, take the thread id off `references` (NOT off `sections` — see below), write it straight to `data.li_thread_id`, then `get_conversation({ thread_id })`. A name search reaches a thread of any age: LaTonia Monroe Naylor (07-21) and Bego Lozano (~8 weeks quiet) both resolved this way on 2026-08-14, and neither is anywhere near the inbox window.

   Cap Pass B at **40 thread checks per run** (browser calls are sequential and slow). If the watch list is longer, work it in priority order and carry the remainder with a cursor in `get_outreach_state({ key: "reply_sweep_cursor" })` / `set_outreach_state`, oldest-checked first, exactly like `query_rotation`. Record the cap being hit in the report so a growing backlog is visible instead of silent.

   **Pass C — content sweep, for what the tracker's stages have gone stale about.** One `search_conversations({ keywords: "<term>", limit: 10 })` on a rotated term from our own copy (`demo`, `app`, `built`, `tribed`), same rotation state pattern. Each query returns at most 10 threads, so this is a sampler, not coverage — its job is to surface threads whose lead record says nothing is pending. The 2026-08-14 run of `"demo"` turned up Bruce Wren, Neal Foard and Deborah Ivanoff, none of them in the inbox window.

   **On any reply found, by any pass:** stop that lead's sequence with `log_outreach_touch` (`advanceTo: "replied"`, or `"interested"` when the reply shows real interest; `nextAction: "Handle reply (Mode 2)"`), store `data.li_thread_id` if it is not already there, and draft a Mode 2 reply for the user. Replies are never auto-sent. If the reply shows interest and the lead has no demo yet, build the demo first (Job D) so the Mode 2 draft has something real behind it. **A lead whose demo exists and was never sent is a reply that was already missed** — draft the delivery, do not re-open the sequence.
2. **Acceptances → follow-up 1.** Two selections, and you need both. **The due list alone does not find acceptances** — read "Why acceptance needs its own sweep" below before changing any of this.

   **2a. Today's due leads awaiting acceptance.** Whatever `nextActionAt` says is ready.

   **2b. The acceptance sweep — the oldest unchecked invites, due or not.** `list_outreach_leads({ accountId: "digital_university", channel: "li", activeOnly: true })`, paging until `hasMore` is false. Then, in the run: keep leads whose `data.li_state` is `"awaiting_accept"`, drop anything human-owned (`data.automated === false`, or an `li_state` outside the drip's three), drop anything already carrying today's `data.li_degree_checked_at`, and order the rest by `data.li_degree_checked_at` **ascending with nulls first** — never-checked leads go first. Check the first **`ACCEPTANCE_SWEEP_BUDGET = 12`** of them. Do not raise that to clear a backlog in one run: 12 per run drains a 76-lead cohort in about six runs, and profile reads are sequential browser calls competing with the day's sends.

   Both selections then run the same check: **read the connection degree** (see the acceptance-proof rule above). Use `search_people({ keywords: "<full name>", network: ["F"] })` **first** — a miss answers "No results found" in two lines, where `get_person_profile` spends thousands of tokens of posts on the same one fact. Reach for the profile read only when a hit needs certainty, or when you also want a fresh anchor or URN. Only a badge reading **"· 1st"**, or a `network: ["F"]` hit, is an acceptance.

   **Stamp every check, accepted or not:** `update_outreach_lead` with `data.li_degree`, `data.li_degree_checked_at` (ISO now) and `data.li_degree_source`. This is the sweep's cursor, not bookkeeping. Without it the same handful of leads get re-read every run while the rest of the backlog is never reached — exactly the state the account was in on 2026-08-17.

   Accepted: record the proof first, then send. `update_outreach_lead` with `data: { li_degree: "1st", li_degree_checked_at: "<ISO now>", li_degree_source: "profile_top_card", li_accepted_at: "<ISO now>" }`. **Then read the thread before writing a word into it:** `get_conversation` (by `data.li_thread_id` if stored, else a name `search_conversations`). Anything from them in there and follow-up 1 is cancelled — park the lead for a human (`automated: false`, `li_state: "human"`, `advanceTo: "replied"`) and draft a Mode 2 reply instead. A newly-detected acceptance is the single most likely place to drop a template on top of a live conversation, because the acceptance and the reply often arrive together. An unreadable thread defers the send to tomorrow; it never clears it.

   Thread clean: `send_message` for `data.li_followup_1` (by `linkedin_username`; pass `data.li_urn` as `profile_urn` only when the tracker holds one AND this run read it off that same person's own profile — never a stored URN from an older run, see the URN rule above), then `log_outreach_touch` with `nextAction: "Send follow-up 2"`, `nextActionAt` 3-4 days out, `automated: true`.

   **The message cap is the worker's ramp, not the nominal 25.** Read `warmup.messagesPerDay` from `https://li-worker.tribed.io/health` before you plan the sends and stop at that number. `MESSAGE_RAMP = (5, 8, 12, 20)` indexed by week since `warmup.json.sessionStartedAt`, and `login.sh` deletes `warmup.json`, so any re-login resets it to 5/day. Acceptances found beyond the ramp keep their degree stamp and their acceptance stamp, stay in `awaiting_accept`, and send on the next run — a detected acceptance is never lost to a spent budget.

   Reads "· 2nd" or anything else: write `data: { li_degree: "<what it read>", li_degree_checked_at: "<ISO now>", li_degree_source: "profile_top_card" }`, and if the lead already carries `li_accepted_at`, clear it in that same write — that stamp was never true, and leaving it lets the next run believe it. Badge unreadable: stamp nothing and bump `nextActionAt` a day; an unreadable badge is inconclusive, not a negative. A failed `send_message` says nothing about degree in either direction — never read it as "not accepted", just retry tomorrow.

   Pending 21+ days **and read as non-1st this run**: mark `stage: "lost"` and list them in the report for manual withdrawal (there is no MCP invite-withdraw tool, and LinkedIn throttles accounts with too many pending invites). **Never mass-mark a pending cohort lost on age alone.** "Invited 25 days ago" plus "no `li_degree` on the record" is not evidence of anything except that nobody looked. On 2026-08-17 that cohort was 76 leads deep and marking it lost would have discarded the real acceptances inside it. Age decides nothing until the degree has been read.
3. **Due follow-up 2.** Leads due today with `nextAction: "Send follow-up 2"`, a stored `data.li_degree` of `"1st"`, and still no reply (verify against the thread with `get_conversation` before sending — the touch log can lag a crashed run): send `data.li_followup_2`, log it with `nextAction: "Archive if still quiet"` and `nextActionAt` 14 days out. Due again and still quiet: mark `stage: "lost"`. **No stored `li_degree`, or one that is not `"1st"`: do not send the closer.** Re-read the degree as in step 2 and act on what it says. This is a hard gate, not a preference: the hourly drip re-reads the degree before the closer and drops such a lead back to `awaiting_accept` on its next tick (clearing the false `li_accepted_at` with it). Nothing in the MCP stops you writing `await_fu2` without proof, so the gate only holds if you apply it.
4. **Approved drafts.** `list_outreach_drafts({ status: "approved" })` (plus `status: "error"` for retries), per kind, within today's budget:
   - **li_dm:** `connect_with_person(<li_username from shipTarget/externalId>, <connection_note>)`, then upsert the lead to the tracker (`channel "li"`, `externalId` = publicIdentifier, messages into `data.li_note` / `data.li_followup_1` / `data.li_followup_2`, `data.li_username`, and `data.li_urn` only if this run read a `profile_urn` off that person's own `get_person_profile`) and `log_outreach_touch` (`advanceTo: "reached"`, `nextAction: "Check acceptance"`, `nextActionAt` tomorrow). Approved drafts beyond today's invite budget stay approved and ship on a later run.
   - **ig_dm:** upsert the lead with `channel "ig"`, `owner` from `shipTarget.owner` (e.g. "Martin"), messages in `data` (`data.ig_dm`, `data.ig_followup_1`, `data.ig_followup_2`), `nextAction "Send IG DM"`, `stage "top"` — the VA sends by hand.
   - **email:** push to the Instantly campaign from `shipTarget` (Instantly auto-sends its sequence).
   After each attempt: `mark_outreach_draft_shipped({ draftId, ok, detail | error })`.
   Draft that can never ship rather than failed to ship — the prospect already has a pending invite from an earlier run (`data.li_invited_at` set, `li_state: "awaiting_accept"`, and there is no second invite to send), the profile is gone, or they already replied on another channel: retire it with `reject_outreach_draft({ draftIds: [...], reason: "<the concrete reason>" })` instead of leaving it in the queue for a human to clear by hand. Never reject an `approved` draft; that one is the human's call.

#### Why acceptance needs its own sweep (measured 2026-08-17 — do not re-derive)

The due list is a scheduler, and a scheduler cannot answer "who has accepted?". On 2026-08-17 the account held **83 leads in `awaiting_accept`, 76 of them invited 2026-07-23** — 25 days pending — and all 30 of the ones on page 1 of the due list had **`data.li_degree` unset. Never read once.** The run reported "zero acceptances." Acceptances had in fact come in. Their `data.li_followup_1` was written and approved and simply never sent.

Three mechanics produced that, and the sweep exists because fixing any one alone is not enough:

- **The copy-freshness gate ran ahead of the degree read.** In the hourly drip (`mcp/src/repos/linkedinDrip.ts`) the `li_copy_at` check sat before the branch that resolves the profile, so a lead whose copy was stamped 07-23 was parked "draft today's copy" and returned — its profile never resolved, its degree never read, every tick, for 25 days. **A degree read needs no copy at all.** Gate sends on copy freshness; never gate reads.
- **A lead re-parked at `nextActionAt: today` is permanently due and permanently stuck.** It sorts to the front of the due list (soonest first), fills the page cap, gets re-parked, and repeats. 76 such leads crowd out the genuinely due work behind them while none of them advance. That is why a lead's presence on the due list says nothing about whether it is progressing.
- **A spent message budget used to defer the whole lead.** A tick that had already sent its messages skipped `awaiting_accept` leads entirely, so it learned nothing about who had accepted. Reads cost no quota; only budget the send.

So acceptance checking is now keyed on **the degree stamp, not the calendar**: order by `data.li_degree_checked_at` ascending, nulls first, take a fixed budget, and stamp every check. The stamp is the cursor — it is what makes the sweep resumable across runs and what stops the same few leads absorbing the whole budget forever.

**And never report "zero acceptances" without the denominator.** That phrase is what let this hide for 25 days: it reads as "nobody accepted" when it meant "nobody was asked". The digest says how many degrees were **checked**, how many were unreadable, and how many `awaiting_accept` leads are still unswept.

#### Why the reply sweep has three passes (measured 2026-08-14 — do not re-derive)

Neither messaging tool paginates. Both read the rows LinkedIn happens to have rendered, and the `limit` argument does far less than its name suggests:

| Call | What actually came back |
|---|---|
| `get_inbox({ limit: 50 })` | **17 threads**, oldest Aug 4, with "Load more conversations" still sitting at the bottom of the sidebar. An earlier run the same day got **10**. |
| `search_conversations({ keywords: "demo", limit: 10 })` | 10 threads |
| `search_conversations({ keywords: "demo", limit: 50 })` | **the identical 10 threads.** Above 10, `limit` is a no-op. |
| `search_conversations({ keywords: "<a person's name>", limit: 2 })` | The right thread, at any age. |

The engine source says why. `get_inbox` scrolls the sidebar `limit // 10` times and then reads the rendered rows, but **nothing in the engine ever clicks "Load more conversations"** — the sidebar paginates by button, not by scroll, so the scroll walks to the button and stops. `search_conversations` does not scroll at all; it navigates to `/messaging/?searchTerm=…` and enumerates first paint, which is 10 rows. For both, `limit` caps how many already-rendered rows get click-visited, and cannot load a row LinkedIn has not drawn.

So coverage cannot come from the mailbox. It has to come from the tracker naming who to look up, one person at a time — which is Pass B, and which is why Pass B is the load-bearing one. That is the whole fix. Before it, the run's entire reply sweep for 733 LinkedIn leads was a 10-to-17-thread window: Bego Lozano's reply sat unanswered for ~8 weeks, and LaTonia Monroe Naylor said "Ok." on 07-21 with a demo already built at `app.tribed.io/mission-rich` that was never sent.

Three more things that bite:

- **Read `references`, never `sections`.** `search_conversations` extracts the page text *before* the sidebar hydrates, so `sections.search_results` routinely reads "No messages…yet!" on a query that found the thread perfectly well. Every successful lookup on 2026-08-14 looked like that. The thread id is in `references`. Concluding "no thread" from the `sections` text is a false negative every time.
- **Enumerating marks threads read.** Both tools capture thread ids by clicking each row, because LinkedIn's sidebar carries no hrefs and no thread-id attributes. That is why Pass B keeps `limit: 2` on a name lookup and stores `data.li_thread_id` — a stored id makes every later check a direct navigation that touches nothing else.
- **The per-lead check before a send is not a sweep.** Reading a lead's thread with `get_conversation` right before sending it a follow-up works and stays in place (it caught Eva Steortz and Lion Fludd on 2026-08-14 and correctly stopped two closing follow-ups). But it only ever runs on leads the run was already about to touch, so it can never surface a reply from a quiet lead. Do not let it stand in for Pass B.

**Open: Sales Navigator.** Unresolved as of 2026-08-14. Inbound **InMail** threads do appear in the regular inbox (an "InMail GLG | Paid Phone Consultation" thread showed up in Pass A, and the inbox has an InMail filter tab), so InMail is covered. What is unverified is whether threads started from a **Sales Navigator seat** appear on the regular `/messaging/` surface at all. The account has a seat and threads on it, but no example thread was identified to test against. Both tools only ever read `/messaging/`, and no tool on this MCP can reach `/sales/inbox`, so if SN-native threads are separate they are invisible to the whole pipeline and would need a second read path. Until someone settles it, assume SN threads are NOT visible and do not work a lead solely through Sales Navigator. **To settle it:** take one person you know you messaged from Sales Navigator, run `get_inbox({ limit: 50 })` and `search_conversations({ keywords: "<their name>", limit: 2 })`, and record here whether either sees the thread.

### 2. DRAFT

The day's new prospects (from the IG-first pipeline below, `search_people` on rotated queries, or handed over by the user): skip-check each candidate first (`list_outreach_leads({ search: "<handle>" })` — never re-work someone already in the pipeline), ICP filter (Job B), personalize the survivors (Job A), and queue everything with `queue_outreach_drafts`. Anchor-less prospects get queued with no anchor (→ held), never written generic. Rotate discovery searches via `get_outreach_state({ key: "query_rotation" })` (oldest `last_used` first, nulls first) and stamp them with `set_outreach_state` after.

Then ship the day's LinkedIn first touches, within whatever invite budget phase 1 left:

1. `approve_outreach_draft({ draftIds: [<the li_dm drafts just queued>], accountId: "digital_university" })`. Read `remainingToday` from the result — that is the real budget, not your own count. Anything refused comes back with its reason; act on it rather than retrying (a `held_needs_anchor` draft needs a real anchor, a `daily_cap_reached` one simply waits for tomorrow) and never try to route around a refusal.
2. Ship each approved one exactly as in phase 1's `li_dm` step: `connect_with_person`, verify `note_sent`, upsert + `log_outreach_touch`, then `mark_outreach_draft_shipped`.

`ig_dm` and `email` drafts are NOT approved here — they stay pending for the dashboard, and the digest nudges the human to the Review tab. Nothing else in this phase sends or touches the tracker.

### 3. REPORT

1. `set_outreach_state({ key: "daily_run", value: { at, perChannel: { discovered, kept, removed, drafted, held, shipped, shipErrors } } })` so the dashboard's Review tab shows the last run.
2. **Snapshot the funnel.** Get the stage counts from `get_outreach_lead_counts({ accountId: "digital_university" })` — one call, every lead counted, broken down per channel per stage with no page cap. Never tally `list_outreach_leads` pages for this: that tool returns a page, so a tally of it reports a floor as if it were a total (that is how the 2026-08-09 snapshot came out short — the account had 708 `li` and 513 `ig` leads behind a 500 cap). Map each channel's `byStage` onto the schema-v1 normalized stages and push with `upsert_outreach_metrics` (account, generatedAt, channels[]; `accountId: "digital_university"`). Idempotent on the snapshot date, so re-running the same day is safe.
3. `send_admin_report` digest: replies found (split by which pass found them — a reply that only Pass B or C caught is one the old inbox-only sweep would have missed, and that count is worth watching), how many watch-list leads went unchecked because Pass B hit its 40-thread cap, **the acceptance line (see below)**, follow-up 2s sent, new invites sent, drafts queued and held (nudge to the dashboard's Review tab), drafts rejected with their reasons, kept vs removed by ICP, demos built, stale invites to withdraw, ship errors.

   **The acceptance line always carries its denominator.** Four numbers, never fewer: degrees **checked** this run (due + sweep, stated separately), **accepted**, **unreadable**, and **`awaiting_accept` leads still unswept** (total in that state minus those carrying today's `li_degree_checked_at`). "Zero acceptances" on its own is banned wording: it reads as "nobody accepted" when the honest meaning is usually "we checked 12 of 83". Write "checked 12 of 83 pending invites, 0 accepted, 1 unreadable, 71 still unswept" instead. Followed by the accepted leads that got follow-up 1, and any acceptance held back by the ramp or by stale copy.

### Volume + account safety (hard limits)

This is browser automation on Alfonso's real LinkedIn account, which LinkedIn's terms prohibit. Volume discipline is what keeps the account alive:

- Max **15 connection requests** and **25 messages** per day — those are ceilings, not budgets. The worker's warmup ramp at `https://li-worker.tribed.io/health` is the real budget whenever it is lower (invites and messages both ramp; `login.sh` deletes `warmup.json`, so any re-login resets the ramp to its floor). Read it before planning sends.
- **One run per UTC day, not per local day.** The ramp counters are UTC-keyed, and an early-morning run in WITA lands in the PREVIOUS UTC day — two runs the same local day can double the daily ramp, and `approve_outreach_draft`'s `remainingToday` will not catch it. Before sending, check when the last run actually happened (`get_outreach_state({ key: "daily_run" })` — its `value` is a JSON STRING, parse it and trust `updatedAt`).
- Calls queue through one browser session; never try to parallelize, and expect long runs.
- If a tool errors with anything that smells like a checkpoint, captcha, or verification wall, STOP the run immediately and tell the user. Do not retry.
- Sends happen only inside the daily run or on an explicit ask.
- Other channels: max 25 ig_dm handoffs and 50 emails per run. Approved drafts over a cap wait.
- **End every LinkedIn leg with `close_session`**, run finished or run aborted — the automated browser is never left alive after the work is done.

### Instagram preflight — the two-step gate (run before any IG leg)

Instagram sending is gated per account, and the gate is TWO steps: step one reads the hold, step two tests whether the hold is still true. Step two is the one that gets skipped, and skipping it is expensive. On 2026-08-17 a 72h account-wide hold on `digital_university` rested on three profiles, two of them had already recovered, and the stale block idled four days of Instagram sending against a completely free budget and a 45-lead pool.

**Step 1 — read health.** `get_instagram_session_account_health({ accountId })`. Cheap, no browser. Read `configuredAccounts` off the response and run this gate once per account, passing `accountId` explicitly on every call.

**Step 1a — a MISSING `queueingBlocked` OR `actionControls` IS A BLOCK, NOT A PASS.** Both fields are unconditional in a current MCP build, so their absence means the build serving this run predates the gate and cannot see a hold that is already recorded server-side. That is exactly what happened on 2026-08-17: health came back yellow/active with neither field while the account was three profiles deep into a recorded suppression. If either field is absent, treat that account as blocked, queue nothing for it, do not run step 2 (a build without the fields has no verify tool either), and report "stale MCP build, gate unavailable" by name so `mcp/dist` gets rebuilt. Every worktree carries its own `mcp/dist`, so a stale build is the normal case, not the exception.

**Step 1b — `queueingBlocked: false` and `overall` not red.** Proceed. Nothing else to do.

**Step 1c — red for a reason that is not action controls** (`sessionStatus` not "active", engagement disabled, any `queueingBlockedReason` other than `action_controls_suppressed`). Stand the account down and report it. Step 2 does not apply: re-probing profiles cannot revive a dead session, and a dead session has to be re-authed by hand.

**Step 2 — `queueingBlocked: true` with `queueingBlockedReason: "action_controls_suppressed"` is NOT a stand-down on its own.** Call `verify_instagram_action_controls({ accountId })` BEFORE accepting the hold. An account-wide hold is assembled from observations days apart, a suppression is per-target and recovers on its own, and nothing else ever re-tests the earlier profiles. The tool opens each handle behind the hold through the account's own browser profile and proxy and READS the action row. It never clicks, never opens an attempt, never touches a quota and never records a suppression, so it is free to run against a blocked account and it does NOT restart the block's window. Health itself names it in `actionControls.verifyBeforeHonouring` when a hold is live. It is slow by nature: a browser launch plus one navigation per handle, up to a couple of minutes.

**Honour the hold only if `after.suppressed` is still true.** Read the result:

- `after.suppressed: false` — the hold had no live evidence behind it. Queueing is clear, the run proceeds normally for that account. `recovered` names the handles that render controls again and `droppedObservations` counts what was pruned. Say so in the report: a hold that verified away is a fact worth having.
- `after.suppressed: true` — the hold is real. Stand the account down, queue nothing, send nothing, retry nothing (every attempt restarts the window). Report by name with `after.clearsAt`, `after.distinctProfiles`, `after.scope` and the `stillSuppressed` handles.
- Verify errored, or the tool does not exist on this build — the hold STANDS. Only positive proof relaxes a hold, never a failure. Same fail-closed rule as step 1a.
- `skipped` is non-empty (more than `MAX_VERIFY_HANDLES` profiles behind the hold) — those observations are unchanged and still count toward `after`, so a hold that survives may only be surviving on unprobed evidence. Run verify again to cover them before treating the block as confirmed.

A run that stands Instagram down must state which step did it: no gate fields (stale build), red for a session reason, or a hold that survived verification. "IG blocked" with no step named is not a report.

### Combined cross-channel daily pipeline (IG-first)

The daily job targets one person across all three channels. Start on Instagram and fan out. All of this happens in the DRAFT phase — every message rides the queue.

**Run the Instagram preflight above first.** If it stands the account down, the IG steps below do not run for that account: no discovery, no IG drafts, no IG handoffs. LinkedIn and email are unaffected by an IG hold and still run.

1. **Discover on Instagram** (WebSearch + Apify `figue/instagram-profile-scraper`): qualified coaches 10k-500k followers with a bio link. Capture `external_url` (bio link), `business_email`, bio, captions. Two Apify traps (2026-08-18): **hashtag search is broken and there is no follower-list actor** — discovery comes from WebSearch/name lists fed to the profile scraper, not from Apify-side search. And always pass **`resultsLimit >= 12`** on posts: with fewer, pinned posts fill the window and an active account reads as dead (stale-looking latest posts), which wrongly fails the liveness check.
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

- **Never infer a LinkedIn acceptance.** Only a read connection degree of "· 1st" counts. A send, a composer, a `composer_unavailable`, and a reply all prove nothing. Record `li_degree` + `li_degree_checked_at` + `li_degree_source` beside every `li_accepted_at`, and never queue follow-up 2 for a lead whose stored `li_degree` is unset or not "1st". See the acceptance-proof rule above.
- **Never honour an Instagram hold without verifying it.** `queueingBlocked: true` is step one of a two-step gate: call `verify_instagram_action_controls` and stand the account down only if `after.suppressed` is still true. The check is read-only on Instagram and does not restart the window. A missing `queueingBlocked` or `actionControls` field is a stale build, so it blocks too. See the Instagram preflight above.
- Never work an out-of-ICP lead.
- Never ship a message that fails rule zero — queue it as held (no anchor) instead.
- Unattended runs never send anything that was not queued as a draft first; the queue is the only path. The run may approve its own LinkedIn connection notes within the server-enforced bounds, and only those — IG DMs and email still wait on a person in the dashboard.
- Reply-check before every send; never message someone who already replied — advance their stage instead.
- Sample-then-bulk on any first interactive run against a new batch.
- The connection note has no CTA and no price; pre-demo follow-ups never ask for a call.
- Respect the daily caps. When in doubt, send less.
