# Audit — post-demo clock, CODE half (plan items 1-6)

Branch `feat/li-drip-demo-ladder`, worktree
`/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder`,
based on origin/main 519a4ce. Nothing committed. No skill docs touched.

## Files changed

- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/mcp/src/repos/linkedinDrip.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/mcp/src/repos/linkedinDrip.test.ts`

## linkedinDrip.ts

| Where | What |
|---|---|
| :32-35 (header) | `li_state` list gains `"demo_await_1" \| "demo_await_2"` as drip-owned, pointing at the ladder branch. |
| :82-95 | `DEMO_FU1_DELAY_DAYS = 3`, `DEMO_FU2_DELAY_DAYS = 4`, `DEMO_LADDER_REST_DAYS = 14`, beside `FOLLOWUP_2_DELAY_DAYS`, each commented with the measured problem and the followups.md table that cites them. |
| :810-855 | New ladder predicates beside the demo rail's: `DEMO_LADDER_STATES`, `demoLadderCopy` (li_demo_fu1 / li_demo_fu2 by state), `onDemoLadder` (ladder state AND no `slots_offered_at`), `demoLadderSendable` (copy present, no placeholder, `copyIsFresh(li_demo_copy_at)`). Split exactly like `demoDeliveryStaged` / `demoCopySendable`, so a stale-copy lead is still selected and gets its park written. |
| :889, :897-903 | `isSendReady` is now exported, and returns true for a sendable ladder lead — placed immediately after the `demoCopySendable` check and BEFORE `hasLoggedReply`, because every ladder lead holds a reply by construction. |
| :970-989 | `DRIP_OWNED_STATES` gains the two states (five now). `automated: false` still parks them. |
| :1449-1462 | New `SkipReason` members: `no demo bump copy`, `demo bump copy carries a placeholder`, `no fresh demo bump copy`, `demo bump skipped: they replied`, `no thread to bump the demo in`, `slot offer owns this thread`. |
| :1558-1566, :3679-3680 | `Counters.demoFu1` / `demoFu2` and their zero-init. |
| :2071-2076, :2098-2102 | `SENT_DEMO_FU1` / `SENT_DEMO_FU2` markers, registered in `SEND_MARKERS` under the existing `"demo"` kind (no new `DripSendKind` members) so a day whose only send was a bump is not reported as a stalled rail. |
| :2314-2320 | Message-budget defer for a ladder lead that actually has copy to send; a copy-less lead still gets its park (parks cost no message). |
| :2393-2426 | Demo rail success path: `laddered` = non-empty `data.li_demo_fu1`. When true the write is `li_state: "demo_await_1"`, touch `nextAction: "Demo bump 1 (post-demo ladder)"`, `nextActionAt: plusDays(DEMO_FU1_DELAY_DAYS)`. When false it is byte-for-byte today's behaviour (`REPLY_DUE_STATE`, `REPLY_DUE_ACTION`, +1). `li_demo_sent_at`, `demo_delivered`, the `SENT_DEMO` note and `c.demoDelivered` are untouched. |
| :2426-2545 | The new leg, right after the demo rail and before `to_invite`. In order: `slots_offered_at` → `skip("slot offer owns this thread")` and return with no lead write at all; missing copy → park +0 naming `data.li_demo_fu1`/`li_demo_fu2`; placeholder → park +0; `!copyIsFresh(li_demo_copy_at)` → park +0 (never reads `li_copy_at`); `checkDemoThreadTail` — `"theirs"` → `REPLY_DUE_STATE` + `advanceTo: "replied"` + `REPLY_DUE_ACTION` +1, `"no_thread"` → `li_state: "human"`, `automated: false`, note says a ladder lead with no thread is a delivery bug, `"unreadable"` → defer +1, `"ours"` → send. Send is `worker.sendMessage(username, copy, { profileUrn: data.li_urn, accountId: session })`, then `budget.messages--`; bump 1 stamps `li_demo_fu1_sent_at`, moves to `demo_await_2` at +`DEMO_FU2_DELAY_DAYS`, `c.demoFu1++`; bump 2 stamps `li_demo_fu2_sent_at`, `li_state: "done"`, `nextAction: "Park (demo ladder done)"`, +`DEMO_LADDER_REST_DAYS`, `c.demoFu2++`. No profile/degree re-read: a demo recipient is already in conversation. No new budgets. |
| :3429-3434 | Due-query comment only. The two states are in `DRIP_OWNED_STATES`, so `!isHumanOwned(l)` already admits them; verified, no duplicated logic. |
| :3816-3817 | Tick summary gains `N demo bump 1 sent (post-demo ladder)` / `N demo bump 2 sent (post-demo ladder, now parked)` right after `demoDelivered`. |
| :4055-4067 | `readDripStatus`: a due `onDemoLadder` lead counts in `dueActionable`, and in `copyFreshToday` when `demoLadderSendable` (so `needsCopy` picks up stale bump copy). Placed after the demo-delivery branch and before the `hasLoggedReply` test, for the same construction reason. A `slots_offered_at` lead is not on the ladder and falls through to `dueWithReply`, which is correct: the 48-hour bump is a person's work. |

## linkedinDrip.test.ts

Appended one section, `── The post-demo ladder (2026-09-14) ──` (:5304-5535), using the file's
existing harness: `setup`, `runDrip`, `readStatus`, `conversationPayload`, the `demoLead`
fixture, `theyRepliedLast` / `weSpokeLast`, `todayIso` / `daysAgoIso`. New local helpers:
`ymdPlus(n)` (mirrors the source's `plusDays` so a "+3" assertion names a date),
`BUMP_1_COPY` / `BUMP_2_COPY`, the `ladderLead` fixture, `sendsRecorded`. Import at :372
extended with `isSendReady`.

Nine tests, one per plan item 5(a)-(h) plus the reply-yield:

1. delivery with `li_demo_fu1` → `demo_await_1` at +3, delivery bookkeeping unchanged
2. delivery without it → `reply_due` at +1 (today's behaviour preserved)
3. stale `li_demo_copy_at` → parks +0, sends nothing, stays on the ladder
4. tail `"theirs"` → `reply_due`, stage `replied`, no send, no stamp
5. tail `"ours"` → sends `li_demo_fu1` with `accountId`, stamps `li_demo_fu1_sent_at`, `demo_await_2` at +4, writes the `Sent demo bump 1 to` note
6. bump 2 → sends `li_demo_fu2`, `done` at +14, `Park (demo ladder done)`, stage untouched, summary line
7. `slots_offered_at` → no writes at all and no worker call (the forbidden mocks prove the thread was never even read)
8. `readDripStatus` → stale ladder lead under `needsCopy`, fresh one under `copyFreshToday`, neither under `dueWithReply`
9. `isSendReady` → true fresh, false with `slots_offered_at`, false with stale copy

## Deviations from the plan

1. **`isSendReady` is now exported** (one `export` keyword, :889). Plan item 5(h) asks for a
   direct test of it and it was module-private. It joins `isHumanOwned` and
   `isDemoDeliveryDue`, which are exported for exactly the same "the mirrors must agree"
   reason. Purely additive.
2. **The send does not inspect the payload status.** The task said "mirror how the FU2 branch
   reads the payload"; the FU2 branch (and the demo rail) do not read it at all — the worker
   turns every non-`sent` status into a 422 `WorkerError`, which `processLead`'s catch already
   handles. So the leg mirrors FU2 exactly, and "quotes the status" is honoured the way every
   other rail honours it: the touch note is built from shared `SENT_*` marker constants that
   `classifySendNote` reads back. Adding a bespoke 2xx status parser here would have been the
   only such parser on any message rail.
3. **Two extra `SEND_MARKERS` rows** (the new constants under the existing `"demo"` kind), not
   named in the plan. Without them `readDripStatus.sentToday` misses a tick whose only send was
   a bump and reports the rail as stalled. No type change — no new `DripSendKind`.
4. **New `SkipReason` members.** Implied by the plan's skip/park behaviour; the union is
   exhaustive so the reasons could not be reused loosely.
5. **The warm-up comment leg is untouched** for ladder leads (only the demo rail exempts
   itself). In practice a post-demo lead carries no staged comment copy, and suppressing it
   would be an unplanned behaviour change.

## Tests

```
$ cd mcp && node --import tsx --experimental-test-module-mocks --test src/repos/linkedinDrip.test.ts
# tests 189
# pass 189
# fail 0
```
Baseline was 180/180; the nine new tests are the difference, and no existing test changed.

```
$ cd mcp && npx tsc --noEmit -p .
(no output, exit 0)
```

`git status --short` shows exactly the two files above, modified, uncommitted.

## Open risks

- **Nothing puts existing leads on the ladder yet.** The rail only fires for demos delivered
  from today on (`laddered`). The 62 `demo_sent` leads need the plan's item-10 backfill; until
  that runs the code changes nothing for them.
- **`demo_await_*` with `automated: false`** is parked, like every drip-owned state. A backfill
  that sets `li_state` without also logging a touch with `automated: true` leaves the lead
  silent — the file's standing warning about un-parking, restated in the `DRIP_OWNED_STATES`
  comment.
- **`slots_offered_at` is read but never written by this file.** If Mode 5 ever stamps a
  different key, a lead can carry both clocks. The field name comes from the plan; nothing in
  the repo writes it today.
- **Bump copy depends on `li_demo_copy_at`,** which the delivery rail also uses. A lead that
  gets a fresh delivery stamp while sitting on the ladder would read as fresh bump copy. Not
  reachable today (`li_demo_sent_at` is terminal for the delivery rail), but the two rails now
  share one stamp.

## Review fix pass

Same worktree `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder`,
branch `feat/li-drip-demo-ladder`. Nothing committed. No `mcp/skills` file touched.

### Files changed (this pass)

- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/mcp/src/repos/linkedinDrip.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/mcp/src/repos/linkedinDrip.test.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/src/components/screens/outreach/components/linkedinSequence.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/src/components/screens/outreach/components/linkedinSequence.test.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/src/components/screens/outreach/components/LinkedInLeadSequences.tsx`

### 1. Dashboard mirror (was blocking) — DONE

| File:line | What |
|---|---|
| linkedinSequence.ts:17-22, :44-49 | Header contract documents the two ladder states and the `li_demo_fu1` / `li_demo_fu2` / `li_demo_copy_at` fields. |
| linkedinSequence.ts:66-79 | `DRIP_OWNED_STATES` gains `demo_await_1` / `demo_await_2`; `DEMO_FU1_DELAY_DAYS` / `DEMO_FU2_DELAY_DAYS` written out beside the other mirrored constants. |
| linkedinSequence.ts:108-112 | `LinkedInSequenceState` union gains both. |
| linkedinSequence.ts:271-280 | `storedState` whitelist gains both, so they no longer fall to `"unknown"`. |
| linkedinSequence.ts:449-511 | New projection branch, placed after the shared username gate so it inherits it. Copy gate mirrors `demoLadderSendable` in its branch order (missing copy → placeholder → same-day stamp) and reads `li_demo_copy_at`, never `li_copy_at`. `today` always reads `Demo bump N due <date>`; the slot-offer case says the 48-hour bump owns the thread and `actsToday` is false. |
| linkedinSequence.ts:674-683 | `buildLinkedInDayPlan.messagesToday` counts an acting ladder lead: a bump is a DM on the same budget, and it carries no `todayStep`. |
| LinkedInLeadSequences.tsx:120-123 | `STATE_LABEL`: "Demo sent — bump 1" / "Demo sent — bump 2". |

Deliberate limits, both noted rather than fixed:
- The bumps are NOT modelled as extra `LinkedInStep`s. The three step types are the templated
  rail's; a fourth and fifth type would be a second sequence model in the file whose header
  refuses exactly that. The bump is reported in `today` / `needsHuman`, and the step row shows
  the demo delivery date.
- `state === "handed_back"` still maps a ladder lead's parked step to `followup_2` (the existing
  ternary's else). Wrong label, right behaviour (parked, `actsToday` false); changing it needs
  the step model above.

Tests (linkedinSequence.test.ts:180-249, five cases): the label + due date instead of "Unknown
drip state"; copy-stale vs fresh read off `li_demo_copy_at` while `li_copy_at` is nine days old;
bump 2's missing copy; the slot-offer yield; a handed-back ladder lead.

```
$ npx react-scripts test --watchAll=false src/components/screens/outreach/components/linkedinSequence.test.ts
Test Suites: 1 passed, 1 total
Tests:       20 passed, 20 total
```

### 2. Duplicate-send guard — DONE

`linkedinDrip.ts:844-852` new `demoLadderSentStamp` (the stamp the CURRENT state writes);
`:864` `demoLadderSendable` refuses on it, so a spent bump cannot even reserve a tier-A slot;
`:2537-2550` the branch parks with a note naming the stamp's date, `nextActionAt: plusDays(1)`,
`li_state` untouched. Plain truthiness with the `acceptedFu1Sendable` inversion comment.
New `SkipReason` `"demo bump already sent"` (:1546-1548).
Tests: "a bump whose stamp is already set is refused, not sent again", "bump 2 is refused once
li_demo_fu2_sent_at is set" (the latter also asserts `isSendReady` false).

### 3. Reply guard failing open on the second mailbox — DONE (cheap fix)

`linkedinDrip.ts:891-911` new `leadReplyAfter(lead, since)`: the first `conversation` entry
`from: "lead"` dated after the message this bump is bumping (`li_demo_sent_at` for bump 1,
`li_demo_fu1_sent_at` for bump 2), returning its timestamp so the note can name the date.
Undated messages are ignored (the browser read is then the authority); an unparseable `since`
counts nothing. `:2596-2600` the branch asks the record BEFORE `checkDemoThreadTail` and treats a
hit as `"theirs"` → same `reply_due` + `advanceTo: "replied"` park, with its own note.
Tests: "a reply logged on the lead after the demo stops the ladder without a thread read" (the
forbidden `getConversation` mock proves no browser read), and "the reply that ASKED for the demo
does not stop the ladder" — the false positive that would make the rail unreachable.

### 4. Slot-offer yield moved above `postStagedComment` — DONE

`linkedinDrip.ts:2354-2368`, immediately before the `postStagedComment` call. The in-branch copy
is gone. The comment now says what the code does: a staged warm-up comment is a touch too, so the
yield has to come before it; nothing is read, nothing is written. The existing test "a lead the
Mode 5 slot offer owns is left completely alone" still passes unchanged (it asserts
`writes` empty and `workerCalls` empty).

### 5. Park notes deduped per day — DONE

`linkedinDrip.ts:913-923` new `hasTouchNoteToday(lead, note)` (reads `lead.touches`, no extra
Firestore read); `:2517-2532` a local `parkOnce(note, nextAction, reason, days = 0)` used by all
three `plusDays(0)` parks and by the new duplicate guard. The schedule is unchanged: the touch
already on file today carries the same `plusDays(0)`, so the lead stays due today.
Test: "a park note is written once a day, however many ticks run" — two `runDrip()` calls, one
note, `lead.touches.length === 1`, `nextActionAt` still today.

### 6. `readDripStatus` stall text — DONE

`linkedinDrip.ts:4161-4167` new `dueOnDemoStamp` counter, incremented in both the demo-delivery
branch (:4190) and the ladder branch (:4203); `:4278-4295` the "every one is parked on the
same-day copy gate" reason now names `data.li_demo_copy_at` when the whole actionable set is
demo-rail/ladder, `data.li_copy_at` when none of it is, and both (with the split) when it is
mixed.

### 7. Stale comment — DONE

`linkedinDrip.ts:3451-3454`: "each of those five returns above (to_invite, awaiting_accept,
await_fu2, and the two post-demo ladder states)".

### 8. `demoDeliveredOffLadder` — DONE

`linkedinDrip.ts:1615-1620` counter with its reason; `:2478` incremented when the delivery lands
at `reply_due` for want of `li_demo_fu1`; `:3806` zero-init; `:3944-3945` printed on the tick
summary immediately after `demoDelivered` ("N of them NOT on the post-demo ladder (no
li_demo_fu1 authored, so nothing bumps them)").

### 9. Missing tests — DONE

All in `linkedinDrip.test.ts`, section "The ladder's refusals (2026-09-14, the review of this
rail)": tail `"no_thread"` → `li_state: "human"` + `automated: false` + "delivery bug" note;
tail `"unreadable"` → defer +1, state unchanged, nothing stamped; an `automated: false` ladder
lead → no send, no read, no touch at all; `budget.messages <= 0` → four due ladder leads, three
send, the fourth is left untouched and unsent.
`FakeLead.conversation` gained an optional `at` (mirrors outreach.ts `LeadMessage`).

### Commands and results

```
$ cd mcp && node --import tsx --experimental-test-module-mocks --test src/repos/linkedinDrip.test.ts
# tests 198
# pass 198
# fail 0
```
(189 before this pass; nine new tests, no existing test changed.)

```
$ cd mcp && npx tsc --noEmit -p .
(no output, exit 0)

$ npx tsc --noEmit -p .          # repo root, the dashboard's only tsconfig
(no output, exit 0)

$ npx react-scripts test --watchAll=false src/components/screens/outreach/components/linkedinSequence.test.ts
Tests: 20 passed, 20 total
```

`git status --short` shows exactly the five files above, modified, uncommitted.

### Not done / still open

- Nothing in this pass changes the backfill risk from the first audit: the 62 delivered demos
  still reach the ladder only through plan item 10.
- The dashboard's handed-back step label for a ladder lead (see item 1) and the absent
  `todayStep` for a bump are the two places the three-step model shows its age.

## Nits pass

Same worktree `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder`,
branch `feat/li-drip-demo-ladder`. Nothing committed. No `mcp/skills` file touched by this pass
(the worktree's `git status` now also lists seven `mcp/skills/tribed-outreach/*` files and one
untracked `references/answers.md` — those are another task's, untouched here).

### Files changed (this pass)

- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/mcp/src/repos/linkedinDrip.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/mcp/src/repos/linkedinDrip.test.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/src/components/screens/outreach/components/linkedinSequence.ts`
- `/Users/alfonsobriceno/Documents/Projects/Tribed/NobleAdmin/.claude/worktrees/demo-ladder/src/components/screens/outreach/components/linkedinSequence.test.ts`

### 1. Dashboard mirror of the duplicate-send guard — DONE

`linkedinSequence.ts:465-479` reads `li_demo_fu1_sent_at` on `demo_await_1` and
`li_demo_fu2_sent_at` on `demo_await_2` (the stamp the CURRENT rung writes, so a `demo_await_2`
lead carrying bump 1's stamp is the normal case), ahead of the copy gate, as an `else if` chain:
a spent rung with fresh copy must not read as a send the drip will make. The blocker is
`demo bump N already went out on <date>; the drip refuses to send it twice and li_state needs
moving by hand`, date via the file's own `dayOf`. `actsToday` is already `due && !blocked && ...`,
so it goes false, and `buildLinkedInDayPlan.messagesToday` (`:690`) counts ladder leads only when
`actsToday`, so the lead drops out of the day's message count with no change there.

`linkedinSequence.ts:493-497, :515-521`: a local `action = lead.nextAction || \`Demo bump ${bump}\``
now heads all four `today` lines, the way the `reply_due` branch defers to `lead.nextAction`.

Test `linkedinSequence.test.ts:256` — fresh copy plus `li_demo_fu1_sent_at` two days old: no act,
the blocker names the date and the hand-move, and `today` says parked.

### 2. Budget check no longer pre-empts the guard — DONE

`linkedinDrip.ts:2393-2408`: the defer now reads
`onDemoLadder(data) && demoLadderCopy(data) && !demoLadderSentStamp(data) && budget.messages <= 0`.
Without it a spent-stamp lead was filed "message cap reached" every tick — a send waiting on room
it would never use — and the park below never ran.
Test `linkedinDrip.test.ts:5754` "a spent bump is handed over even on a tick with no budget left":
three shippable bumps take the whole budget, the fourth (spent stamp) still reaches `human` with
the "already went out on" note.

### 3. The duplicate-send park hands the lead over — DONE

`linkedinDrip.ts:2549-2566` now matches the `no_thread` abort: `setLeadData li_state: "human"`,
`logTouch` with `automated: false`, a note naming the stamp's date and saying a person has to
decide where the lead goes next, `nextActionAt: plusDays(1)`. It no longer goes through
`parkOnce` — once the lead is human-owned it never re-enters the due set, so the per-day note
dedupe has nothing left to protect.

`readDripStatus` needed NO change, and that was verified rather than assumed: its `due` test is
`... && (!isHumanOwned(lead) || isDemoDeliveryDue(lead))`, and `demoDeliveryStaged` (`:789-799`)
returns false on `automated === false`, so the lead falls out of `dueActionable` — and therefore
of `needsCopy` — entirely.

Tests updated: `linkedinDrip.test.ts:5544` now asserts `li_state === "human"`, `automated === false`,
the new note text, and `readStatus()` reporting `dueActionable 0` / `needsCopy 0`; `:5578`
(bump 2) asserts `human` instead of `demo_await_2`.

### 4. Unread `DEMO_FU1_DELAY_DAYS` / `DEMO_FU2_DELAY_DAYS` — DELETED

`linkedinSequence.ts:75-80`. `grep -rn` over `src/` found no reader. Deleted rather than used:
this file never computes a ladder date, it prints `lead.nextActionAt`, which the drip already
wrote from its own constants. A comment in their place says so, so the next mirror pass does not
re-add them.

### Commands and results

```
$ cd mcp && node --import tsx --experimental-test-module-mocks --test src/repos/linkedinDrip.test.ts
1..199
# tests 199
# pass 199
# fail 0

$ cd mcp && npx tsc --noEmit -p .
(no output, exit 0)

$ npx tsc --noEmit -p .          # repo root
(no output, exit 0)

$ CI=true npx react-scripts test --watchAll=false src/components/screens/outreach/components/linkedinSequence.test.ts
Test Suites: 1 passed, 1 total
Tests:       21 passed, 21 total
```
(198 → 199 in the drip suite, 20 → 21 in the dashboard suite; no existing test's subject changed,
only the two duplicate-send assertions that the new park behaviour makes wrong.)

### Open risks

- The duplicate-send park is now a one-way door: a lead it fires on leaves the drip for good until
  a person moves `li_state`. That is the intended fix, but it means a backfill that sets a ladder
  state on a lead that already has the matching `*_sent_at` stamp will quietly hand a batch of
  leads to the human queue on the next tick.
- `LinkedInLeadSequences.tsx` was NOT touched this pass (not in the files-touched list). Its
  `STATE_LABEL` still reads "Demo sent — bump 1" for a lead this card now reports as spent; the
  label is the state's, not the blocker's, so it is accurate, just not loud.
