# Engineering / geek README components — what actually works on a GitHub profile

Research for pivoting [@CAI991108](https://github.com/CAI991108) from an academic README to an engineering/hacker one. Every component below was read at its own source (`README.md`, `action.yml`, docs source) on **2026-10-25**, and every byte count and pixel dimension in this file was measured from this machine with `fetch` on the same day. Sanitizer rules, camo proxying and badge heights are inherited from [github-profile-readme.md](./github-profile-readme.md) and not re-derived here.

The constraint that decides everything: a phone gives a README **330–370 px** of usable width (`xsmall 320` breakpoint minus padding, [Primer layout](https://primer.style/product/getting-started/foundations/layout/)). "Dense" that does not survive 330 px is not dense, it is a desktop screenshot.

---

## What "engineering style" actually means

It is not "more badges". The profiles that read as *engineer* rather than *enthusiast* do five things, and each one maps to a component that can be installed:

| Signal | What the reader sees | Component that carries it | Why it reads as engineering |
|---|---|---|---|
| **Checkable claims** | CI status, coverage, release version on a repo the reader can open | [shields](https://shields.io/badges/static-badge) dynamic badges, [codecov](https://codecov.io/gh/pyg-team/pytorch_geometric/branch/master/graph/badge.svg) badge; pattern from [rusty1s](https://github.com/rusty1s/rusty1s) | A badge that resolves to a live number can be wrong, so it is evidence. A "Python" chip cannot be wrong, so it is decoration |
| **Liveness** | Last release date, "currently building X", commit stream | `<!--START_SECTION-->` markers ([ouuan](https://github.com/ouuan/ouuan), [simonw](https://github.com/simonw/simonw)), official WakaTime badge | Dated content decays visibly; undated content is a résumé |
| **Ownership of the plumbing** | Self-hosted stats, own generator, own tooling | [jstrieb/github-stats](https://github.com/jstrieb/github-stats) into a `generated` branch; [stats-extended](https://github.com/stats-organization/github-stats-extended) self-host | Depending on nobody's free instance is itself an engineering statement |
| **One system, not a collage** | Same badge height, same accent hex, same row rhythm | `flat-square` + one `labelColor` (below) | Consistent output implies a build process |
| **Terseness** | 3–10 KB, tables, `<details>` for the tail | `<table>`, `<details>` ([DenverCoder1](https://github.com/DenverCoder1/DenverCoder1)) | Engineers read diffs; a page that could be a diff is trusted more |

The failure mode in the opposite direction is equally documented: 40+ badges in one block encode no ranking, so the reader takes none of them in ([community discussion #28686](https://github.com/orgs/community/discussions/28686)).

---

## Component catalog

| Component | What it shows | Install cost | Token / scope | Size and reliability |
|---|---|---|---|---|
| **[lowlighter/metrics](https://github.com/lowlighter/metrics)** (Action) | One SVG infographic from 47 plugins / 335 options, in 4 templates (`classic`, `repository`, `terminal`, `markdown`) | ~10 min: workflow + repo secret + `permissions: contents: write` ([action setup](https://github.com/lowlighter/metrics/blob/main/.github/readme/partials/documentation/setup/action.md)); free on public repos | `token` is **required** but "no scopes are required by default"; `read:org`, `repo`, `read:packages`, `read:project`, `gist` only for the matching plugins ([core docs](https://github.com/lowlighter/metrics/blob/main/source/plugins/core/README.md)) | Measured 3.3–168.9 KB per SVG, all **480 px** wide. Commits the file to your repo on a cron |
| **metrics shared instance** | Same, in one URL: `https://metrics.lecoq.io/<user>` | ~1 min, nothing to install | none | Free, "some plugins are not available", images cached **1 hour**, "Availability not guaranteed… small server that I rent" ([shared docs](https://github.com/lowlighter/metrics/blob/main/.github/readme/partials/documentation/setup/shared.md)). **Measured today: HTTP 500 for every user tried** |
| **[Platane/snk](https://github.com/Platane/snk)** | The contribution graph as a snake eating its cells; SVG or GIF | One workflow step + a push step ([Platane's own workflow](https://github.com/Platane/Platane/blob/master/.github/workflows/main.yml)) | **No PAT**: `github_token` defaults to `${{ github.token }}` ([action.yml](https://github.com/Platane/snk/blob/main/action.yml)) | Measured **90.4 KB, 880 × 192**; light and dark variants byte-identical in size |
| **[github-profile-summary-cards](https://github.com/vn7n24fzkq/github-profile-summary-cards)** | **Five** cards, not four: `profile-details`, `repos-per-language`, `most-commit-language`, `stats`, `productive-time` | Either the hosted API (zero install) or an Action that commits `profile-summary-card-output/<theme>/`; 16 themes listed | Action needs a PAT (`public_repo` + `read:user`); hosted API supplies its own token for you | Measured 4.5–9.5 KB, **340 × 200** per card, **700 × 200** for `profile-details`. Hosted on `*.vercel.app` |
| **[github-readme-stats](https://github.com/anuraghazra/github-readme-stats) `api/pin`** | One repo as a card: name, description, language, stars, forks | Zero for the hosted URL; the Action variant needs a workflow | Public repos need nothing; private stats need a PAT (`repo`, `read:user`) | Pin card default cache **240 h**. Upstream README now carries a CAUTION: "no longer maintained", successor is [stats-extended](https://github.com/stats-organization/github-stats-extended) |
| **[shields.io](https://shields.io/badges/static-badge)** static / dynamic | Any label+message+colour, or any live number from a GitHub/PyPI/npm endpoint | Zero — pure URL | None for static; GitHub-data badges are fetched server-side by shields | Measured 1.5–4.5 KB, **20 px** tall in `flat`/`flat-square`/`social`, 18 px `plastic`, 28 px `for-the-badge`. `cacheSeconds`: "values specified below the default will be ignored" |

### Style semantics — read out of the badge SVG itself

Same badge (`Python`, `logo=python`) rendered in all five styles, then inspected for gradients and corner radii:

| `style=` | Height | Corners | Gradients in the SVG | Reads as |
|---|---|---|---|---|
| `flat` (default) | 20 px | `rx="3"` | 2 | Soft, slightly glossy; the generic "shields badge" |
| `flat-square` | 20 px | none | **0** | Flat, square, typographic — the only style that survives 40 of them |
| `plastic` | 18 px | `rx="4"` | 2 | 2 px shorter than everything: do not mix |
| `for-the-badge` | **28 px** | none | 0 | Uppercase shout; 8 px taller than the rest |
| `social` | 20 px | `rx="2"` | 4 | A GitHub counter look; belongs on a follower count, not a stack list |

Style names and `style` values are the documented set ([shields docs](https://shields.io/badges/static-badge)); the gradient/radius counts are measured from the SVGs today.

### Measured on 2026-10-25 (this machine)

| Artefact | Bytes | Intrinsic size | Consequence |
|---|---|---|---|
| `metrics.classic.svg` | 73.2 KB | 480 × 223 | Fits the 150 KB/image budget; 1.5× a phone's width |
| `metrics.terminal.svg` | 48.8 KB | 480 × 306 | The terminal *look* as an image; 306 px is half of a 640 px phone viewport |
| `metrics.plugin.isocalendar.fullyear.svg` | **168.9 KB** | 480 × 310 | Over a 150 KB per-image budget on its own |
| `metrics.plugin.languages.svg` | 3.3 KB | 480 × 71 | The best bytes-per-information in the whole metrics catalogue |
| snk `github-contribution-grid-snake.svg` | 90.4 KB | **880 × 192** | 2.7× a phone's width; scaled to ~37 % on mobile |
| snk dark variant | 90.4 KB | 880 × 192 | Same payload for a second theme variant |
| summary-cards `stats` / `repos-per-language` | 5.9 / 4.5 KB | 340 × 200 | The only cards that fit a phone at 1:1 |
| summary-cards `profile-details` | 9.5 KB | 700 × 200 | 2× a phone; text inside becomes unreadable when scaled |
| shields `flat-square`, label + logo | 2.6 KB | 67 × 20 | ~4.9 chips per 330 px row |
| shields label-less chip (`/badge/-React-45b8d8?logo=react`) | 4.5 KB | 59 × 20 | ~5.6 per row: logo + word, no label block |
| shields two-segment chip (`/badge/Python-3776AB`) | 1.0 KB | **49 × 20** | ~6.7 per row — the narrowest chip there is |
| shields two-segment chip, long identity label (`/badge/Google%20Scholar-4285F4?logo=googlescholar`) | 1.5 KB | 111 × 20 | ~3 per row; width is text-driven, so measure before building a row |
| shields `for-the-badge` | 2.6 KB | **97.5 × 28** | 8 px taller than everything else — the classic misalignment |

A dynamic GitHub badge is bigger than a static one; all measured at 20 px tall:

| Badge | Width | Documented cache (`cache-control`) |
|---|---|---|
| `github/actions/workflow/status/.../ci.yml` | **198 px** | 60 s |
| `github/last-commit/...` | 170 px | 120 s |
| `github/commit-activity/y/...` | 156 px | 120 s |
| `github/contributors/...` | 108 px | 120 s |
| `github/v/release/...` | 94 px | 300 s |
| `github/stars/...` | 88 px | 1 800 s |
| `pypi/v/...`, `github/license/...` | 78 px each | 21 600 s / 14 400 s |

Eight such badges are ~1,000 px of row — three phone widths. This is the arithmetic behind "6–8 badges, ≤ 2 rows".

### Pinned-repo grids (`api/pin`)

GitHub itself allows only "up to six repositories and gists, combined" ([pinning docs](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/pinning-items-to-your-profile)), which is the entire reason a README pin grid exists. The endpoint is `api/pin?username=<gh>&repo=<repo>` ([readme](https://github.com/anuraghazra/github-readme-stats)), and four knobs decide whether a 2×2 looks like a grid or like debris:

| Knob | Effect | Source |
|---|---|---|
| `hide_border=true` | Removes the card outline — the single biggest visual difference between "a wall of boxes" and "a grid". It also overrides `border_color` | [common options](https://github.com/anuraghazra/github-readme-stats): "Does not apply when `hide_border` is enabled" |
| `theme=<name>` | Must be **identical on every card in the grid**, or four cards carry four palettes. `theme=transparent` is the theme-correct choice for a page that is read in both modes | [themes](https://github.com/anuraghazra/github-readme-stats); stats-extended adds `dark_github` / `light_github` pairs for `<picture>` ([readme](https://github.com/stats-organization/github-stats-extended)) |
| `description_lines_count=1..3` | Clamps the description to a fixed number of lines (values outside 1–3 are clamped by the service). This is what stops one card being 40 px taller than its neighbour and destroying the row | [pin options](https://github.com/anuraghazra/github-readme-stats) |
| `show_owner=true` | Adds the owner name; needed if the card points at someone else's repo | [pin options](https://github.com/anuraghazra/github-readme-stats) |

Alignment without CSS — three methods, in order of robustness:

| Method | Markup | Behaviour |
|---|---|---|
| Two `<td>` cells | `<table><tr><td width="50%">…</td><td width="50%">…</td></tr></table>` | Deterministic 2-up; the single row break is explicit, so a 2×2 is two `<tr>`s and a 2×4 is four. Tables collapse to one column on mobile |
| Two bare images | `<a><img align="center" …></a>` twice, separated by a newline | What [anuraghazra](https://github.com/anuraghazra/anuraghazra) ships: images are inline, so two that fit sit side by side and wrap when they do not. Zero layout markup, zero control |
| Explicit `<br>` after each pair | `img img` `<br>` `img img` | Forces the 2-up row boundary where you want it while keeping the no-table simplicity |

**Cost of a 2×4 grid:** eight camo-proxied images, eight cards' worth of bytes, on the block most likely to be below the fold. Four is the number that earns its place; put the rest in a `<details>` or let the profile's own Pinned section do the work.

### Failure modes worth designing around

The failure that matters is not "the image is slow", it is **"the image is stale or blank and nothing tells you"**. Every one of these is documented by the component itself:

| Component | Failure | What you actually see | Guard |
|---|---|---|---|
| metrics (Action) | Runner has no `docker` or `jq` | Action exits: `"docker" is not installed on current runner but is needed to run metrics` ([action.yml](https://github.com/lowlighter/metrics/blob/main/action.yml)) | Use `ubuntu-latest`; the action pulls `ghcr.io/lowlighter/metrics:v3.x` unless `use_prebuilt_image: no` |
| metrics (Action) | Token lacks a plugin's scope | **The error is drawn inside the SVG** — a broken card, on your profile, publicly ([setup docs](https://github.com/lowlighter/metrics/blob/main/.github/readme/partials/documentation/setup/action.md)) | Set `plugins_errors_fatal: yes` so the job fails instead |
| metrics (Action) | API quota below the guard (defaults: 200 REST, 200 GraphQL) | "Action will cancel itself without any errors" — a green build and a stale image ([core docs](https://github.com/lowlighter/metrics/blob/main/source/plugins/core/README.md)) | Alert on the rendered file's mtime, not on the workflow status |
| metrics (Action) | Commit loop | A new commit every run forever | Default `committer_message` carries `[Skip GitHub Action]`, which metrics itself ignores ([core docs](https://github.com/lowlighter/metrics/blob/main/source/plugins/core/README.md)) |
| metrics shared instance | Server trouble | HTTP 500 or a 1-hour-stale render; **observed 500 for every user tried on 2026-10-25** | Do not put a shared instance in a `README.md` you care about |
| summary-cards hosted API | Coverage differs from the Action | Language/stats cards aggregate only "your top 100 repositories by stars" on the hosted API; the Action uses all of them ([readme](https://github.com/vn7n24fzkq/github-profile-summary-cards)) | Use the Action if the numbers are a claim you are making |
| snk | The push step fails | The `output` branch keeps the old SVG, or 404s, and camo serves the failure quietly | Keep the push step and the generate step in the same job; check the branch after the first run |
| any `*.vercel.app` card | Host unreachable from the viewer's network | A blank image box with your `alt` text | Prefer Action-committed SVGs; they are served from your own repo |

---

## Density without noise

Eight rules, each one traceable to a measurement or a primary doc.

| # | Rule | Evidence |
|---|---|---|
| 1 | **One height for the whole page.** Pick `flat-square` (20 px, square, no gradient) and never mix in `plastic` (18) or `for-the-badge` (28) | Style table above; mixing 20 px and 28 px in one row is what makes badge walls look broken |
| 2 | **One `labelColor`, one accent.** Badge *text* colours should differ only by meaning, never by brand | [thmsgbrt](https://github.com/thmsgbrt/thmsgbrt) gives every chip its brand colour and the effect is confetti; `labelColor` is a supported query param ([shields docs](https://shields.io/badges/static-badge)) |
| 3 | **Group by row, ≤ 6 chips.** Rows are the only unit a reader parses; GitHub gives you no gaps, so separate chips with a space and rows with `<br>` | At 49–111 px per chip, 330 px fits 3–6; wrapping is otherwise uncontrolled and ragged |
| 4 | **Rank by importance, then cut.** 6–12 visible chips; everything else behind `<details>` | 40 unranked badges encode no ranking ([#28686](https://github.com/orgs/community/discussions/28686)); [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) keeps 64 in reach by burying them in 7 `<details>` blocks |
| 5 | **Prefer the two-segment or label-less form.** `/badge/Python-3776AB` (label, no message) measures 49 px; `/badge/-React-45b8d8?logo=react` (message only) measures 59 px; the three-part default measures 67 px | Measured. All three are documented shapes of the [static badge grammar](https://shields.io/badges/static-badge) ("Label, (optional) message, and color") |
| 6 | **Anything with a number in it belongs in text, not a chip.** `Python` is a claim; `v2.4.1 · 1.2k stars · CI green` is information | Compare [rusty1s](https://github.com/rusty1s/rusty1s)' four checkable badges with 43 technology chips on thmsgbrt |
| 7 | **A `<table>` is the only reliable grid.** Two `<td width="50%">` cells give a 2-up row that collapses to one column on mobile | `table/tr/td` and `width`/`valign` are in the [sanitizer allowlist](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb); [simonw](https://github.com/simonw/simonw) ships a 3-column table in production |
| 8 | **Dark mode is a config problem, not a colour problem.** Use `<picture>` + `prefers-color-scheme`, or a card family that has `dark_*` themes | [GitHub Blog: theme-adaptive images](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/); stats-extended documents `theme=dark_github` / `light_github` ([readme](https://github.com/stats-organization/github-stats-extended)) |

### Teardowns — eight dense profiles, measured

| Profile | Raw size | Structure | What to take | What not to take |
|---|---|---|---|---|
| [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) | 30.9 KB / 300 L | Zero Markdown headings — HTML `<h2>`/`<h3>` only; 64 shields refs (56 default `flat`, **8 `for-the-badge`**); 27 `github-readme-stats` URLs; 7 `<details>` with emoji `<summary>`; all sections delimited by `<!-- comment -->` markers | `<details>` as the pressure valve; comment markers as section anchors | Size and the 20 px/28 px mix; it is a second job to maintain |
| [thmsgbrt](https://github.com/thmsgbrt/thmsgbrt) | 10.5 KB / 88 L | One `<h1>`, one `<h3>`, then 43 shields refs — **39 of them `flat-square` label-less chips**, each with its own `alt`, each given its brand colour | The label-less chip trick; `alt` on every chip | Per-chip brand colours, 39 of them, unranked |
| [ruppysuppy](https://github.com/ruppysuppy/ruppysuppy) | 9.0 KB / 117 L | 6 `##` headings, then 44 shields badges, essentially all `for-the-badge`, grouped under `### Languages`, `### Front-End Development`, `### Back-End Development`, `### Automation`, `### Block-Chain Development`, `### Backend as a Service (BaaS)`, `### Design Tools` | Grouping by sub-heading makes 44 badges scannable | `for-the-badge` at 28 px × 44 = ~1,200 px of vertical badges |
| [SP-XD](https://github.com/SP-XD/SP-XD) | 6.9 KB / 103 L | No headings; 2 GIFs above the fold, 31 shields, 2 view counters, a Spotify widget inside `<details>`, 2 code fences | Nothing structural — the fence-in-`<details>` idea is the only reusable bit | GIF hero, hit counters, Spotify: motion and vanity above the fold |
| [simonw](https://github.com/simonw/simonw) | 3.4 KB / 61 L | One sentence of current work + 4 links, then a `<table>` of `<td valign="top" width="33%">`: recent releases (8 dated entries between `<!-- recent_releases starts/ends -->`), blog, TIL | The live-index pattern: dated, generated, markdown-only, ~3 KB | The 33 % columns — they stack into a long list on mobile |
| [ouuan](https://github.com/ouuan/ouuan) | 9.1 KB / 234 L | **Zero badges**; one table, one `<picture>`, three `<!--START_SECTION:…-->` blocks (blog posts, top followers, WakaTime) | Density by text + three auto-feeds; proof that badge count is not density | 234 lines with no headings means no Outline |
| [jstrieb](https://github.com/jstrieb/jstrieb) | 811 B / 20 L | Centred heading, a link to his project index, then two self-hosted SVGs from his own [`github-stats`](https://github.com/jstrieb/github-stats) `generated` branch, switched with `#gh-dark-mode-only` | Ownership: he wrote the generator and commits to a branch | The fragment switch — his light-mode `<img>` ships `…#gh-dark-mode-only#gh-light-mode-only`, a double fragment that cannot match; `<picture>` avoids this class of bug |
| [anuraghazra](https://github.com/anuraghazra/anuraghazra) | 2.8 KB / 46 L | Repo-relative header at `width="80%"`, then `<code><img height="20" alt=... src=...github/explore/topics/...></code>` wrappers, then two `api/pin` images in a row with `align="center"` (no table; wrapping does the 2-up) | Zero third-party services: the cheapest possible tech row | Nothing — but note it works because the icons are pinned `github/explore` commit URLs, i.e. frozen assets in exchange for stability |

**One signal the brief expected that the sample does not support: dotfiles.** None of the eight profiles above links a dotfiles repo (grepped for `dotfiles` across all eight `README.md` files: zero hits). The artefact is real — [mathiasbynens/dotfiles](https://github.com/mathiasbynens/dotfiles) is the canonical public one — but in this sample the engineer signal comes from *checkable* things (CI, coverage, releases, a live feed), not from a dotfiles link. Treat it as optional garnish, not a required block.

---

## Terminal/geo tricks that survive sanitization

**What actually happens to a fenced code block**, from GitHub's own stylesheet ([primer/css `src/markdown/code.scss`](https://github.com/primer/css/blob/main/src/markdown/code.scss)):

| Rule | Effect on your ASCII |
|---|---|
| `pre > code { white-space: pre; word-break: normal }` | Every space is preserved; columns line up |
| `.highlight pre, pre { overflow: auto }` | Long lines **scroll horizontally, never wrap** — so over-wide art is a sideways scroll on a phone |
| `pre { font-size: 85% }` | Code is smaller than body text; monospace advance ≈ 8.2 px at a 16 px base |
| `code, tt { white-space: break-spaces }` (inline) | Inline code wraps; fenced code does not |
| `samp { font-size: 85% }` | `<samp>` is styled by GitHub, not just allowed |

**Width budget, derived from the above:** 330 px ÷ ~8.2 px per column ≈ **40 monospace columns** on a 360 px phone, and ≈ 95 columns in an 800 px desktop column. Terminal art that assumes 80 columns is a horizontal scroll on every phone.

**Lexers that exist** (checked against [linguist `languages.yml`](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml), which is what GitHub's docs point at as the list of valid keywords):

| Fence | Resolves to | Verdict |
|---|---|---|
| ```` ```bash ```` / ```` ```sh ```` | `Shell` (aliases `sh`, `shell-script`, `bash`, `zsh`, `envrc`) | Colours commands and strings — the "real shell" look |
| ```` ```console ```` | `ShellSession` (aliases `bash session`, `console`; `tm_scope: text.shell-session`) | A *different* lexer from `bash`: tuned for transcripts with prompts. Use this one for `$ cmd` blocks |
| ```` ```text ```` | `Text` | **No highlighting at all** — the safe choice for pure ASCII art |
| ```` ```json ```` / ```` ```yaml ```` / ```` ```toml ```` / ```` ```diff ```` / ```` ```dockerfile ```` | `JSON` (aliases `geojson`, `jsonl`, `sarif`, `topojson`), `YAML`, `TOML`, `Diff`, `Dockerfile` | All present; `json` is the one that reads "config file" instantly |
| ```` ```mermaid ```` / ```` ```geojson ```` / ```` ```topojson ```` / ```` ```stl ```` | Intercepted by GitHub's **diagram renderer**, not the highlighter ([Create diagrams](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams)) | A fenced block that becomes a flow chart or an **interactive map** — the single most under-used "engineering" primitive, and it is pure text in the source |

**Which tricks look genuinely terminal:**

| Trick | Markup | Works? |
|---|---|---|
| Prompt transcript | ```` ```console ```` with `$ ` prefixes, ≤ 40 cols | Yes — the only colour-correct, selectable, searchable "terminal" |
| Plain ASCII card | ```` ```text ```` with box-drawing characters | Yes, alignment and monochrome both intact |
| Key caps | `<kbd>Ctrl</kbd>+<kbd>K</kbd>` | Yes — GitHub styles `kbd` itself; GitHub's own docs use it ([basic syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)) |
| Timeline / roadmap | `<!-- comment -->`-delimited sections inside `<details>` | Yes |
| Rendered terminal as image | metrics `template: terminal` — "a template mimicking a SSH session" ([template docs](https://github.com/lowlighter/metrics/blob/main/source/templates/terminal/README.md)) | Yes, 48.8 KB at 480 × 306: the only way to get **real colour** in a terminal hero, because the colours live inside the SVG |
| Callout boxes | `> [!TIP]` / `> [!WARNING]` | Yes — GitHub's docs source uses them and they render as coloured boxes ([example page](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks)) |
| `<pre>` + `<span style="color:…">` | — | **No.** `style` is stripped, the elements survive, everything renders in one colour ([github/markup](https://github.com/github/markup); [Vaibhavs10](https://github.com/Vaibhavs10/Vaibhavs10) is the live example) |
| Raw ANSI escapes | `\x1b[32m` | **No** — nothing interprets them; there is no terminal emulator in the render path |
| Tabs for alignment | — | **No** — tab width is not yours; use spaces |
| Colour swatches | `` `#0d1117` `` in backticks | Documented only "in issues, pull requests, and discussions" ([basic syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)) — **unverified in a profile README** |

Syntax highlighting only changes colour: "Syntax highlighting changes the color and style of source code" ([docs](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks)). It never reflows or re-orders characters, so a highlighted fence is safe for art; only the palette can fight you.

---

## Recommended architecture

Ten blocks, ordered so that a reader who stops after 200 px already knows the answer. Left column is what the block *is*; the component column is the cheapest thing that does the job.

| # | Block | Component | Budget | If it fails |
|---|---|---|---|---|
| 1 | **Hero** — name, one line of role, 4 text links (site · GitHub · Scholar · email) | Markdown + `<h1 align="center">` | 0 KB | Never |
| 2 | **Status line** — `> **Now:** building X · reading Y · open to Z` | blockquote, one line | 0 KB | Never |
| 3 | **Terminal** — 8–12 lines: `whoami`, `cat stack.txt`, `git log --oneline -3` | ```` ```console ```` fence or metrics `terminal` template | 0–49 KB | Fence always renders; the SVG needs the Action |
| 4 | **Badge grid** — 3 rows × ≤6 `flat-square` chips, one `labelColor`, grouped `langs / infra / tools` | shields static | ~2.6 KB × 15 | Never |
| 5 | **Pinned work 2×2** — four cards, two per `<td>` | `api/pin` (stats-extended or Action-committed SVG) | ~4 × 6 KB | Public instance rate-limits; Action-committed files do not |
| 6 | **One stats island** — pick exactly one of metrics / summary-cards / stats card | metrics `languages` (3.3 KB) or summary-cards `stats` (5.9 KB) | ≤ 10 KB | Shared instances go down; the Action does not |
| 7 | **Selected work table** — repo · one-line why · stack · **real CI/coverage/release badges** | `<table>` + dynamic shields | ~1 KB of text | Never |
| 8 | **Release / changelog feed** — 5 dated entries between markers | `<!--START_SECTION-->` + a generator, or simonw's `recent_releases` pattern | ~1 KB | Goes stale silently if the cron dies |
| 9 | **`<details>` tail** — full tool list (30+ chips), talks, awards, optionally a dotfiles repo | `<details>` + `flat-square` chips | ≤ 60 KB collapsed | Never |
| 10 | **Contact** — text links, `<kbd>` hint, no image badges | Markdown | 0 KB | Never |

Skeleton (the parts that are easy to get wrong):

````markdown
<h1 align="center">Zijin CAI</h1>
<p align="center">PhD, AI · CUHK-Shenzhen — systems for <b>X</b> and <b>Y</b><br>
<a href="https://cai991108.github.io/">site</a> · <a href="…">github</a> · <a href="…">scholar</a> · <a href="mailto:…">mail</a></p>

> **Now:** shipping <b>A</b> · reading <b>B</b> · open to <b>C</b>

```console
$ whoami
zijin — phd, ai. builds small fast things.
$ cat stack.txt
python · pytorch · cuda · rust · typescript · postgres
$ git log --oneline -3
```

<p align="center">
<img alt="Python" src="https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white&labelColor=0d1117"> …
</p>

<table><tr>
<td width="50%"><a href="…"><img alt="repo a" src="https://…/api/pin/?username=CAI991108&repo=a&hide_border=true&theme=…"></a></td>
<td width="50%"><a href="…"><img alt="repo b" src="https://…/api/pin/?username=CAI991108&repo=b&hide_border=true&theme=…"></a></td>
</tr></table>

<details><summary>Full toolchain (32)</summary>

…one row per domain…

</details>
````

Two hard rules that come out of the numbers: **at most one 480 px-wide SVG per screen** (metrics and snk are 480/880 px, i.e. wider than a phone), and **every image gets `alt` and an `<a href>`** — an unlinked badge is a dead end ([shields `link` param](https://shields.io/badges/static-badge) exists but wrapping in `<a>` is the portable form).

**Self-hosted vs public instance.** The reliability tradeoff is documented by the maintainers themselves: the public `github-readme-stats` endpoint "is best-effort and can be unreliable due to rate limits and traffic spikes" and self-hosting or the Action is recommended ([README](https://github.com/anuraghazra/github-readme-stats)); metrics' shared instance is "hosted on a small server that I rent… Availability not guaranteed" ([docs](https://github.com/lowlighter/metrics/blob/main/.github/readme/partials/documentation/setup/shared.md)). The cause is arithmetic: unauthenticated GitHub API calls get **60 requests/hour**, authenticated ones **5,000/hour per token** ([rate limits](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)) — a public card service serves thousands of viewers from a handful of tokens, so it must cache hard (pin cards 240 h, top-langs 144 h) and still fall over.

Self-hosting on Vercel costs: fork the repo, a Vercel project, one PAT stored as `PAT_1` with `repo` + `read:user`, and a standing job of keeping the fork synced ([README, "Keep your fork up to date"](https://github.com/anuraghazra/github-readme-stats)). The **GitHub Action variant avoids the whole problem**: it renders once per cron into a static SVG committed to your repo, so there is "no per-request API call" ([README](https://github.com/anuraghazra/github-readme-stats)) — per-view cost is one camo-cached file, and Actions minutes are free for public repositories ([billing docs](https://docs.github.com/en/billing/concepts/product-billing/github-actions)). It also sidesteps two of this machine's constraints: no Vercel account, and `*.vercel.app` is unreachable here anyway.

---

## What to avoid

| Anti-pattern | Why it fails, with the receipt |
|---|---|
| Mixing badge heights | 20 / 18 / 28 px in one row looks like a rendering bug, not a style ([measured](#measured-on-2026-10-25-this-machine)) |
| 40+ unranked chips | Encodes no priority; the reader retains none ([#28686](https://github.com/orgs/community/discussions/28686)) |
| Per-chip brand colours everywhere | Turns a system into confetti — see thmsgbrt's 39 brand-coloured chips |
| `skillicons.dev` rows | Measured 54.8 KB / 6.4 s for 15 icons in the [earlier audit](./github-profile-readme.md), for logos with no numbers attached |
| Capsule-render header + typing SVG | ~350 px of motion before the first sentence; type is unselectable and unsearchable ([prior audit](./github-profile-readme.md)) |
| View counters (`komarev`, `hits.seeyoufarm.com`) | A third-party request per page view for a number nobody acts on |
| ASCII art at 80 columns | `pre { overflow: auto }` means it scrolls sideways on a phone instead of scaling ([primer](https://github.com/primer/css/blob/main/src/markdown/code.scss)) |
| `<pre><span style="color:…">` | `style` is stripped; you get one colour ([github/markup](https://github.com/github/markup)) |
| `#gh-dark-mode-only` on `<img>` | Still supported ([2021 changelog](https://github.blog/changelog/2021-11-24-specify-theme-context-for-images-in-markdown/)), but easy to malform — jstrieb's light block carries two fragments. `<picture>` is the documented shape |
| Depending on one free public instance | Today `metrics.lecoq.io` returned **HTTP 500 for every user tried**; pin/summary cards on `*.vercel.app` are unreachable from this network at all |
| A daily bot commit per metric | The 827 empty "Updated with Dev Metrics" commits in this repo's history are the canonical example of activity theatre ([prior audit](./github-profile-readme.md)) |
| Emoji as the only heading text | The Outline menu is the only navigation a long page gets; `## 🛠️ Skills` is findable, `##` alone is not |
| An unlinked badge | Looks like a chip, behaves like nothing; wrap every `<img>` in `<a href>` |

---

## Sources

**Components (primary: their own README / action.yml / docs)**
- [lowlighter/metrics README](https://github.com/lowlighter/metrics) · [action.yml](https://github.com/lowlighter/metrics/blob/main/action.yml) (every input, the docker/`jq` runner check, `use_prebuilt_image`) · [Action setup](https://github.com/lowlighter/metrics/blob/main/.github/readme/partials/documentation/setup/action.md) (token scopes, workflow YAML, `permissions: contents: write`) · [shared instance](https://github.com/lowlighter/metrics/blob/main/.github/readme/partials/documentation/setup/shared.md) (limited plugins, 1 h cache, availability caveat) · [core plugin docs](https://github.com/lowlighter/metrics/blob/main/source/plugins/core/README.md) (`token` "no scopes required by default", `NOT_NEEDED`, `extras_css`, `extras_js` run in puppeteer at render time, `output_action`, `quota_required_*`, `plugins_errors_fatal`, `verify`, `retries`) · [terminal template](https://github.com/lowlighter/metrics/blob/main/source/templates/terminal/README.md)
- [Platane/snk README](https://github.com/Platane/snk) · [action.yml](https://github.com/Platane/snk/blob/main/action.yml) (`github_token` defaults to `github.token`; `outputs` query options `palette`, `color_snake`, `color_dots`, `color_background`) · [Platane's workflow](https://github.com/Platane/Platane/blob/master/.github/workflows/main.yml) (`target_branch: output` via `crazy-max/ghaction-github-pages`)
- [vn7n24fzkq/github-profile-summary-cards](https://github.com/vn7n24fzkq/github-profile-summary-cards) — five endpoints and their parameters, 16 themes, colour overrides, hosted-vs-Action repo coverage ("top 100 repositories by stars" vs all), PAT scopes, `profile-summary-card-output/`
- [anuraghazra/github-readme-stats](https://github.com/anuraghazra/github-readme-stats) — unmaintained CAUTION, `api/pin` + options, `hide_border`, per-card caches, "best-effort" warning, self-host env vars, "Keep your fork up to date", Action variant
- [stats-organization/github-stats-extended](https://github.com/stats-organization/github-stats-extended) — successor, domain-only migration, `dark_github`/`light_github` themes in `<picture>`
- [stats-organization/github-readme-stats-action](https://github.com/stats-organization/github-readme-stats-action) — `card`/`options`/`path`/`token`, static-SVG model
- [jstrieb/github-stats](https://github.com/jstrieb/github-stats) — self-generated `overview.svg`/`languages.svg` on a `generated` branch
- [mathiasbynens/dotfiles](https://github.com/mathiasbynens/dotfiles) — the canonical public dotfiles repo, i.e. what a "dotfiles" link points at

**Badges**
- [shields static badge](https://shields.io/badges/static-badge) — path grammar ("Label, (optional) message, and color"), `style` values, `labelColor`, `logo`, `logoColor`, `logoSize`, `cacheSeconds` (sub-default values ignored), `link`

**GitHub, first-party**
- [Creating and highlighting code blocks](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks) — Linguist selects the grammar; "find out which keywords are valid in the languages YAML file"; highlighting changes colour and style
- [github-linguist `languages.yml`](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml) — `Shell` (aliases `bash`, `sh`), `ShellSession` (aliases `console`, `bash session`), `JSON` (aliases `geojson`, `topojson`), `Text`, `YAML`, `TOML`, `Diff`, `Dockerfile`; no `Console`, `Bash`, `ANSI` or `GeoJSON` language key
- [Creating diagrams](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams) — `mermaid`, `geojson`, `topojson`, `stl` fences; maps are interactive
- [primer/css `src/markdown/code.scss`](https://github.com/primer/css/blob/main/src/markdown/code.scss) — `white-space: pre`, `overflow: auto`, `font-size: 85%`, `samp` styling
- [Basic writing and formatting syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) — Outline from ≥ 2 headings, `<kbd>` usage, colour models limited to issues/PRs/discussions
- [Pinning items to your profile](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/pinning-items-to-your-profile) — "up to six repositories and gists, combined" (why README pin grids exist)
- [Rate limits for the REST API](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api) — 60 req/h unauthenticated, 5,000 req/h authenticated
- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions) — free for public repositories on standard hosted runners
- [Theme context for images in Markdown (2021 changelog)](https://github.blog/changelog/2021-11-24-specify-theme-context-for-images-in-markdown/) — `#gh-dark-mode-only` / `#gh-light-mode-only` is documented, not folklore
- [Theme-adaptive images (GitHub Blog)](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/) — the `<picture>` + `prefers-color-scheme` snippet
- [`html-pipeline` `SanitizationFilter`](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb) · [github/markup](https://github.com/github/markup) — the allowlist and the "inline styles removed" statement

**Profiles cited** — [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) · [thmsgbrt](https://github.com/thmsgbrt/thmsgbrt) · [ruppysuppy](https://github.com/ruppysuppy/ruppysuppy) · [SP-XD](https://github.com/SP-XD/SP-XD) · [simonw](https://github.com/simonw/simonw) · [ouuan](https://github.com/ouuan/ouuan) · [jstrieb](https://github.com/jstrieb/jstrieb) · [anuraghazra](https://github.com/anuraghazra/anuraghazra) · [rusty1s](https://github.com/rusty1s/rusty1s) · [Vaibhavs10](https://github.com/Vaibhavs10/Vaibhavs10)

**Secondary** — [community discussion #28686 on badge walls](https://github.com/orgs/community/discussions/28686)

### Unverified / caveated

- **`*.vercel.app` is unreachable from this machine** (`github-readme-stats`, `github-stats-extended`, `github-profile-summary-cards` all failed at the connection level today). Their URLs, parameters, caches and defaults are from their own docs; what the cards *look* like for this account is unverified, and pin-card pixel width is therefore not stated here.
- **`metrics.lecoq.io` returned HTTP 500** ("failed to process metrics correctly") for `CAI991108`, `anuraghazra` and with `?base=header`. That is an observation from one moment, not a documented behaviour.
- **Colour swatches from inline hex** are documented only for issues/PRs/discussions; whether a profile README renders them is untested here.
- **Whether HTML `<h2>` (rather than Markdown `##`) contributes to the Outline menu** is not documented; DenverCoder1 relies on it.
- **Percentage `width` values** — the attribute name passes the sanitizer and [anuraghazra](https://github.com/anuraghazra/anuraghazra) ships `width="80%"` on his header, so percentages are at least in production use; how they behave inside a `<td>` grid was not rendered-verified here. Two `<td>` cells are the safer shape.
- **`extras_js` in a README**: metrics documents that it executes "in puppeteer context during the rendering phase", so the shipped SVG is already transformed; whether any script inside an SVG is executed when displayed by GitHub was not verified. Treat JS as a build-time tool, not a runtime one.
- **Badge-width arithmetic** assumes GitHub's markdown body at a 16 px base and Primer's 85 % code size; browsers differ by a pixel or two per column.

*Researched on 2026-10-25.*
