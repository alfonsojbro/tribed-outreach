---
name: daily-outreach
description: Run the daily Tribed IG + LinkedIn Gojiberry pipeline — filter new leads to ICP, then write personalized connection notes and follow-ups onto the survivors.
---

Run the Tribed daily outreach pipeline. Follow references/gojiberry.md (Job C) exactly.

For each active Gojiberry campaign/list (LinkedIn first, then Instagram):

1. Pull the leads added since yesterday (`list_contacts` with `dateFrom` = today, or diff ids).
2. ICP filter (Job B): enrich followers via Apify `harvestapi/linkedin-profile-scraper` and keep only `followerCount >= 2000` OR `creator` OR `influencer`. Remove the rest from the list.
3. Personalize the survivors (Job A) in the Tribed voice, anchored on one real detail (rule zero):
   - step 0 connection note (<280 chars, no CTA, no price)
   - step 2 follow-up 1 (light bump)
   - step 5 follow-up 2 (new angle / soft close)
   - steps 1/4/6 emails only if email is enabled for that campaign (Mode 6 / email.md)
   Write with `update_contact({ id, personalizedMessages: [...] })`.
4. Hold any lead with no usable anchor for manual review instead of shipping a generic message.
5. Report: new leads found, kept vs removed, personalized, and held-for-review, per channel.

Do not enroll out-of-ICP leads. Do not send a message that fails rule zero.
