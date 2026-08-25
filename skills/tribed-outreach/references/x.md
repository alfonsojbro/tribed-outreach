# Mode 8 — X (Twitter) outreach

Register is the same as Instagram: texting a friend you respect. X is even more
allergic to polish than IG — a message that reads like marketing dies on sight.

Read the rail split below before anything else. It decides which tools you are
allowed to call and which safety rule applies to which action.

## Two rails, two engines, one account

X outreach runs on TWO independent rails. They may act as the same X account,
but they share no code, no gate and no cap ledger. Confusing them is the main
way this leg goes wrong — it is what produced both the 2026-08-24 idle queue and
the 2026-08-25 deliberately-empty one.

| | Hosted **x-drip** | **Session worker** |
|---|---|---|
| Code | `mcp/src/repos/xDrip.ts` → `repos/x.ts` | `mcp/src/repos/xWorker.ts` (`x-worker/`) |
| Engine | X **API v2 over OAuth** (`XConnection`) | logged-in **Playwright browser** |
| Runs | hourly job, unattended, 13–23 UTC | only when a run or operator calls a tool |
| Public comment | `x.createComment` → `POST /2/tweets` | `reply_x_post_session` |
| Like | `x.reactToPost` | — |
| Follow | `x.followUser` | `follow_x_profile_session` |
| DM | `x.sendDm` (needs `X_DRIP_DM_ENABLED`) | `ship_x_outreach_draft` / `send_x_dm_session` |
| Reads the DM inbox | **cannot** | `read_x_session_inbox` |
| `messageAvailable` probe | cannot | `view_x_profile` |
| Cap ledger | **none** | `get_x_session_account_health` → `caps` |
| Gate | `X_DRIP_ENABLED` only | `caps.gates.browserAdapterReady`, `sessionStatus` |

**The drip owns the daily outbound leg**, as of 2026-08-24. Like, public
comment, follow, and — when `X_DRIP_DM_ENABLED` is on — the DM all ship from the
drip, off copy the morning run staged onto the lead. Staging it is **Fill 4 in
references/pipeline.md**, and that is the only place the outbound X leg is
driven from now.

**The session worker owns what the API physically cannot do.** X API v2 cannot
read the Message-requests tray where cold replies from strangers land, and
cannot tell you whether a stranger accepts DMs at all. `read_x_session_inbox`
and `view_x_profile` are the whole reason the browser rail still exists. That is
why the rail is **reduced to reply triage and probing**, not retired.

## The reply ban covers `reply_x_post_session` ONLY

`reply_x_post_session` drives the browser: it opens the post, types into the
reply box, and clicks. **That path has never been verified against live X** (as
of 2026-08-25, only the follow path and the DM pre-click guard are). Do not call
it. Do not post a public reply through the session worker. A human lifts this
once the path is verified on the canary.

**The drip's public comment is a different engine and is NOT under the ban.**
`data.x_comment` is posted by `x.createComment`, an authenticated
`POST https://api.x.com/2/tweets` carrying a reply reference, on the OAuth
connection. No browser, no selectors, no `x-worker`, no `browserAdapterReady`.
The codebase says so in its own tool description: `reply_x_post_session` is
documented as "distinct from `comment_on_x_post`, which replies AS a coach's
connected account via the API". Nothing about the unverified browser reply
applies to the drip.

**So Fill 4 staging is NOT on hold.** Stage X leads with `x_state: "to_touch"`,
`nextActionAt` today and `data.x_comment`, normally. The 2026-08-25 run withheld
`x_state` and `nextActionAt` from its 6 new channel-`"x"` leads because it read
the reply ban as covering the drip. That reading was wrong, and those leads are
invisible to the drip until the fields are backfilled — do that on the next run.

## What the split actually costs you

Two consequences follow from the table, and neither is obvious:

**1. The drip has no health gate.** `runXDrip` never calls
`get_x_session_account_health`, never reads `caps`, and never checks
`browserAdapterReady`. **A disarmed session worker does not stop the drip.** Its
only pacing is its own constants: 5 leads per tick, 40 per run, a 25% tick skip,
shuffled lead order, a 20–90s pause between leads, inside the send window. So
"the X rail is disarmed" is only ever a statement about the browser rail — never
report it as the X leg being down.

**2. The two ledgers do not know about each other.** The drip's writes charge
nothing against `caps.reply`, `caps.follow` or `caps.dm`. If the OAuth
connection on `XConnection/digital_university` is the same handle as the session
worker's login, then one real X account is being worked by two rails that each
believe they are inside budget. **Confirm the identity before raising volume on
either rail**: read the connected username with `x_connection_status`
(`communityId: "digital_university"`) and compare it to the session worker's
account handle. If they match — the expected case, both being the founder's
`@alfonsojbro` — then the drip's 40-per-run is spending the same real-world
budget the session `caps` describe, and the session rail must stay on reads.
This is an open item, not a settled one: nothing in the code enforces it.

## The constraint that shapes DM work: most people cannot be cold-DMed

X only delivers a DM to a stranger when that account has "allow message requests
from everyone" ON. Most do not. A handle alone never tells you, and a DM to a
closed account is not a rejection you can retry — the control simply is not on
the page. The drip degrades gracefully here (a closed-DM recipient 403s and is
logged as a skip, not a failure), but a skipped DM is still a wasted slot.

So the order is always **check, then spend**:

1. `view_x_profile({ accountId, username })` — read-only, costs no cap, never
   clicks. Returns `messageAvailable`.
2. `messageAvailable: false` → **do not stage `data.x_dm` and do not queue an
   `x_dm` draft.** The DM cannot land. Stage the lead for the drip's public
   touch only (like + comment + follow), which is where the warming happens
   anyway. Stamp `data.x_dm_available: false` with the date so the next run does
   not re-probe the same profile all week.
3. `messageAvailable: true` → DM-workable. Queue the draft.

It also returns `exists`, `suspended`, `protected`, `alreadyFollowing`. A
suspended or protected account is dead for outreach; mark the lead lost.

## Warm-up is behavioural, not just volume

Ramps only throttle OUTBOUND actions. That is half of a warm-up. The other half
is that the account posts and engages genuinely, and outreach is a small
fraction of what it does.

What matters is the RATIO, not the history. An account with years of activity
that goes quiet and starts only DMing strangers looks exactly like a bought
account being switched on. So:

- **Precondition, checked per run, not once.** The account must still be posting
  and replying organically. Alfonso confirmed `@alfonsojbro` is in regular
  personal use (2026-08-23); if he stops, the leg shrinks rather than carries on.
- Cold outreach stays the minority of the account's daily actions. The drip's
  40-lead ceiling is a like + comment + follow each — roughly 120 writes on a
  full run. That is NOT automatically the minority of a personal account's day.
  Raise volume and organic activity together, or not at all.
- Never let a day be outreach-only.

The numeric platform ceilings (400 follows/day, 500 DMs/day) are the wrong
yardstick. The binding constraint is looking inauthentic, and mass cold contact
is the single most-reported behaviour on X.

## Discovery — WebSearch first, qualify second

X's own API has no search on our tier (handle lookup, follow, DM, post only).

**Find with WebSearch, not with a scraper's search.** Same shape the Instagram
routine settled on, for the same reason: scraper search surfaces are unreliable
while a web search finds selling evidence directly and costs nothing. Verified
working for X on 2026-08-23.

Query the indexed profile pages, targeting people who SELL coaching rather than
people who merely talk about fitness:

```
"x.com" online fitness coach "1:1 coaching" spots open clients
"x.com" nutrition coach "coaching clients" transformation
"x.com" wellness coach "dm me" coaching -crypto -forex -trading
```

Always carry the exclusions: X is thick with crypto/forex/dropshipping
"coaches" who are not our ICP.

**Then qualify, because search has no sense of liveness.** Google will happily
present a coach whose last post was two years ago, and handles inferred from
result titles are wrong maybe a fifth of the time. `view_x_profile` is free,
read-only and charges no cap — run it on every candidate before spending
anything.

The cross-channel bridge is usually cheaper than cold discovery: bio links on IG
and LinkedIn leads already in ICP often carry the X handle.

**Extraction rule.** One lead per distinct person, never one per post. Keep the
post or bio line that proved they sell coaching as the anchor candidate.

**This ICP fit is still unproven.** X's coach population skews more info-product
and B2B, with fewer hands-on 1:1 coaches. Run the ICP filter (Job B in
pipeline.md) hard and report the hit rate rather than assuming the queries work.

**Apify is the optional deepener, not the finder.** `apidojo/tweet-scraper`
($0.0004/tweet) can pull recent tweets for a sharper anchor or a recency check.
Two traps: the generic `call-actor` MCP tool ships with an empty parameter
schema, so object inputs may not bind from some clients — use an actor-specific
Apify tool or the REST API with `APIFY_TOKEN`. And never let a paid scrape stand
in for the free `view_x_profile` check.

## Comment copy — write about the person, never about one post

`data.x_comment` fires against whatever their MOST RECENT post is **at send
time**, not the post you read when you staged it. The drip resolves the latest
post itself. So write the comment about the person and their body of work; a
comment about one specific post lands as a non sequitur under something newer.

Follow the outreach comment rule: one or two phone-reply sentences reacting to
ONE thing they actually say. Never the three-beat critic essay, never "most
people miss this", never an aphorism.

## DM copy

Same rules as the IG cold DM (references/instagram.md): the named app, the
gift-first hook, rule zero in line one, no self-introduction paragraph, max two
short paragraphs. Use the same format library (`aida` / `pas` / `bab` /
`reveal`) and tag the format on the lead.

X-specific: keep it shorter than the IG equivalent. The DM lands in a narrow
column and a wall of text reads as automation. Two or three sentences is plenty.

The ask stays the small named one: "i made an initial version of <AppName>, can
i send it over?" — never "want me to build yours?".

### Hard rule: NO LINK in a cold DM. Ever.

A link in a cold opener is one of the strongest spam triggers on X, for the
automated filters and for the human "report" button alike. It can lock the
account's DMs outright while the numeric daily cap is nowhere near spent.

The ask already does the work without one: it asks PERMISSION to send the app
rather than pasting it. That is not a stylistic preference, it is the safety
property — the link goes in the SECOND message, after they reply and there is an
actual conversation.

This applies to bare domains and shorteners too, not just full URLs. A draft
carrying a link in the first touch is rejected in review, not edited into shape
after approval.

## Draft-first survives the rail change

The drip does not invent DM copy. It sends `data.x_dm`, and the only sanctioned
way that field gets populated is through the review queue:

```
queue_outreach_drafts  (kind "x_dm", slot "dm", anchor required)
        ↓
a HUMAN approves in the dashboard Review tab
        ↓
the approved copy is stamped onto the lead as data.x_dm
        ↓
the drip sends it, when X_DRIP_DM_ENABLED is on
```

`approve_outreach_draft` is LinkedIn-only and refuses an `x_dm`, by design — do
not work around it. Never write `data.x_dm` from unapproved copy: that bypasses
the human gate the queue exists to enforce. A lead with no usable anchor gets
its draft queued with `anchor` omitted so it lands as `held`, rather than
shipping something generic.

`ship_x_outreach_draft` remains the session-worker shipping path and still
works; it is the manual escape hatch for a single approved draft, not the daily
leg. The direct tools (`send_x_dm_session`, `follow_x_profile_session`) are for
Alfonso's explicit one-target commands only.

## The daily split: who does what

**The morning pipeline run (Fill 4)** stages the queue: source and skip-check
handles, probe `messageAvailable`, author `data.x_comment` (and `data.x_dm` from
an approved draft), then upsert with `channel "x"`, `externalId` = the handle
lowercased, `data.x_username`, `x_state: "to_touch"`, `nextActionAt` today. Drip
pace is ~5 leads a tick and 40 a run, so a couple dozen due leads keeps it fed
without flooding it. Report cap vs staged vs sent, and report a zero with its
denominator.

**The `tribed-daily-x` scheduled task** does NOT stage and does NOT send
outbound. It runs reply triage: read the inbox including the requests tray, take
repliers off automation, draft Mode 2 replies for Alfonso, and refresh
`messageAvailable` for leads the drip could not DM. It exists because the API
cannot see that tray.

Session-worker gate readings, for the triage task's reads:

| Reading | Meaning |
|---|---|
| tool missing from the list | MCP build predates the X rail. Stand down, report it. |
| `caps.gates.browserAdapterReady: false` | Browser adapter disarmed. Reads still work; no session action can send. **This does not stop the drip.** |
| `caps.gates.sessionStatus` ≠ `"active"` | Session needs a re-login (`login-plain.sh`, then `check-session.sh`). |
| `capsError` present | The worker reported no usable number. Spend nothing; never guess a cap. |

Reading the inbox honestly: conclude "no replies" ONLY when `partial` is false.
An empty list with `partial: true`, or `ok: false`, means the inbox is UNKNOWN,
not empty. Report it that way.

On any inbound last message: take the lead off automation immediately —
`log_outreach_touch` with `advanceTo: "replied"`, `automated: false`, which also
clears `nextActionAt` so the drip stops touching them — and draft a Mode 2
reply. Replies are never auto-sent.

Session caps ramp with the SESSION's age, not the account's, and carry a
deterministic ±20% daily jitter, so a ceiling of 20 legitimately reads as 18 or
21 on a given day. A re-login resets the ramp to week one. Read them off `caps`
every run; never carry yesterday's, never reuse Instagram's, never sum them.

| week | DMs | follows | replies |
|---|---|---|---|
| 1 | 3 | 5 | 5 |
| 2 | 8 | 10 | 12 |
| 3 | 14 | 18 | 20 |
| 4+ | 20 | 25 | 30 |
