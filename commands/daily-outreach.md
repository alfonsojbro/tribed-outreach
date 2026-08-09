---
name: daily-outreach
description: Run the daily Tribed outreach pipeline — replies, follow-ups, and approved drafts via our LinkedIn MCP, then queue new personalized drafts, self-approving LinkedIn connection notes and routing everything else to the NobleAdmin dashboard.
---

Run the Tribed daily outreach pipeline. Follow references/pipeline.md (Job C: SHIP → DRAFT → REPORT) exactly. This is an unattended run: every first-touch message is written to the review queue on the Tribed MCP (`accountId "digital_university"`) before anything sends, so the exact copy is always on the record. LinkedIn connection notes (`li_dm`) may then be approved by the run itself with `approve_outreach_draft` under Alfonso's direct-send authorization (2026-08-06) and shipped in the same run, up to the server-enforced daily cap. IG DMs and email still ship only after a human approves them in the dashboard (admin.tribed.io, Messaging > Review). LinkedIn sends through our self-hosted `linkedin` MCP; the tracker is the sequencer.

Preflight: one `get_my_profile` call. On a session/login error, stop and tell the user to re-auth (`uvx mcp-server-linkedin@latest --import-from-browser`).

1. SHIP (order matters — replies first):
   - Replies: `get_inbox`, match against open li leads, advance stage + store `data.li_thread_id`, draft Mode 2 replies for the user (never auto-send). Interested + no demo → build it (Job D, max 5/run).
   - Acceptances: due leads awaiting acceptance → if connected, `send_message` `data.li_followup_1`, schedule follow-up 2 (+3-4d); else bump a day; 21+ days pending → lost.
   - Due follow-up 2: verify no reply via `get_conversation`, send `data.li_followup_2`, archive-if-quiet in 14d.
   - Approved drafts (`list_outreach_drafts({ status: "approved" })` + "error" retries): li_dm → `connect_with_person` with the note + upsert lead with follow-ups in `data`; ig_dm → upsert to the VA's list (owner from shipTarget); email → push to the Instantly campaign. `mark_outreach_draft_shipped` per draft. A draft that can never ship (pending invite already out, profile gone, replied elsewhere) → `reject_outreach_draft` with a concrete `reason`; never reject an `approved` draft.
   Caps: 15 connection requests, 25 messages, 25 IG handoffs, 50 emails per run. STOP the LinkedIn leg on any checkpoint/captcha/verification error and report it.

2. DRAFT — IG-first cross-channel discovery plus rotated `search_people` sourcing (rotation via `get_outreach_state("query_rotation")`). Skip-check each candidate (`list_outreach_leads({ search })`), ICP-gate (2,000+ followers OR creator/influencer flag; Apify `harvestapi/linkedin-profile-scraper` for bulk), personalize survivors in the Tribed voice (rule zero: one real anchor; connection note + case-study follow-up + video follow-up), and queue with `queue_outreach_drafts` (kinds li_dm / ig_dm / email). Anchor-less prospects queue with anchor omitted (held) — never write generic.

   Then ship the day's LinkedIn first touches, within whatever invite budget phase 1 left: `approve_outreach_draft({ draftIds: [<the li_dm drafts just queued>], accountId: "digital_university" })`, read `remainingToday` from the result as the real budget, then ship each approved one exactly as in phase 1's li_dm step. Anything refused comes back with its reason — act on it, never route around it. `ig_dm` and `email` drafts are NOT approved here; they stay pending for the dashboard.

3. REPORT — `set_outreach_state({ key: "daily_run", ... })` with per-channel counts, snapshot the funnel with `get_outreach_lead_counts` → `upsert_outreach_metrics` (never tally `list_outreach_leads` pages — that reports a floor as a total), and `send_admin_report`: replies, sends, invites, drafts queued/held (nudge to the Review tab), drafts rejected with reasons, demos built, stale invites, errors.

Do not work out-of-ICP prospects. The run may approve its own `li_dm` drafts and nothing else — `approve_outreach_draft` refuses every other kind, and it can never un-approve, rewrite, or resurrect a draft a human decided on.
