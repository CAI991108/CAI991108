# AGENTS.md

This repository **is** the GitHub profile page. Whatever lands in `README.md` is
rendered at <https://github.com/CAI991108> for every visitor. Treat it as a
published artefact, not a scratchpad.

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
- Not available: `style`, `class`, `id`, `<font>`, `<iframe>`, `<video>`, `<script>`.
- Every `<img>` needs `alt`; every badge is `style=flat` at 20 px; one accent colour.
- Image budget: ≤ 150 KB each, ≤ 400 KB total. Anything up to ~400 px wide reads
  on a phone; wider must be pure decoration.
- At least two `##` headings — that is what gives readers the Outline menu.

## Layout

| Path | Contents |
|---|---|
| `README.md` | The profile itself. The only file most visitors see. |
| `research/` | Cited research notes (`github-profile-readme.md`). Committed. |
| `.scratch/` | Spec + tickets. **Gitignored**, local only. |
| `docs/agents/` | This setup: tracker, labels, domain vocabulary. |
| `docs/private/` | Personal material (resume). **Gitignored**. |
| `assets/` | Optimised images referenced by the README. |

## Build and verification

There is no build step. Verification is a checklist over the rendered artefact:

```powershell
# no inline styles survived into the file
Select-String -Path README.md -Pattern 'style=' | Measure-Object

# image weight budget
git ls-files '*.png' '*.webp' '*.jpg' | ForEach-Object { Get-Item $_ } |
  Measure-Object Length -Sum

# every image has alt text  (manual read is fine; there are few)
Select-String -Path README.md -Pattern '<img' | Select-Object -ExpandProperty Line
```

Target: `style=` count 0, image total ≤ 400 KB, at least two `##` headings.

## Delivery

Credentials cannot be supplied by a git credential helper in this environment —
the helper's prompt script needs a signal pipe the sandbox denies, so `git push`
fails with `failed to execute prompt script (exit code 66)`.

Working route (token never printed):

```powershell
$t = (gh auth token).Trim()
$b64 = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("x-access-token:$t"))
git -c http.extraHeader="Authorization: Basic $b64" push origin main
```

Also note: this machine's schannel TLS stack is broken, so `git` needs
`http.sslBackend=openssl` (already set repo-locally) and plain
`Invoke-WebRequest`/`curl.exe` will fail. Use `node -e "fetch(...)"` for HTTP.
