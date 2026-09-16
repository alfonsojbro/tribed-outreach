# Reply send policy — what a run may send by itself

This file governs **sending** a reply. What to WRITE is still `references/replies.md` (Mode 2),
`references/postdemo.md` (Mode 5) and `references/answers.md` (the five recurring buying questions).
Nothing here changes a word of the copy; it decides whether that copy leaves the machine.

## The policy, and the one line that flips it

Two modes, stored per account as the outreach state key `reply_send_policy`:

- **`manual` — the shipping default, and what is live today.** Every reply is drafted, queued `held`,
  and put in front of Alfonso on WhatsApp. NOTHING is auto-sent. A `hold` under this policy is the
  correct answer, not a failure.
- **`auto_with_holds`** — a confident reply built from the approved answer bank sends itself. Everything
  the gate is unsure about still holds, and the hard stops still block.

Read it at the top of every reply pass. Never assume the mode:

```
get_reply_send_policy { accountId: "digital_university" }
```

Flipping it is Alfonso's decision, made after he has read the five stock answers in
`references/answers.md`. A run never flips it on its own judgement. When he says yes, it is one call:

```
set_reply_send_policy { accountId: "digital_university",
                        patch: { replySendPolicy: "auto_with_holds" } }
```

## Per reply, in this order

1. **Classify.** Sentiment (`positive` | `neutral` | `negative` | `ambiguous`), kind (`stock_answer` |
   `routine_followup` | `question` | `objection` | `other`), which approved answers it is built from
   (`bankKeys`, empty when you wrote it yourself), whether it carries an `[ALFONSO TO CONFIRM]` blank,
   and your confidence.
2. **Ask the gate.** `assess_reply_send { accountId, passId, threadId, leadId?, reply, lead, thread? }`.
   It loads the policy and the thread's send history for you. It answers `send`, `hold` or `blocked`,
   with every reason that fired.

   `lead.isFirstReplyInThread` is REQUIRED and has no default. Compute it from the thread you already
   opened in step 1: it is `true` when `unibox_get_thread` shows exactly ONE message with
   `direction: "in"`, and `false` otherwise. Do not guess it and do not pass `false` to get past the
   schema — it is what stops the first reply from a high-follower account sending itself, and it is the
   one conversation worth waiting an hour for. If you could not read the thread, you cannot answer the
   gate: hold instead.
3. **If `send`: RE-READ THE THREAD FIRST.** The draft was written minutes ago against a thread that may
   have moved. Re-fetch it — `unibox_get_thread`, and on LinkedIn `read_linkedin_session_inbox` with
   `threadIds` (that surface is the authoritative one). If a newer inbound has landed, throw the draft
   away and re-classify from step 1. Only then `unibox_send_message`, then `record_reply_send
   { mode: "auto" }`, then `log_outreach_touch`, then one line in the vault triage note.
4. **If `hold`:** `queue_outreach_drafts` with status `held` and slot `reply`, then ONE `send_brain_update`
   at `needs_input`:

   ```
   HOLD <Name> (<LI|IG|X>) · they said: "<preview>" · draft: <the full draft> · reply "brain: ok <Name>" to send
   ```

   The full draft goes in the line. A hold Alfonso cannot read is a hold he cannot answer.
5. **If `blocked`:** send nothing and ask nothing. It goes in the pass digest with its reason.

## The digest, every pass

Whenever N + M + K > 0, one `send_brain_update` at `completed`:

```
Auto-sent N: <names> · Held M: <names> · Blocked K: <names, each with its reason>
```

## The hard stops (blocked, not held)

- `human_owned` — the lead is a human's: `data.li_state` is `human`, or `data.automated` is `false`.
  A thread somebody took over is never handed back to a machine, not even for approval.
- `cooldown_24h` — this thread was already AUTO-answered inside the window. A reply Alfonso approved by
  name does not arm the cooldown; his decision is not the machine's.
- `pass_cap` — the pass has spent its auto-send allowance.

## The uncertain triggers (hold)

A question or an objection with no approved bank answer · a risky topic (price negotiation, discounts,
contracts, ownership, export, refunds, legal, GDPR) · negative or ambiguous sentiment · **any hardship
signal** · a draft carrying an `[ALFONSO TO CONFIRM]` blank · the first reply from a high-follower
account · low confidence · any kind other than a stock answer or a routine follow-up.

## The bereavement rule

**Any hardship signal holds. Always, under every policy.** A bereavement, an illness, a hospital, a
diagnosis, a surgery, a divorce, burnout, grief — in English or in Spanish. And the held draft must
**acknowledge it first, before anything else**: no pitch, no demo, no ask in the same breath. One human
sentence, then nothing. Alfonso decides whether the conversation continues at all, and when.

## The approval loop

Each pass, before drafting anything new:

1. `list_brain_commands { status: "pending" }`.
2. Match each command against the held drafts, case-insensitively: `ok <name>` or `send <draftId>` means
   send; `skip <name>` means drop it.
3. For a send: `approve_outreach_draft` → re-read the thread (step 3 above, the same rule applies) →
   `unibox_send_message` → `record_reply_send { mode: "approved" }` → `log_outreach_touch` →
   `complete_brain_command` with a one-line confirmation of what went out.
4. For a skip: `reject_outreach_draft` → `complete_brain_command`.

## The documented gap

Approving from WhatsApp works, but it is coarser than it should be, and nothing new was built for it:

- Outside the linked ops room, a command must carry the `brain:` prefix (`brain: ok Kat`). Bare
  `approve` / `skip` / `status` only work inside the ops room itself.
- There are no per-draft buttons and no per-draft ids in the WhatsApp line. Matching is by NAME, so two
  held drafts for two people with the same first name are ambiguous — when that happens, hold both and
  say so rather than guessing.
- A command that matches nothing is left pending, never guessed at.

## Connections

- `references/replies.md` — Mode 2, what the reply actually says.
- `references/answers.md` — the five approved answers a `stock_answer` is built from.
- `references/pipeline.md` — the daily run this pass sits inside.
