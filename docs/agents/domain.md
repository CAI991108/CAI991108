# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

- **`GLOSSARY.md`** at the repo root, or
- **`GLOSSARY-MAP.md`** at the repo root if it exists: it points at one `GLOSSARY.md` per context. Read each one relevant to the topic.
- **`docs/adr/`**: read ADRs that touch the area you're about to work in.

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The `/domain-modeling` skill creates them lazily when terms or decisions actually get resolved.

## File structure

Single-context repo (this one):

```
/
├── GLOSSARY.md          ← created lazily, if ever
├── docs/adr/            ← created lazily, if ever
├── README.md            ← the artefact under construction
├── research/            ← cited research notes
└── docs/agents/         ← this configuration
```

## This repo's vocabulary

The page has a handful of terms that must stay consistent between the spec and the
README. Treat these as the canonical names:

| Term | Means |
|---|---|
| **dashboard island** | The single `## Coding activity` block holding every stats card. Deliberately one block, never scattered. |
| **hero** | The first ~120 px: name, role, direction, four text links. |
| **chip** | A label-less shields badge (`/badge/<message>-<color>`), used for focus areas. |
| **Selected work** | The curated project table. Not "projects", not "portfolio". |

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined above. Don't drift to synonyms.

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders), but worth reopening because…_
