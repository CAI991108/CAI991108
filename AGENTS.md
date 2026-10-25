# AGENTS.md

This repository **is** the GitHub profile page. Whatever lands in `README.md` is
rendered at <https://github.com/CAI991108> for every visitor. Treat it as a
published artefact, not a scratchpad.

## Start here

Read `.scratch/profile-redesign/HANDOFF.md` before touching anything: current
commit, locked design decisions, open threads, and this machine's workarounds.

## Agent skills

### Issue tracker

Work tickets live as local markdown under `.scratch/profile-redesign/issues/`, one file per ticket, with `Blocked by:` lines declaring edges. See `docs/agents/issue-tracker.md`.

### Triage labels

Default five-role vocabulary, recorded in each ticket's `Status:` line. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context. The page vocabulary (hero, dashboard island, chip, Selected work) is defined in `docs/agents/domain.md`. See `docs/agents/domain.md`.

## Rendering constraints (non-negotiable)

GitHub sanitises README HTML. These were verified against `html-pipeline`'s
`SanitizationFilter` and live pages — see `research/github-profile-readme.md`:

- **Never write `style=`.** It is stripped, silently. Layout only via `<p align>`,
  `<table>`, and `<img width|align>`.
- Allowed and useful: `h1`–`h6`, `p align`, `img` (`src`/`width`/`height`/`alt`),
  `table`/`tr`/`td`, `details`/`summary`, `br`, `hr`, `sub`, `sup`, `kbd`, `picture`.
- GitHub's renderer appends `max-width: 100%` to every `<img>`, so a wide image
  shrinks to the container instead of overflowing.
- Every `<img>` needs `alt`. Badges are `flat-square`, 20 px tall, coloured by
  section (Mocha accents), at most 6 per row.
- Image budget: ≤ 150 KB each, ≤ 400 KB total. Anything up to ~400 px wide reads
  on a phone; wider must be pure decoration (the snake is the one exception).
- At least two `##` headings — that is what gives readers the Outline menu.

## Layout

| Path | Contents |
|---|---|
| `README.md` | The profile itself. The only file most visitors see. |
| `research/` | Cited research notes. Committed. |
| `.scratch/` | Spec + tickets + preview harness. **Gitignored**, local only. |
| `docs/agents/` | This setup: tracker, labels, domain vocabulary. |
| `docs/private/` | Personal material (resume). **Gitignored**. |
| `assets/` | Optimised images referenced by the README. |

## Build and verification

There is no build step. Verification is a checklist over the rendered artefact:

```powershell
# no inline style attributes survived  (badge URLs also contain "?style=" — match the attribute)
Select-String -Path README.md -Pattern 'style\s*=\s*"' | Measure-Object

# image weight budget
git ls-files '*.png' '*.webp' '*.jpg' | ForEach-Object { Get-Item $_ } |
  Measure-Object Length -Sum

# outline headings
Select-String -Path README.md -Pattern '^## ' | Measure-Object

# 360 px viewport: a plain --window-size screenshot lies on Windows (the window is
# clamped wider, then cropped). Drive CDP instead and read documentElement.scrollWidth.
node .scratch/preview/cdp-360.mjs https://github.com/CAI991108 .tmp/check.png 360
```

Target: `style="` count 0, image total ≤ 400 KB, at least two `##` headings,
`scrollWidth` ≤ viewport.

## Preview

GitHub renders the README, so previews must go through GitHub's own renderer:

```powershell
$env:GH_CONFIG_DIR = (Join-Path $PWD ".tmp\gh-config")
pwsh -File .scratch/preview/build.ps1      # renders README.md to .scratch/preview/
node .scratch/preview/serve.mjs            # http://127.0.0.1:8123/
```

The server does **not** survive a harness restart — rebuild, restart it, then open
the URL in the sidebar. See `.scratch/profile-redesign/HANDOFF.md` §7.

## Delivery

Credentials cannot be supplied by a git credential helper in this environment —
the helper's prompt script needs a signal pipe the sandbox denies, so `git push`
fails with `failed to execute prompt script (exit code 66)`.

Working route (token never printed). `GH_CONFIG_DIR` is required because `gh`
cannot write to AppData here:

```powershell
$env:GH_CONFIG_DIR = (Join-Path $PWD ".tmp\gh-config")
$t = (gh auth token).Trim()
$b64 = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("x-access-token:$t"))
git -c http.sslBackend=openssl -c http.extraHeader="Authorization: Basic $b64" push origin main
```

Also note: this machine's schannel TLS stack is broken, so `git` needs
`http.sslBackend=openssl` (already set repo-locally) and plain
`Invoke-WebRequest`/`curl.exe` fail on the public internet. Use
`node -e "fetch(...)"` for HTTP; localhost requests are fine.
