# Why calls dropped while outreach went up (2026-10-07)

Verdict: the offer is not the main problem. Three things are, in this order:
1. The follow-up system stopped sending.
2. The post-demo ladder never asks for a call.
3. Warm leads wait on Alfonso for 11 to 21 days.

Sources: get_outreach_metrics, list_outreach_snapshots (67 days), list_outreach_analyses (09-22 to 10-07), the meeting, demo_sent and audit_proposal lead lists, and git history of the skill.

## The numbers

- Meetings: 7 in mid-August, 9 since 09-22. Flat for 6+ weeks.
- 7 of the 9 meeting leads were first contacted in July. Only 2 came from September contacts (Jack Broudy LI, Vero IG).
- Demos sent: 3 in mid-September, 46 now. 26 of the 46 opened the demo. 0 of them became a meeting.
- 35 leads sit in `audit_proposal`. That stage has no call step.
- Reached rose from about 850 to 1,306. Most of the new volume is IG and X.

| Channel | Reached | Reply rate | Meetings |
|---|---|---|---|
| LinkedIn | 652 | 10.3% | 7 |
| Instagram | 442 | 1.8% | 2 |
| X | 211 | 0.9% | 0 |

The volume went up in the two channels that do not book calls.

## Cause 1: the follow-up system stopped sending

From the daily analyses (all open):
- LinkedIn follow-ups due grew from 238 (09-23) to 471 (10-07). Follow-ups proven sent: 0.
- The acceptance sweep has not run for 8 days. 1 acceptance proven in 8 days. The due list is mostly unaccepted invites, so the writer has nothing it can send.
- Post-demo bumps: on IG, `demo_bump` cancels every bump (20 since 09-21). On LinkedIn the ladder has not run since 09-18 and cannot read Sales Navigator threads.
- Demos from July and August got no follow-up until Martin hand-sent "did you get a chance to look at it?" on 10-04. Examples: Hannah (demo 07-21), Christina (08-09), Gabrielle (08-29).
- Rails silently stopped: IG sent 0 of 65 on 09-27; the main IG account queued 0 on 10-07 (connection down); X sent 0 on 10-05.

## Cause 2: the ladder routes interest away from the call

Skill changes, in order:
- 09-07: "commitment gate — earn yesses before offering the call".
- 09-14: the yes is "two slots plus the tour"; the demo does not unlock the price.
- 09-21: three rungs (app, audit proposal, video walkthrough); "asked-for-time means no call ask"; bump 2 offers the audit.

Effect: interested leads get an app, then a question bump ("would you actually start a new singer that way?"), then an audit offer. None of these steps asks for 15 minutes. Meetings stopped growing in the same window.

## Cause 3: warm leads wait on Alfonso

- Gerald Joseph asked for a call on 09-26. Still held behind the reply gate on 10-07.
- Rachel Sklar accepted $149/mo on 09-25. Still blocked on 5 product answers.
- Bostjan Novak and Jack Broudy (calls booked) wait for an agreement and terms in writing.
- 27 items wait on Alfonso, oldest 21 days.
- Bostjan's booked call was invisible on the board on 09-22 (next action six weeks in the past).

## Smaller signals

- Invite quality: one spam complaint (Jared Spool), one "AI emails dont motivate me". Martin's invites go without a note until 11-01 (note cap used).
- IG records 0 seen over 220+ sends. This may be a measurement gap, not zero interest.

## Recommended fixes (in order)

1. Today: clear the warm queue by hand. Gerald, Rachel, Bostjan, Jack, and the 26 leads who opened a demo. One message each with a direct call ask and tribed.io/book.
2. Change the post-demo ladder: bump 1 asks for a 15-minute call. The audit becomes the fallback, not the next step.
3. Fix the acceptance sweep and the IG `demo_bump` cancel bug. Then follow-ups can send again.
4. Move volume back to LinkedIn. Pause IG/X growth until they show replies.
