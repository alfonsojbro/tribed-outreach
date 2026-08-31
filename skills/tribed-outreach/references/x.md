# Mode 8 — X (Twitter) outreach

Register is the same as Instagram: texting a friend you respect. X is even more
allergic to polish than IG — a message that reads like marketing dies on sight.

Read the rail split below before anything else. It decides which tools you are
allowed to call and which safety rule applies to which action.

## THE PUBLIC COMMENT LEG IS DEAD — platform ban, not our bug (2026-08-30)

X restricted programmatic replies at the platform level: `POST /2/tweets` with
a reply reference returns **403 not-authorized-for-resource** ("You can only
reply to or quote posts where you are mentioned or are the author") unless the
post's author has @mentioned or quoted the replying account first. It applies
to **Free, Basic, Pro AND Pay-Per-Use** — every tier we can buy; only
Enterprise is exempt. Announced by @XDevelopers
(x.com/XDevelopers/status/2026084506822730185, "X API v2 Update: Addressing
LLM-Generated Spam" on devcommunity.x.com). No scope, no reconnect, no
purchasable upgrade changes it. The receipt: the drip's live attempts on
@youbfit (lead Jb3EG9GEdMZrAY1u7gcg, digital_university) 403'd identically on
2026-08-28 and 2026-08-29 with `tweet.write` present and the pin resolved.

The browser rail's `reply_x_post_session` stays banned separately (unverified
path; its canary failed). So between the two rails there is NO working public
comment, and **the X outbound leg is follow + DM, full stop.**

What this retired, all removed from `xDrip.ts` on 2026-08-30:

- the comment step, the warm like that rode with it, and the `caps.reply`
  charge;
- the whole pinned-post machinery: `data.x_comment`, `x_comment_post_url` /
  `x_comment_post_id` / `x_comment_post_at`, `x_comment_needs_fresh` /
  `_block_reason` / `_blocked_post_id`. These fields are now INERT on a lead —
  the drip never reads them, an old pin block no longer keeps a lead out of
  the queue, and Fill 4 must stop staging them;
- the 7-day comment window as a hard liveness gate on staging. Liveness is
  still an ICP-quality signal (a dormant account reads nothing), but it no
  longer blocks the touch, because there is no comment to pin.

Comment-shaped warmth, if wanted at all, is a HUMAN action in the X app on
Alfonso's phone — never a tool call. Do not re-derive this monthly: check the
dated announcement above before assuming X has relaxed it.

## Two rails, two engines, one account

X outreach runs on TWO independent rails. They act as the SAME X account —
`@alfonsojbro`, confirmed on both sides 2026-08-25 — but they share no code, no
gate and no cap ledger. Confusing them is the main way this leg goes wrong — it
is what produced both the 2026-08-24 idle queue and the 2026-08-25
deliberately-empty one.

| | Hosted **x-drip** | **Session worker** |
|---|---|---|
| Code | `mcp/src/repos/xDrip.ts` → `repos/x.ts` | `mcp/src/repos/xWorker.ts` (`x-worker/`) |
| Engine | X **API v2 over OAuth** (`XConnection`) | logged-in **Playwright browser** |
| Runs | hourly job, unattended, 13–23 UTC | only when a run or operator calls a tool |
| Public comment | **DEAD** — X 403s API replies on every buyable tier (see top) | **banned** (`reply_x_post_session` unverified) |
| Like | retired with the comment leg | — |
| Follow | `x.followUser` | `follow_x_profile_session` |
| DM | `x.sendDm` (needs `X_DRIP_DM_ENABLED`) | `ship_x_outreach_draft` / `send_x_dm_session` |
| Reads the DM inbox | **cannot** | `read_x_session_inbox` |
| `messageAvailable` probe | cannot | `view_x_profile` |
| Liveness (newest OWN post) | cannot | `view_x_profile` |
| Cap ledger | **none** (must borrow the worker's — see below) | `get_x_session_account_health` → `caps` |
| Gate | `X_DRIP_ENABLED` only | `caps.gates.browserAdapterReady`, `sessionStatus` |

**The drip owns the daily outbound leg** by design, as of 2026-08-24. The leg
is the follow and — when `X_DRIP_DM_ENABLED` is on — the DM, off copy the
morning run staged onto the lead. Staging it is **Fill 4 in
references/pipeline.md**, and that is the only place the outbound X leg is
driven from now.

**The session worker owns what the API physically cannot do.** X API v2 cannot
read the Message-requests tray where cold replies from strangers land, and
cannot tell you whether a stranger accepts DMs at all. `read_x_session_inbox`
and `view_x_profile` are the whole reason the browser rail still exists. That is
why the rail is **reduced to reply triage and probing**, not retired.

## Both reply paths are now closed (history of the two bans)

`reply_x_post_session` drives the browser: it opens the post, types into the
reply box, and clicks. That path was never verified against live X and its
canary failed. Do not call it.

The drip's API comment (`x.createComment` → `POST /2/tweets` with a reply
reference) was a different engine and was briefly believed workable — until X
closed it platform-wide (top section). Between 2026-08-25 and 2026-08-30 this
file said the API leg was open; the two days of live 403s proved otherwise.

**Fill 4 staging is NOT on hold — it just stages less.** Stage X leads with
`x_state: "to_touch"`, `nextActionAt` today, and (for DM-workable leads) an
approved `data.x_dm`. Do NOT stage `data.x_comment` or its pin fields any
more; the drip ignores them.

## What the split actually costs you

Two consequences follow from the table, and neither is obvious:

**1. The drip had no health gate. FIXED 2026-08-25.** `runXDrip` now reads
`xWorker.readAccountCaps(X_DRIP_SESSION_ACCOUNT)` — the same `/health` behind
`get_x_session_account_health` — before it spends anything, and:

- an unreadable ledger **throws** (the job runner logs and alerts); it never
  guesses a cap;
- `enabled`, `armed`, `browserAdapterReady` and `sessionStatus` all have to be
  open, and a gate `/health` did not report counts as closed. **A disarmed
  session worker now stops the drip too**, so "the X rail is disarmed" is a
  statement about BOTH rails;
- follows charge `caps.follow` and DMs `caps.dm`, each capped at its own
  `remainingToday` (`caps.reply` is no longer charged — no comment leg), and
  the run **stops** on the class that runs out rather than skipping past the
  lead — leads behind the cut keep their `nextActionAt` and are retried whole.

Its human-cadence constants (5 leads per tick, 40 per run, a 25% tick skip,
shuffled order, a 20–90s pause between leads, inside the send window) still
apply on top; the ledger is the ceiling, not a replacement.

**2. Both rails drive the same real account. Settled 2026-08-25 — do not
re-derive.** The session worker's `x-accounts.json` gives account
`digital_university` an `expectedHandle` of `alfonsojbro`. Firestore holds
exactly ONE `XConnection` document: its id is `tribed` and its `username` is
`alfonsojbro`. Same handle, both rails. The drip now spends inside the session
`caps` (above), but the metering is **one-way**: `remainingToday` is ADVISORY.
The worker charges its counters at reservation inside its own action endpoints,
and there is no reservation endpoint the drip can call — its writes go out on
the OAuth API and never pass through the worker, so nothing the drip spends is
ever visible in `usage.*`. Each rail reads the same snapshot and neither sees
the other's spend. **Until a reserve endpoint exists, the two rails must not
write on the same UTC day** (the day is the worker's `usage.date`, surfaced as
`caps.day`, not the drip process's).

### The arithmetic

`MAX_LEADS_PER_RUN` (40) is the page size, not the spend. The spend is
`MAX_LEADS_PER_TICK` (5) times the ticks that fire: a 13–23 UTC window is 10
hourly ticks and ~25% sit out, so **up to ~37 leads a day** — each one a
follow and (DM step on, copy staged) a DM.

| per day | drip, unmetered | session ramp, week one | ratio |
|---|---|---|---|
| follows | ~37 | 6 | ~6x |
| DMs | ~37 | 3 | ~12x |

Those ramp figures are the real `/health` reading on 2026-08-25 for a session
**one day old**. The drip now reads them per tick and stops on the class that
runs out; the page-size ceiling is just the queue's shape.

### The decision

**The session worker's `/health` ledger is the one X budget for `@alfonsojbro`,
and the drip's WRITE steps stay off until the drip reads it and charges against
it.** Plan every X number off `get_x_session_account_health`, and only that.

Implemented 2026-08-25 (`mcp/src/repos/xDrip.ts`). The remaining condition is
operational, not code: the rails share a snapshot they cannot reserve against,
so they must not both write on the same UTC day.

Not "lower `MAX_LEADS_PER_RUN`": a smaller unmetered number is still unmetered,
and it still runs on a rail with no warmup ramp, no action gap and no proxy. Not
"session rail to reads only" either — the session rail is the one carrying the
safety machinery (warmup ramp, `minActionGapSeconds` 900, `minSendGapSeconds`
3600, attempt-charged counters, per-account sticky egress). Disarming the
careful rail to protect the careless one is backwards. The session rail keeps
its reads AND its metered writes; the drip is the one that has to earn its.

### Live status: ACTIVE as follow + DM since 2026-08-30

There are THREE ids, and they are three namespaces: `X_DRIP_ACCOUNT` is the
lead pool (`Outreach/{id}/leads`), `X_DRIP_CONNECTION` the OAuth token
(`XConnection/{id}`), `X_DRIP_SESSION_ACCOUNT` the session worker whose ledger
meters the rail (`X_WORKERS[{id}]`). The last two default to `X_DRIP_ACCOUNT`.
The session account and the pool happen to hold the same string; that is a
coincidence of naming, not one identifier.

**mcp-ops env, verified on the running container 2026-08-30:**
`X_DRIP_ENABLED=true`, `X_DRIP_DM_ENABLED=true`,
`X_DRIP_ACCOUNT=digital_university`, `X_DRIP_CONNECTION=tribed` (set on the
box, not on Fly — Fly runs no jobs, so its unset value is irrelevant).
`X_DRIP_SESSION_ACCOUNT` is unset and correctly defaults to the pool id,
which matches the worker key. The drip HAS run live: the 2026-08-28 and
2026-08-29 touches (the @youbfit 403s, the "comment skipped" siblings) are
its work, through the pre-pin build the box was still running.

**Connection and worker, verified 2026-08-30:** `XConnection/tribed` is
connected as `@alfonsojbro` with `tweet.write`, `dm.write`, `like.write`,
`stale: false` — and **NO `follows.write`**: the token predates the scope
bump. The follow is the drip's whole public touch now, so `runXDrip`
preflights the scope and THROWS rather than marking leads done with nothing
sent. **The one activation step left is a reconnect via `connect_x`** (Alfonso
clicks the link; the current scope list includes `follows.write`). The worker
gates are all open (`enabled`, `armed`, `sessionStatus: "active"`,
`browserAdapterReady: true`), caps dm 3 / follow 5 on the warmup ramp.

The one still-open operational rule: `remainingToday` is ADVISORY (no
reservation endpoint on the worker), so **the two rails must not write on the
same UTC day** — the drip owns the day; session-worker writes are for
Alfonso's explicit one-target commands only.

## The constraint that shapes DM work: most people cannot be cold-DMed

X only delivers a DM to a stranger when that account has "allow message requests
from everyone" ON. Most do not. A handle alone never tells you, and a DM to a
closed account is not a rejection you can retry — the control simply is not on
the page. The drip degrades gracefully here (a closed-DM recipient 403s and is
logged as a skip, not a failure), but a skipped DM is still a wasted slot.

So the order is always **check, then spend**:

1. `view_x_profile({ accountId, username })` — read-only, costs no cap, never
   clicks. One call returns BOTH `messageAvailable` and the liveness fields
   (`lastPostAt`, `lastPostAgeDays`, `lastPostId`, `lastPostUrl`, `timelineRead`).
2. **Liveness first.** Months-stale `lastPostAgeDays`, or `timelineRead:
   false`, and the candidate is a poor lead — see "Discovery" below. A DM
   probe on a dormant profile is a question about a lead that should not
   exist.
3. `messageAvailable: false` → **do not stage `data.x_dm` and do not queue an
   `x_dm` draft.** The DM cannot land. Stage the lead for the follow touch
   only. Stamp `data.x_dm_available: false` with the date so the next run does
   not re-probe the same profile all week.
4. `messageAvailable: true` → DM-workable. Queue the draft.

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

**Then qualify on LIVENESS, not just existence.** Search has no sense of
whether anyone is home, and neither do `exists` and `messageAvailable`. Google
will happily present a coach whose last post was two years ago, and handles
inferred from result titles are wrong maybe a fifth of the time.
`view_x_profile` is free, read-only and charges no cap — run it on every
candidate before spending anything, and read THREE things off it, not one:

| Field | Fails the candidate when |
|---|---|
| `exists` / `suspended` / `protected` | false / true / true — dead for outreach |
| `lastPostAgeDays` | months-to-years stale — DORMANT, nobody is home to notice a follow or read a DM |
| `messageAvailable` | false — not DM-workable; the follow still is |

**Dormancy is an ICP-quality gate now, not a comment-window one.** With the
comment leg platform-dead there is no post to pin, so the old 7-day hard rule
lost its mechanical reason. What survives is the judgment call: a coach who
has not posted in months is not selling on X, and a follow + DM lands on an
account nobody checks. The 2026-08-27 audit is the cautionary tale — all six
staged leads passed `exists` and `messageAvailable`, and their last posts ran
2021 to 2026 (@TilleyGeorgie 1754 days; @youbfit's 2026-08-18 post is its only
one since 2018). Prefer recently-active accounts; take a dormant one only when
the method evidence is strong and say so on the lead.

**Unknown is not a pass.** `timelineRead: false` — protected, suspended, or the
timeline did not render — means the post fields are UNKNOWN, never "they do not
post". Hold that candidate out of the pool exactly like a dormant one and report
it under its own count (`liveness unknown`), rather than staging on a guess. A
guess here is what parks a dead lead in the queue for a week.

**Re-probing is cheap; re-staging a corpse is not.** A candidate skipped for
dormancy is not permanently dead — people come back. Stamp what you learned
(`data.x_account_dormant`, `data.x_last_post_at`, `data.x_last_post_checked_at`,
below) so a later run can re-check it without re-deriving anything.

**The gate rides the browser rail, so a disarmed adapter means ZERO staged, not
poor ICP fit.** `/profile-view` is behind `_require_adapter`: with
`caps.gates.browserAdapterReady: false` every `view_x_profile` call answers 503,
liveness is unknown for every candidate, and the gate correctly stages nothing.
Report that as the qualify rail being down and name the gate — never as "the X
queries do not find our ICP", and never by staging unqualified handles to make
the number look better. This is the one thing a disarmed adapter DOES stop on
the X leg; the drip itself still runs (see Fill 4).

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

## Comment copy — RETIRED (2026-08-30)

There is no comment copy any more. X 403s programmatic replies on every
buyable tier (top section), so `data.x_comment` and its pin fields
(`x_comment_post_url` / `x_comment_post_id` / `x_comment_post_at`,
`x_comment_needs_fresh` / `_block_reason` / `_blocked_post_id`) are retired:
do not stage them, do not sweep for them, do not restage blocks. Old values
left on leads are inert — the drip ignores them and an old pin block no
longer keeps a lead out of the queue.

The pinned-post design (2026-08-25 to 2026-08-30) lives in git history at
`mcp/src/repos/xDrip.ts` if a working reply path ever returns. The copy-voice
rules it carried (react to ONE thing they actually say, phone register, never
the three-beat critic essay) still apply — to the DM below.

## DM copy

Same rules as the IG cold DM (references/instagram.md): the named app, the
gift-first hook, rule zero in line one, no self-introduction paragraph, max two
short paragraphs. Use the same format library (`aida` / `pas` / `bab` /
`reveal`) and tag the format on the lead.

**Anchor the DM on their METHOD, not on a post** (Alfonso, 2026-08-27). Write
it about what they teach and who they teach it to — the named program, the
population, the promise — because that is what an app would be built around
and it is what they recognise as theirs.

This is a durability property, not a style note. Post-anchored copy rots: the
post ages and a dormant profile has nothing to anchor to at all.
Method-anchored copy does not age, so the DM leg stays workable even on a
dormant lead — in the current pool @susanceklosky, @fitwjenn and @MissCGough
are all long quiet yet still show `messageAvailable: true`. With the comment
leg platform-dead, the DM is the only place copy ships at all, so these rules
carry the whole weight of the touch.

The rest of the DM rules are unchanged, and the two that get broken most are:
still no link in a cold opener, ever, and still no self-introduction paragraph.

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
handles, run `view_x_profile` for liveness AND `messageAvailable` in one call,
prefer candidates the liveness read favors (see below), stage `data.x_dm`
from an approved draft for DM-workable leads, then upsert with `channel "x"`,
`externalId` = the handle lowercased, `data.x_username`, `x_state:
"to_touch"`, `nextActionAt` today. No `data.x_comment` and no pin fields —
the comment leg is dead (top section). Drip pace is ~5 leads a tick and 40 a
run, so a couple dozen due leads keeps it fed without flooding it. Report cap
vs staged vs sent, report dormant-skipped and liveness-unknown as their own
counts, and report a zero with its denominator.

**The liveness keys, on the lead and on a skipped candidate alike.** These are
the keys the 2026-08-27 audit stamped on the six dormant leads already in the
pool, so a run reads and writes the same three:

| Key | What goes in it |
|---|---|
| `data.x_last_post_at` | the newest OWN post's timestamp, ISO, off `lastPostAt` — null when `timelineRead` was false |
| `data.x_last_post_checked_at` | when this probe ran, ISO. A stamp older than a week is not evidence about today |
| `data.x_account_dormant` | `true` when the profile reads long-quiet (months stale); `false` once a re-probe finds a fresh post |

**These keys are a gate again, and the DRIP is what enforces it (2026-08-31).**
The 7-day comment window died with the comment leg, but the keys did not: the
drip reads `data.x_account_dormant` and `data.x_last_post_at` at send time and
PARKS the lead rather than following and DMing it — `data.x_state: "done"`,
`nextActionAt` cleared, an `automated: true` note naming the age it read, and
`data.x_dormant_parked_at` stamped. The tick summary counts it as
`N parked as dormant`. The threshold is the drip's own,
`X_DRIP_DORMANT_MAX_AGE_DAYS`, default **30 days** — not 7. A follow and a DM do
not care how recently the person posted; the only question is whether anybody is
home to read the DM, so a fortnight of quiet is still worth the touch and three
months is not. This exists because a lead that went quiet after it was staged
used to keep its `nextActionAt` forever: @youbfit was the entire due X queue on
2026-08-30 in that shape.

Three limits worth knowing:

- **An absent `data.x_last_post_at` is UNKNOWN, not dormant.** A lead nobody
  probed is still touched. The gate is only as good as the probing, so keep
  probing in Fill 4.
- **The drip can only retire, never revive.** It reads stamps; it does not read
  X. Only `view_x_profile` sees a profile come back, and unparking is the whole
  set or nothing.
- **`data.x_last_post_checked_at` is never written by the drip.** That key means
  "when `view_x_profile` last ran". Faking it would tell the next liveness pass
  the account was looked at today and hide a revived profile.

Prefer active accounts when staging. If a dormant one is staged on strong method
evidence, clear `data.x_account_dormant` in the same write or the drip parks it
on its next tick. Clearing the flag is a fresh `view_x_profile` read, not a
manual edit: it describes what was seen, so only a new look changes it.

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
