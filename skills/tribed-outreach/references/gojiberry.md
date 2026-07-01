# Mode 7 — Gojiberry ops: personalization + daily pipeline

Gojiberry finds LinkedIn leads by intent and runs multi-step campaigns. This mode covers two jobs: (A) writing personalized connection notes + follow-ups onto leads, and (B) the daily pipeline that filters new leads to ICP and personalizes them automatically. Requires the Gojiberry MCP (contacts/campaigns/agents) and the Apify MCP (follower enrichment).

## How Gojiberry is wired (know this before touching anything)

- **Agents** find leads and drop them into a **list**. **Campaigns** do the outreach and are bound to a list (list membership = campaign membership).
- A campaign is an ordered `steps` array. Our two campaigns share this shape:

  | stepNumber | type | meaning |
  |---|---|---|
  | 0 | invitation | LinkedIn connection request (this is where the connection *note* goes) |
  | 1 | email | cold email |
  | 2 | message | 1st LinkedIn message after connect (follow-up 1) |
  | 3 | visitProfile | profile view, no copy |
  | 4 | email | email follow-up |
  | 5 | message | 2nd LinkedIn message (follow-up 2) |
  | 6 | email | email follow-up |

  Message steps default to `messageMode: "ai"` with an empty template, so with no action they go out AI-generated (and the invitation goes out with **no note at all**).
- **Per-contact personalization is the lever.** The MCP has no campaign-template editor, but `update_contact` accepts `personalizedMessages: [{ stepNumber, content }]`. Content set for a step overrides the AI/template for that one contact on that step. This is how we inject a real connection note (step 0) and hand-written follow-ups (steps 2, 5) — and emails (steps 1, 4, 6) if enabled.
- There is **no hard delete** for contacts. To take a lead out of outreach, use `remove_contacts_from_list` (drops them from the list and therefore the campaign). The record still exists.

## Job A — write personalized messages onto leads

For each in-ICP lead:

1. Gather the anchor. Best source is the scraped LinkedIn profile (`headline`, `about`, a named program/book, `topSkills`, current role). Apply **rule zero** — one real detail, never the niche.
2. Write, in the Tribed voice:
   - **step 0 — connection note.** Mode 4A rules: under ~280 characters (confirm the count), one line on their detail, one line hinting something was built for them, no price, no features, no explicit CTA.
   - **step 2 — follow-up 1 (light bump).** Mode 3 Touch 1: one or two sentences, the thing exists and is theirs, door open. One soft question.
   - **step 5 — follow-up 2 (new angle / soft close).** Mode 3 Touch 2 or 3: a fresh reframe or the takeaway, lightest CTA or none.
   - (Optional) **steps 1/4/6 — emails.** Mode 6 (email.md), only if email is enabled for the run.
3. Rotate skeletons and CTAs across leads (see core). Never reuse a line across contacts.
4. Write it: `update_contact({ id, personalizedMessages: [{stepNumber:0, content}, {stepNumber:2, content}, {stepNumber:5, content}] })`.
5. Verify on the first lead of a batch by re-reading with `get_contact` before doing the rest.

Always sample 3–5 leads for the user's approval before writing a whole list.

## Job B — the ICP filter (run before personalizing a new list)

Gojiberry's own fit score does NOT measure audience (it grades role/industry and tends to sit ≥0.80 for everyone), so filter on real audience signal:

1. Pull the list with `list_contacts({ listId })` (paginate; responses are large — page at ~100 and keep bulk data out of context).
2. Enrich followers with Apify actor `harvestapi/linkedin-profile-scraper`, mode `"Profile details no email ($4 per 1k)"`, passing the contacts' `publicIdentifiers`. It returns `followerCount`, `connectionsCount`, `creator`, `influencer`. Cost ≈ $0.004/lead.
3. **Keep** a lead if `followerCount >= 2000` OR `creator === true` OR `influencer === true`. Everyone else is out of ICP.
4. `remove_contacts_from_list` the out-of-ICP leads (batch by listId).

## Job C — the daily IG + LinkedIn pipeline

Run once per day after the agents have sourced new leads. For each active list / campaign:

1. **Find the day's new leads:** `list_contacts({ listId, dateFrom: <today> })` (or diff against yesterday's ids).
2. **ICP filter** them (Job B). Remove the ones that fail.
3. **Personalize** the survivors (Job A): connection note (step 0) + follow-up 1 (step 2) + follow-up 2 (step 5); add emails (1/4/6) if email is on for that campaign.
4. **Mark ready** if needed (`readyForCampaign`) so Gojiberry enrolls them.
5. **Report:** counts of new leads, kept vs removed, personalized; flag any lead with no usable anchor for manual review (don't ship a vague message).

### Channel split — LinkedIn vs Instagram

**Gojiberry is LinkedIn (+ email) only.** All agents source LinkedIn profiles and campaigns run invitation/message/email steps — there is no Instagram in Gojiberry. So the two channels are handled in different systems:

- **LinkedIn → straight into Gojiberry.** Do Jobs A–C above: ICP-filter, then write step 0 / 2 / 5 (and emails if enabled) with `update_contact.personalizedMessages`.
- **Instagram → into the Tribed MCP tracker for the VA.** IG DMs are not sent by an automation; IG leads live in the Tribed outreach tracker (the Tribed MCP: `list_outreach_leads` / `add_outreach_lead` / `upsert_outreach_leads` / `update_outreach_lead`, `accountId "tribed"`), and the VA sends them by hand. For each IG lead:
  1. Anchor comes from the profile/feed (Mode 1 / instagram.md), not scraped LinkedIn fields. IG has no connection-note step — start at the DM.
  2. Draft in the Tribed voice: a cold DM plus two follow-ups (Mode 3). Write them into the lead's `data` bag, e.g. `data.ig_dm`, `data.ig_followup_1`, `data.ig_followup_2`.
  3. Set `channel: "ig"`, `owner` to the VA handling IG (e.g. "Martin"), a clear `nextAction` ("Send IG DM"), and `stage: "top"` so it surfaces on the VA's list. Upsert with `upsert_outreach_leads`.
  4. If a lead has no usable anchor, leave the message fields empty and flag it — don't write a generic DM.

Keep daily LinkedIn send volumes within safe limits. On both channels, hold anchor-less leads rather than send generic.

### Combined cross-channel daily pipeline (IG-first)

The daily job targets one person across all three channels. Start on Instagram and fan out:

1. **Discover on Instagram** (WebSearch + Apify `figue/instagram-profile-scraper`): qualified coaches 10k-60k followers with a bio link. Capture `external_url` (bio link), `business_email`, bio, captions.
2. **IG copy → the VA.** Draft a gift-first DM + two follow-ups (Mode 1 + Mode 3). Upsert to the Tribed pipeline with `accountId "digital_university"` (this is the record; NOT "tribed"), `channel "ig"`, `owner "Martin"`, messages in `data`. IG is never auto-sent — the VA sends by hand.
3. **High-confidence match to LinkedIn.** The reliable bridge is the **IG bio link**: if it's a `linkedin.com/in/` URL use it directly; if it's a personal site/Linktree, fetch it and read the LinkedIn link off it; else search by exact name + a corroborating signal (same company / same website domain). If nothing confidently matches, keep the lead **IG-only** — never guess, a wrong match messages the wrong person.
4. **Matched → Gojiberry.** ICP-gate (2,000+ or creator), then `create_contact` + `add_contacts_to_list`, and write step 0/2/5 with `personalizedMessages`.
5. **Email → Instantly.** Enrich the email (IG business_email, bio, or LinkedIn email search), write a cold email (Mode 6), and push to the Instantly campaign (auto-send). Instantly is the email sender, not Gojiberry's email steps.
6. **Notify Martin** (WhatsApp) and report per channel.

Canonical routing: IG → digital_university pipeline (VA sends); LinkedIn → Gojiberry campaign; email → Instantly. The digital_university pipeline (admin.tribed.io) is the dashboard of record.

## Guardrails

- Never enroll an out-of-ICP lead.
- Never ship a message that fails rule zero — hold it for manual review instead.
- Sample-then-bulk on any first run against a new list.
- The connection note (step 0) has no CTA and no price; pre-demo follow-ups never ask for a call.
