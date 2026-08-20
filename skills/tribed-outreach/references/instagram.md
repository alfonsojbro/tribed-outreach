# Mode 1 — Instagram cold DM

Apply the core principles at the casual end of the register: texting a friend you respect. Contractions, lowercase mid-sentence energy, no corporate polish.

## Length: no hard cap — we are running a format experiment

There is no character cap on IG cold DMs (the old ~350 rule is retired, Alfonso's call 2026-08-20). Instead, every cold DM is written in one named format from the library below, and the format is tagged on the lead so reply rates can be mined per format. Let the format decide the length: a PAS can be tight, an AIDA can breathe. Bloat is still bloat — every sentence must earn its place.

**The first line still does all the selling.** Cold DMs land in Requests, where only the opening line shows before the tap. Whatever the format, the anchor detail (rule zero) goes in line one.

## Name their app, don't sell "a demo"

Give the app THEIR name: their program, method, or brand ("the Momfidence app", "the Fuel for Life app"). A named app is real in a way "a demo" is not — it sounds like it exists and belongs to them already. Saying "demo" is allowed but no longer required; prefer the named app. The gift-first hook stays in every format: the app exists, the message offers to show it.

## Format library (rotate; tag every send)

Pick ONE format per lead. Never the same format twice in a row in a session. At send time, stamp the lead via `update_outreach_lead`:

```
patch.data.dm_format  = "aida" | "pas" | "bab" | "reveal"
patch.data.dm_chars   = <length of the sent message>
```

The tracker mining pass reads these tags to compute reply rate per format. A send without a tag is a wasted data point.

### aida — Attention, Interest, Desire, Action

Attention = their own words or a live detail from the profile. Interest = the pain that detail implies. Desire = the named app, 2-3 concrete things it holds. Action = question CTA.

Live example (sent to @katrinahausmann, 2026-08-20, ~405 chars):
> Katrina, saw you're taking on 2 more women like Hannah right now. that's exactly when coaching starts eating your dms, every check in another thread to dig up. so I turned the Momfidence Method into its own app. daily plan, core + pelvic floor progressions that actually move week to week, protein habits, your check ins all in one place. your clients open it every morning instead of waiting on you. want a look?

### pas — Problem, Agitate, Solve

Problem = what their audience or their own workflow gets wrong. Agitate = one line making it concrete. Solve = the named app. Keep it tight; PAS dies when it rambles.

Live example (sent to @featherstonenutrition, 2026-08-20, ~290 chars):
> Meghann, people screenshot your fueling posts and lose them by race week. then they nail the long run once and wing the rest of the block. so I turned your method into the Fuel for Life app, daily fueling that moves with the training block, long run tracker, race week built in. wanna see it?

### bab — Before, After, Bridge

Before = where their method lives now (dms, pdfs, posts, check ins). After = the picture of it as their app. Bridge = it already exists, question CTA.

> Katrina, right now Momfidence lives in dms and check ins. picture your clients opening a Momfidence app every morning instead, plan, core work, protein habits. it already exists. wanna see it?

### reveal — lead with the strange fact the app exists

The confession/reveal skeleton from before, sharpened: open with the app by name existing, then why they qualified, then CTA.

> Meghann, weird one, I built you an app. Fuel for Life, your carb loading and race week guidance as a real daily plan people follow. it exists, want a look?

## Reading the screenshot

Pull before writing: first name or handle (first name warms it up), the one anchor detail (rule zero), their own vocabulary for their audience ("athletes", "warriors", "mi gente"), program or challenge names — the program name doubles as the app name — language and region cues (Spanish bio, voseo, .ar links, location, flag emoji). If no anchor detail is visible, ask the user, don't pad.

## Register examples (older, pre-format-library — register reference only)

These still show the register and language range. They predate the named-app rule and the format tags; don't copy their "demo" phrasing or their skeletons verbatim.

**Confession, English, fasting coach:**
Dana, ok this is a little forward but your autophagy explainer finally made fasting click for me, and I ended up building a demo app around your method. fasting timer, AI coaching, a spot for your community, the whole thing. it's done and it's yours to look at, wanna see it?

**Reveal-first, River Plate Spanish, mindset coach:**
Mati, che, te hice una app. en serio, ya existe. me quedé pensando en tu post sobre hábitos que no dependen de la motivación y terminé armando una demo con tu método adentro, coaching con IA y seguimiento para tu gente. ¿te la paso?

**Idea-first, neutral Spanish, movement coach:**
Lucía, tu serie de movilidad de cadera está muy bien armada, se nota el método detrás. se me ocurrió que eso ya es prácticamente una app, así que hice una demo con seguimiento y guía para tus alumnos. ¿quieres verla?
