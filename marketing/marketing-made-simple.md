# Tribed's marketing, run through Marketing Made Simple

Source: *Marketing Made Simple*, Donald Miller and Dr. J.J. Peterson (StoryBrand). This file applies its
five-part sales funnel to Tribed's own marketing, with copy that is ready to lift.

The book's opening argument is the one that matters most for us: **branding is not marketing.** Branding is
fonts, colors, and feel. Marketing is the right words in the right order, ending in a specific offer. Every
gap below is a words gap, not a design gap.

The funnel has five parts, executed in this order:

1. A **one-liner** you can say out loud
2. A **wireframed website** built on the grunt test
3. A **lead generator** that trades value for an email address
4. A **nurture email sequence**
5. A **sales email sequence**

Underneath all five sits the **BrandScript**: the story the customer is the hero of, and we are the guide.

---

## 1. Where Tribed stands today (evidence, not opinion)

Checked 2026-08-24 against the live tenants and the outreach tracker.

| Funnel part | Status | Evidence |
|---|---|---|
| BrandScript | Partial | `get_marketing_context(tribed)` has positioning, pains, ICP, and objections. It has no character/want/failure/success arc, so nothing enforces one story across channels. |
| One-liner | **Missing** | Positioning is a 3-line paragraph that opens with what we build. There is no sayable problem/solution/result line anywhere in the skill, the tenant, or the plugin. |
| Website | Unverified + empty in-tenant | `list_site_pages(tribed)` returns zero pages, `siteEnabled: false`. `list_landing_variants(tribed)` shows the control page at 0 bytes and disabled. The public tribed.io marketing site is hosted outside this admin and was not reachable from this session, so section 3 is written as a spec to check the live page against, not as an audit of it. |
| Lead generator | **Missing** | `list_lead_magnets` returns `[]` for both `tribed` and `contentpreneur-community`. Meanwhile the LinkedIn content strategy set on 2026-07-19 says *every post ends with a lead-magnet CTA: comment a keyword to get a free resource by DM.* We are promising an asset that does not exist. |
| Nurture emails | **Missing** | `get_email_sequence(welcome)` returns zero steps in both tenants. |
| Sales emails | **Missing** | Same. The outreach tracker's email channel shows 4 leads at the top and **0 reached**. Mode 6 of the skill is fully written and has never run. |

Outreach funnel, synced 2026-08-24 (account `digital_university`):

- Combined: 796 reached, 48 replied (6%), 6 meetings, 3 won
- LinkedIn: 360 reached, 41 replied (**11.4%**), 3 won
- Instagram: 436 reached, 7 replied (**1.6%**), 0 won
- Email: 0 reached

Read that next to the funnel table and the diagnosis is not subtle. **The opener is the entire machine.**
There is no website that closes, no asset that captures an email, and no sequence that follows anyone who
did not reply to a DM. 748 people we contacted are simply gone. The book's whole point is that a funnel is
what stops that leak, and we have four of its five parts missing.

The good news: the gift-first move already is StoryBrand. We give the coach a free branded demo, we position
ourselves as the guide, and the customer stays the hero. We do not need a new story. We need to say the
story we already tell in DMs everywhere else.

---

## 2. The BrandScript

Fill this once and never wander from it. Every headline, DM, post, and email below comes out of it.

**A character.** A coach, expert, or author with a proven method and an audience.

**Has a problem.**
- External: their method is scattered across a course platform, a community tool, a scheduler, a video host,
  and their DMs. Five bills, five logins.
- Internal: they feel like a tenant on someone else's platform, and quietly embarrassed that most clients
  never finish the program they are proud of.
- Philosophical: a body of work that took ten years to build should not live on rented land, and it should
  not be delivered as a PDF nobody opens twice.

**And meets a guide.**
- Empathy: "You did not get into coaching to manage five subscriptions and a Zapier account."
- Authority: apps already live in the App Store and Google Play under other people's names, built by us.
  Name them and link them. (Confirm and list the real count and titles before publishing.)

**Who gives them a plan.**
1. Send us your handle or your program.
2. See your app before you decide. We build the demo for free.
3. We ship it to the App Store and Google Play under your name. One bill, we run the technical side.

**And calls them to action.**
- Direct: *Get my free app demo.*
- Transitional: *See what goes inside a coach's app* (the lead generator, section 4).

**That helps them avoid failure.** Another year of renting. Five bills. Clients who quit in week two. Their
best material trapped in a platform they do not own. Someone else in their niche launching first.

**And ends in success.** Their name on a client's home screen. People opening their program daily instead of
buying it and forgetting it. An AI that answers in their voice at 11pm. One company, one bill, and the
customer relationship is theirs.

### The explanatory paragraph (book's template, filled)

> At Tribed we know you are the kind of coach who wants your work to actually change someone's life, not just
> sell once. To do that, your method has to be somewhere your people open every day. The problem is your
> program is spread across five tools none of which you own, which makes you feel like a tenant in your own
> business. We think a method that took you a decade to build deserves better than a folder of PDFs. We know
> what it is like to stitch that stack together. That is why we build coaches their own app, with an AI coach
> trained on your material and speaking in your voice. Here is how it works: send us your handle, see your app
> for free before you decide, and we ship it to the App Store under your name. Get your free demo, so you can
> stop renting your business from five platforms and start owning the relationship with the people you coach.

---

## 3. The one-liner

Problem, solution, result. Short enough to say at a dinner table. This is the single highest-leverage missing
asset, because it goes everywhere: the site header, the LinkedIn bio, the first line of a reply when a
prospect asks "what is this?", the answer to "so what do you do?".

**Primary:**
> Most coaches run their business on five platforms they do not own, and their clients quit by week two.
> We turn your method into your own app, with an AI coach trained on your voice. Your people open it daily,
> and you own the relationship instead of renting it.

**Short, for a bio or a header sub:**
> Coaches rent their business from five different tools. We build you your own app with your method inside it.
> Your clients actually finish, and the relationship is yours.

**Spoken, for a DM reply or a call:**
> I build coaches their own app. Your program inside it, an AI trained on your voice, in the App Store under
> your name.

Rules for using it: say it the same way every time, never open with what we build (that is the current
positioning paragraph's mistake), and never lead with AI. Lead with their problem.

---

## 4. The website, section by section

The order below is the book's, and it is not decorative. Check the live tribed.io page against each one.

### Header (must pass the grunt test)

A stranger has five seconds to answer three questions: what do you offer, how will it make my life better,
what do I do to buy it.

- **Headline:** Your program. Your name. Your app.
- **Sub:** We turn your method into a branded app with an AI coach trained on your voice, live in the App
  Store and Google Play. You bring the message, we handle everything technical.
- **Primary button:** Get my free app demo
- **Secondary button:** See what goes inside

The book is blunt about two things here that are cheap to fix: **put the call to action in the top right AND
in the middle of the header**, so the same button appears twice above the fold, and **kill the rotating hero
slideshow** if there is one, because customers do not wait for a carousel to come back around. Images should
be people using the product and looking better for it, so use a real phone in a real hand showing a real
coach's app, not an abstract gradient.

### Stakes (what it costs to do nothing)

Heading: **What renting costs you**

- Five subscriptions and five logins to deliver one program
- Clients who buy in week one and disappear in week three
- Your best frameworks living inside a platform that can change its rules tomorrow
- An audience you reach only when an algorithm decides to let you
- Somebody else in your niche shipping their app first

### Value stack (the success side, made visual)

Heading: **What changes when the app is yours**

- Your name sitting on your client's home screen, next to their bank and their messages
- An AI coach that answers a 11pm question the way you would answer it
- Your program as a day-by-day track people tick off, not a PDF they open once
- One bill and one company instead of five tools and a Zapier account
- Every member's email, payment, and progress owned by you

Keep the language specific and sensory. The book's example contrast is "you will be satisfied" versus "your
lawn will make your neighbors jealous". Ours is not "a central hub for your community", it is "your name on
their home screen".

### Guide (empathy, then authority)

> You did not start coaching so you could manage subscriptions. We did the stitching so you do not have to.

Authority proof, in this order: apps already shipped and live in the stores, named coaches using them, and
the count of communities running on Tribed today. Logos are weak proof here. Screenshots of real apps with
real names on them are strong proof.

### The plan (three steps, always three)

1. **Send your handle.** No forms, no discovery call.
2. **See your app first.** We build a working demo from your own content, free, before you decide anything.
3. **We ship it.** App Store and Google Play, under your name, one monthly bill.

### Explanatory paragraph

Use the filled paragraph in section 2, placed further down the page for the reader who wants detail.

### Video

A 90-second walkthrough of one real coach's app. We already offer this in outreach follow-up 2, so it
partially exists and just needs a public cut.

### Testimonials, and the asset we are missing

Follow-up 1, the best-performing follow-up we have, currently says "a similar coach" with no name, because we
have no named stories on record. That is the weakest link in an otherwise strong sequence. Collect three, each
in the book's structure: what life was like before, what changed, what the result was. Named, with a photo and
their app. This one asset upgrades the website, the follow-ups, the nurture emails, and the sales emails at
the same time.

### Pricing, in three options

The book is specific: offer three price points, spell out what each includes, and expect most buyers to take
the middle one. Today pricing is $149 or $199 depending on account size and is never shown publicly, only
quoted in a DM. That is a fine rule inside a DM, where quoting a range invites haggling. On a website it costs
us qualified buyers who bounce because they cannot tell if this is $50 or $5,000.

Recommended shape (fill the real numbers before publishing):

| Launch | Pro (most popular) | Studio |
|---|---|---|
| Your app, your program, AI coach, community, tracking | Everything in Launch plus multiple programs, deeper AI training, priority builds | Everything in Pro plus custom features and hands-on launch support |

Note the tension deliberately: publish the tiers on the site, keep quoting **one** price in DMs, and let the
site do the qualifying.

### Junk drawer footer

Everything that does not belong above: about, contact, terms, careers, the blog.

---

## 5. The lead generator (the biggest gap)

The book's line is exact: *give them a reason to give you their email address or they will not.* Right now
nothing on our side asks for an email at all, and the LinkedIn strategy already promises a free resource that
does not exist. Two things to build, in this order.

### 5a. Ship the PDF this week, because we already promised it

Working title: **What actually goes inside a coach's app: the 9-part build sheet.**

Built on the book's four-section lead-generator template:

**Section 1**
- *Problem:* You have a method that works in person, and every attempt to package it online has turned into a
  course people buy and never finish.
- *Empathy and trust:* We have built this exact thing for coaches in fitness, mindset, and business, and the
  pattern of what makes people open an app daily is now boringly predictable.

**Section 2**
- *Agitate:* The part that stings is not the tech. It is watching someone pay you, disappear, and quietly
  decide your method did not work for them, when what actually failed was the delivery.
- *Solution:* There are nine pieces that decide whether a coaching app gets opened daily or deleted in week
  two. Here they are.

**Section 3, the body.** The nine pieces, one short paragraph each: the daily open reason, the program as
tracked days, the AI trained on your material, the onboarding questionnaire, habits, the community space,
the paywall and what sits outside it, notifications that are not spam, and the launch to your existing list.

**Section 4**
- *Stakes:* Miss these and you have built a course with an app icon, which is the most expensive way to learn
  this lesson. Get them right and your program becomes something people open on a Tuesday morning without
  being told to.
- *Call to action:* Want to see this applied to your own method? We will build the demo free.

Where it goes: a section on the site, an exit-intent pop-up (the book recommends the pop-up explicitly, with a
ten-second delay or an exit trigger), the LinkedIn keyword CTA that already promises it, and the Instagram
bio link.

### 5b. Then make the demo itself self-serve, because that is our real magnet

Our gift-first demo is a better lead generator than any PDF, and it is the reason LinkedIn replies at 11.4%.
The problem is it only exists when a human builds one, which caps it at outreach volume.

**Recommendation:** a public "see your app" page. A coach enters their handle, we generate the preview with
the same pipeline the demo skill already runs, and the email address unlocks it. That converts every visitor,
every podcast mention, and every one of the 748 people who never replied to a DM into a capture, using the
asset we already know converts.

This is the single highest-return item in this document. Everything else on this page is words. This one turns
the thing that already works into something that scales past manual outreach.

---

## 6. Nurture emails

Weekly, valuable whether or not they ever buy, mostly teaching, with a soft PS. Sent to everyone who
downloads the build sheet or generates a preview. Subjects stay short, lowercase, and boring, matching our
cold-email rules.

| # | Subject | Angle |
|---|---|---|
| 1 | your build sheet | Deliver the asset, one line on what to read first, nothing else asked |
| 2 | week two | Why clients quit in week two and what changes it. The delivery problem, not the content problem |
| 3 | the five bills | The stack teardown, with real tool names and real monthly numbers |
| 4 | what the AI actually answers | Concrete examples of questions a coach's AI handles overnight, in their voice |
| 5 | [coach name]'s app | One named story, the before and after, with screenshots |
| 6 | you already wrote it | Their existing posts, PDFs, and programs are the app. Nothing new needs writing |

Every one ends the same way: a one-line PS offering the free demo. The teaching is the gift. The PS is the ask.

## 7. Sales emails

Sent after the nurture sequence, or triggered when someone opens a demo and goes quiet. This mirrors Mode 5 in
the outreach skill, which handles the same moment in DMs.

| # | Subject | Job |
|---|---|---|
| 1 | your demo | Deliver the demo link, one line, one ask |
| 2 | the tenant problem | Problem and solution, straight down the BrandScript |
| 3 | how [coach] launched | A named customer story, structured before / change / result |
| 4 | isn't this just white label | Kill the top objection head on. Our own marketing context says this is the objection and also our best-performing content angle, so it belongs here |
| 5 | your method is a product | The paradigm shift. Content is what you post, a product is what people use. Yours is already a product waiting to be delivered like one |
| 6 | last one on this | The direct ask, with a PS that restates the whole offer in three lines for the person who only reads the PS |

Write all of these in the same voice as the DMs: no em dashes, no hashtags, short declarative sentences, and
never a benefits lecture. The banned-phrase list in the skill applies unchanged.

---

## 8. What this changes in the outreach skill

The book's funnel and our DM system are the same story told at different volumes. Four concrete edits:

1. **Add the one-liner to `SKILL.md`.** Modes 2 and 5 both need a fixed answer to "what is this exactly?" and
   currently improvise one per conversation.
2. **Replace the anonymous case study in follow-up 1** with a named story the moment we have one. It is the
   proven follow-up and it is running on our weakest evidence.
3. **Give non-repliers somewhere to go.** Today a prospect who does not reply to two follow-ups is dropped.
   With a lead generator, touch three becomes the build sheet instead of silence, and the drop becomes a
   capture.
4. **Turn on Mode 6.** Cold email is fully specified and has sent zero messages. The nurture and sales
   sequences above give the channel a job that does not depend on cold volume.

And one allocation call, straight from the numbers: Instagram has taken 436 touches for 7 replies and zero
wins, while LinkedIn produced all 3 wins from 360. The book's read on this would not be "post more on
Instagram", it would be that Instagram traffic has nowhere to land. Fix the landing place first, then judge
the channel.

---

## 9. Order of execution

The book's own finding is that partial implementation produces partial results, and that the companies that
implement the most of the checklist grow the most. So the order matters more than the polish.

1. Write the one-liner and put it everywhere this week. Zero cost.
2. Fix the website header to pass the grunt test, with the CTA repeated top right and mid-header.
3. Ship the build-sheet PDF, because we are already promising it on LinkedIn.
4. Collect three named customer stories.
5. Build the self-serve demo preview page and put the email gate on it.
6. Turn on the nurture sequence, then the sales sequence.

What to measure, in this order: email addresses captured per week (currently zero), demo previews generated,
demo-to-call rate, and only then reply rate. Reply rate is the metric we watch today because the opener is the
only thing we have. It should stop being the headline number once there is a funnel behind it.
