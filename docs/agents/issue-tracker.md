# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- The map is `.scratch/<feature-slug>/map.md` — the deck markdown backend discovers an effort by this file. It carries five blocks: `## Destination`, `## Notes`, `## Decisions so far`, `## Not yet specified`, `## Out of scope`.
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Current efforts

| Effort | Directory | What it is |
|---|---|---|
| Profile redesign | `.scratch/profile-redesign/` | Rewriting the profile README (spec + tickets 01–06) |

## Why local and not GitHub Issues

This repository *is* the public profile page (`CAI991108/CAI991108`). Work tickets
like "compress the certificates" would be visible to every visitor, so they stay
local. `.scratch/` is gitignored.
