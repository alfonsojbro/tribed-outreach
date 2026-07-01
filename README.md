# tribed-outreach

Tribed's full outreach system as a Claude/Cowork plugin — Instagram + LinkedIn + cold email, reply and objection handling, follow-up sequences, and Gojiberry campaign automation (personalized connection notes + follow-ups, ICP filtering, and a daily IG+LinkedIn pipeline).

## What's inside

```
tribed-outreach/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # lets this repo act as its own marketplace
├── commands/
│   └── daily-outreach.md    # /daily-outreach — runs the daily Gojiberry pipeline
└── skills/
    └── tribed-outreach/
        ├── SKILL.md         # shared core: gift-first move, rule zero, banned phrases, ICP, pricing
        └── references/
            ├── instagram.md # Mode 1 — IG cold DM
            ├── replies.md   # Mode 2 — reply / objection handling (pre-demo)
            ├── followups.md # Mode 3 — follow-up sequences
            ├── linkedin.md  # Mode 4 — connection note + InMail
            ├── postdemo.md  # Mode 5 — post-demo objections → book the walkthrough
            ├── email.md     # Mode 6 — cold email + email follow-ups (Corey-informed, Tribed voice)
            └── gojiberry.md # Mode 7 — Gojiberry personalization + ICP filter + daily pipeline
```

## What changed from the old `tribed` skill

- Renamed to **tribed-outreach** and packaged as a plugin so it can be shared and versioned.
- Added **Mode 6 — cold email** (subject lines, frameworks, cadence) adapted from Corey Haines' cold-email method into the Tribed gift-first voice, and folded its transferable principles into the core (personalization must connect to the point, angle-rotating follow-ups, one low-friction ask).
- Added **Mode 7 — Gojiberry ops**: the campaign step map, per-contact `personalizedMessages` workflow, the 2,000-followers-or-creator ICP filter, and the daily IG+LinkedIn pipeline, plus a `/daily-outreach` command.

## Install (you and your VA — shared source of truth)

This repo is its own marketplace, so both of you install the same versioned plugin and stay in sync.

1. **Create the GitHub repo (once).** From inside this folder:
   ```bash
   git init
   git add .
   git commit -m "tribed-outreach v1.0.0"
   git branch -M main
   git remote add origin https://github.com/alfonsojbro/tribed-outreach.git
   git push -u origin main
   ```
   Then edit `.claude-plugin/marketplace.json` and `.claude-plugin/plugin.json` to replace `alfonsojbro` with the real repo URL, and push again. Add your VA as a collaborator on the repo.

2. **Both of you add the marketplace and install** (in Cowork/Claude Code):
   ```
   /plugin marketplace add alfonsojbro/tribed-outreach
   /plugin install tribed-outreach@tribed-outreach
   ```
   (Or add it via Settings → Capabilities.)

3. **Staying in sync.** Whoever edits the skill commits and pushes to `main`. The other runs `/plugin marketplace update tribed-outreach` (or reinstalls) to pull the new version. The Git history is your single source of truth and changelog.

## Not using GitHub yet?

You can install locally from this folder, or zip it as `tribed-outreach.skill` and click to install — but local/zip copies don't sync between two people. The Git route above is the one that keeps you and your VA on the same version.
