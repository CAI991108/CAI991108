# GitHub Profile README redesign — research notes for Zijin CAI (@CAI991108)

This file collects what primary sources actually say about GitHub profile READMEs, so the redesign can be built on verified constraints instead of folklore. Everything below was checked on 2026-10-24 against: GitHub's own docs (profile READMEs, writing/formatting, Camo, large files, Actions workflow syntax), GitHub's public sanitizer source ([`html-pipeline` `SanitizationFilter`](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb)) and [`github/markup`](https://github.com/github/markup), a first-party [GitHub blog post on dark/light README images](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/), the [shields.io badge docs](https://shields.io/badges/static-badge) and [logo docs](https://shields.io/docs/logos), the [simple-icons slug list](https://github.com/simple-icons/simple-icons/blob/master/slugs.md), [WakaTime's API docs](https://wakatime.com/developers), and the actual source of the actions/services in use (`github-readme-stats`, `waka-readme-stats`, `waka-readme`, `skill-icons`, `github-profile-views-counter`). It also includes direct HTTP probes of every image endpoint in the current `README.md`, a scan of the repo's git history, and render-level checks of live GitHub pages to see which HTML survives. Where a statement is an observation from those probes rather than a documented rule, it says so; where something could not be verified from this machine, that is flagged too (notably `*.vercel.app`, which is unreachable from here — see [§ Rendering constraints](#rendering-constraints-verified)).

---

## The anatomy that works

A profile README has one job that a GitHub profile page does not already do: it explains **what you work on, how good you are at it, and where to find the proof** — in that order — to someone who arrived from a paper, a repo, or a recruiter's shortlist. The page around it already shows pinned repos, the contribution graph, follower counts and the org you belong to, so a README that restates those numbers adds nothing.

**The five-second test.** Nielsen Norman Group's eye-tracking work on web scanning ([F-shaped pattern](https://www.nngroup.com/articles/f-shaped-pattern-reading-web-content/)) describes the behaviour that matters here: readers land on the first lines, scan down the left edge, and leave. Practically, that means the first screen must contain (a) name, (b) one-line role/affiliation, (c) 3–5 working links, and nothing animated above it. The current README spends ~350 px of vertical space on a 300 px capsule-render banner plus a typing-SVG line before any information appears ([README.md:1-9](../README.md#L1-L9)) — on a 360 × 640 px phone that is the whole first screen.

**Headings are navigation, not decoration.** GitHub generates an "Outline" menu for Markdown files with headings — documented in [Basic writing and formatting syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax): "When you use two or more headings, GitHub automatically generates a table of contents that you can access by clicking the 'Outline' menu icon within the file header." Profiles that read well use 4–6 `##`/`###` sections; profiles that read badly use zero headings and 40 inline badges instead. Note the exception: [xinychen](https://github.com/xinychen/xinychen) deliberately uses **bold** labels instead of headings, which keeps the Outline clean for a short page.

**Proof-of-work beats self-description.** The strongest profiles point at things a reader can click and check: [simonw](https://github.com/simonw/simonw) auto-generates a "Recent releases" list between `<!-- recent_releases starts -->` / `<!-- recent_releases ends -->` markers inside a table cell; [rusty1s](https://github.com/rusty1s/rusty1s) leads with real project badges (PyPI version, CI status, docs, coverage) for [PyTorch Geometric](https://github.com/pyg-team/pytorch_geometric); [xinychen](https://github.com/xinychen/xinychen) lists 5–6 formatted citations under **Selected Publications**. None of them describe themselves as "passionate" or "hardworking".

**Two lengths work; the middle is where profiles die.** [rasbt](https://github.com/rasbt/rasbt) is 580 characters / 13 lines and works because every line is a verifiable claim (books, blog, site). [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) is 30,886 characters / 300 lines and works because it is a *maintained* dashboard (64 shields badges, 26 custom-icon-badges, 20 self-hosted stats cards). A 3–6 KB README with 6 stats images and no argument is neither: it is the length of a dashboard without the upkeep, and the length of a bio without the density.

**Order that the strong profiles actually use:** identity → affiliation/role → "what I work on now" (≤ 3 bullets) → selected work with a one-line "why it matters" → publications/preprints (research profiles) → current availability → contact. Contact should be **text links**, not badges: an image badge containing a phone number is both unreadable at mobile width and a permanent public disclosure (see [§ Anti-patterns](#anti-patterns)).

**Also documented, and easy to miss:** the profile README only appears when the repo is public, named exactly like the username, and contains a non-empty root `README.md` ([Managing your profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)).

---

## Teardowns

Eight profiles, each checked by reading the raw `README.md` (sizes are raw bytes) and, where relevant, the services it calls.

### 1. [rasbt](https://github.com/rasbt/rasbt) — 580 B / 13 lines — the "authority by prose" archetype
Four short paragraphs: who he is (AI research engineer, LLMs), what the work is (books, open source), then a flat `#### Links` list of blog, website, X, LinkedIn. No images, no badges, no stats.
**Copy:** the discipline of ending with a plain link list; the confidence of not needing decoration.
**Don't copy:** nothing here transfers badly — but it only works because the name already carries weight. A PhD student needs the proof section that rasbt can omit.

### 2. [simonw](https://github.com/simonw/simonw) — 3,433 B / 61 lines — the "live index" archetype
Opens with one sentence of current work and four links, then a `<table>` with three `<td valign="top" width="33%">` columns: *Recent releases*, *On my blog*, *TIL*. The release list is regenerated between HTML comment markers, so the page is always current without hand editing.
**How:** three-column table (works because `table`/`td` and the `valign`/`width` attributes are allowed — see [§ Rendering constraints](#rendering-constraints-verified)); comment markers for a generator; `align="right"` also used elsewhere in the file.
**Copy:** the marker-comment pattern for anything auto-updated; the three-column density at the bottom of the page.
**Don't copy:** the 33 % columns are unreadable on mobile — GitHub stacks them, and the effect is a long list; fine for a 60-line page, not for a 300-line one.

### 3. [rusty1s](https://github.com/rusty1s/rusty1s) — 3,073 B / 42 lines — researcher + library maintainer
Reference-style link definitions at the top (`[pypi-image]:`, `[docs-image]:` …), then a compact block of real badges for PyTorch Geometric (PyPI, CI, docs, coverage) and links to the paper site, Google Scholar, Medium.
**Copy:** link-reference style keeps the visible text clean; badges that assert something checkable (version, CI, coverage) rather than vanity counters; a direct `scholar.google.de` link instead of a fake "Google Scholar badge" image.
**Don't copy:** the badge cluster is only justified because the project is the point — do not replicate it for personal stats.

### 4. [xinychen](https://github.com/xinychen/xinychen) — 4,735 B — academic, closest to CAI991108's case
Badge row built from **shields static badges with two path segments** (message + colour, no label): `https://img.shields.io/badge/Google-Scholar-blue`, `https://img.shields.io/badge/LinkedIn-0077B5`, `https://img.shields.io/badge/YouTube-FF0000`. A stats card floated right with `<img align="right" …>`. Then: affiliation sentence (MIT postdoc, PhD from U. Montreal), three bullets with emoji markers, and **Selected Publications** with full citation lines.
**Copy:** the whole shape — affiliation → 3 bullets → publications → links; `<img align="right">` is a legal, cheap way to keep a card out of the reading column; the two-segment badge trick gives label-free chips that read like buttons.
**Don't copy:** emoji bullets are a taste call; and the floated stats card duplicates what the profile page already shows.

### 5. [NielsRogge](https://github.com/NielsRogge/NielsRogge) — 6,810 B / 78 lines — AI/ML research flavour, zero badges
`### Hi there 👋`, one short bio, then a *list of models he contributed to Transformers* (TAPAS, ViT, DEiT …), each linking to Hugging Face docs, plus arXiv links for the papers. 55 huggingface.co links, 10 arxiv.org links, no shields, no stats.
**Copy:** "here are the specific artifacts I built, each linked" is the single most convincing thing an ML profile can do — and it costs 6 KB of text.
**Don't copy:** no headings at all means no Outline; and the bio opens with age/location, which is dead weight.

### 6. [Vaibhavs10](https://github.com/Vaibhavs10/Vaibhavs10) — 6,322 B / 24 lines — the terminal-card gimmick (and why it half-works)
A single `<pre>` block containing a Rich/typer-style box-drawing card with `<span style="color: #008080">` colouring.
**Verified behaviour:** GitHub strips the `style` attributes and keeps the `<pre>` and `<span>` elements — I fetched the rendered profile page and the HTML contains `<pre><span>…` with **no** `font-family` and no `Menlo` string anywhere. So the card renders in GitHub's monospace font, in one colour. The alignment survives; the colour does not.
**Copy:** nothing structural — but it is a useful reminder that "it looked right in my terminal" is not a rendering model.
**Don't copy:** colour-dependent ASCII art.

### 7. [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) — 30,886 B / 300 lines — the dashboard, done at maximum
Profile image, typing-SVG, then a long sequence of tables of custom badges (`custom-icon-badges.demolab.com`, 26 uses), 64 shields badges, YouTube video cards (`ytcards.demolab.com`), and **20 calls to his own deployment** `denvercoder1-github-readme-stats.vercel.app`. Also uses `<!--START_SECTION:…-->` markers and `<details>` blocks.
**Copy:** the two genuinely good ideas — self-host the stats cards (he does not depend on the public instance) and put long lists behind `<details>` so they collapse.
**Don't copy:** the volume. This profile is a second job to maintain, and its information density per pixel is low; it works for a creator with a YouTube channel and would read as noise for a researcher.

### 8. [thmsgbrt](https://github.com/thmsgbrt/thmsgbrt) — 10,460 B / 88 lines — the "everything badge" style, and a layout trick worth stealing
43 shields badges in `flat-square` style, most in the **label-less** form `https://img.shields.io/badge/-React-45b8d8?style=flat-square&logo=react&logoColor=white` — a leading `-` produces an empty label and a chip that is just logo + word. Flags rendered inline as tiny PNGs, an emoji GIF in the `<h1>`.
**Copy:** the label-less badge pattern for a compact tech row; `flat-square` keeps 43 badges visually quiet.
**Don't copy:** 43 badges is a wall — the reader cannot tell which five matter.

**Also worth a glance:** [anuraghazra](https://github.com/anuraghazra/anuraghazra) (2,804 B) uses a repo-relative header image `./assets/gh-readme-header.png` and inline `<code><img height="20">` icons — the cheapest way to get a tech row with no external service; [blackcater](https://github.com/blackcater/blackcater) hosts its own social SVGs at `github.com/blackcater/blackcater/raw/main/images/*.svg` and sizes them with `height="40"`.

### At a glance

| Profile | Raw size | Archetype | Third-party renderers | Copy this | Not this |
|---|---|---|---|---|---|
| rasbt | 580 B | authority by prose | 0 | brevity, plain link list | omitting proof (needs reputation) |
| simonw | 3.4 KB | live index | 0 (generated by own tooling) | comment-marker auto-sections, 3-column tail | 33 % columns on mobile |
| rusty1s | 3.1 KB | researcher + OSS | 4 (PyPI, CI, docs, coverage) | link-reference style, checkable badges | badge cluster without a project |
| xinychen | 4.7 KB | academic | 2 (shields, one stats card) | affiliation → bullets → publications | emoji bullets, floated stats card |
| NielsRogge | 6.8 KB | ML research | 0 | "artifacts I built, each linked" | no headings, age/location in bio |
| Vaibhavs10 | 6.3 KB | terminal card | 0 | — (colour does not survive) | colour-dependent ASCII art |
| DenverCoder1 | 30.9 KB | dashboard | 3 hosts, 20 cards | self-hosted stats, `<details>` | sheer volume |
| thmsgbrt | 10.5 KB | everything-badge | 3 hosts, 43 badges | label-less chip trick, `flat-square` | 43 badges with no ranking |

---

## Component catalog

| Component | What it shows | URL pattern | Requirements | Cost / reliability |
|---|---|---|---|---|
| shields static badge | Any label + message + colour | `https://img.shields.io/badge/<label>-<message>-<color>?style=&logo=&logoColor=&labelColor=` ([docs](https://shields.io/badges/static-badge)) | None. Pure URL | Generated per request by shields, then CDN-cached; no external data, so it cannot "break" semantically. Heights (measured): flat 20, flat-square 20, social 20, plastic 18, for-the-badge 28 px. `cacheSeconds` is a query param but values below the badge's default are ignored |
| shields 2-segment badge | Message + colour only (no label) | `https://img.shields.io/badge/<message>-<color>` | None | Same. Reads like a button/chip; good for Scholar/ORCID/HF links |
| shields dynamic JSON | Any number from any JSON API | `https://img.shields.io/badge/dynamic/json?url=&query=&label=&color=` ([docs set](https://shields.io/badges/dynamic-json-badge)) | A public JSON endpoint + a JSONPath `query` | Depends on a third party staying up and keeping its schema; shields caches. Only worth it if you actually want a live number (e.g. citation count) |
| github-readme-stats stats card | Stars, commits, PRs, rank | `https://github-readme-stats.vercel.app/api?username=<gh>` | Public GitHub API; private stats need a self-hosted PAT | Public instance documented as "best-effort and can be unreliable due to rate limits and traffic spikes" ([readme](https://github.com/anuraghazra/github-readme-stats)); default cache 24 h, `cache_seconds` clamped to 21600–86400 |
| …top-langs card | Language byte shares | `…/api/top-langs/?username=<gh>&layout=donut-vertical` | Same | Default cache 144 h. Measures *bytes of code in repos*, not skill |
| …WakaTime card | Coding time by language | `…/api/wakatime?username=<wakatime-username>` | WakaTime profile **public**, with BOTH "Display code time publicly" and "Display languages, editors, os, categories publicly" enabled; up to 24 h before a new account appears | Default cache 24 h. `username` is the *WakaTime* username, not the GitHub handle |
| …pin card | One repo as a card | `…/api/pin/?username=<gh>&repo=<repo>` | Public repo | Default cache 240 h |
| github-readme-stats Action | Same cards, as committed SVGs | `readme-tools/github-readme-stats-action` | Workflow + `GITHUB_TOKEN` (PAT for private stats) | The README's own recommended alternative to the public instance; no per-view API calls |
| streak-stats | Current/longest streak | `https://streak-stats.demolab.com?user=<gh>` | Public contributions | Live service; measured 7.0 KB / 1.9 s in my probe |
| activity-graph | 31-day contribution line | `https://github-readme-activity-graph.vercel.app/graph?username=<gh>` | Public contributions | Vercel-hosted; unreachable from this machine (see constraints) |
| WakaTime badge (official) | "Total time coded since <date>" | `https://wakatime.com/badge/user/<uuid>.svg` | A WakaTime account; the badge URL contains your user UUID | Returns a 20 px-tall SVG, `cache-control: max-age=3600`. Verified live: `width="190.4" height="20"`, 200 OK |
| waka-readme-stats (action) | Writes WakaTime stats into the README between markers | `anmol098/waka-readme-stats@master` | Secrets `WAKATIME_API_KEY` (an **OAuth token**, not the plugin key) + `GH_TOKEN` (PAT; `user` scope for the GitHub-data sections) | Commits to your repo on a cron — this is what produces daily bot commits ([readme](https://github.com/anmol098/waka-readme-stats)) |
| waka-readme (action) | Minimal WakaTime section | `athul/waka-readme@master` | `WAKATIME_API_KEY`; `GH_TOKEN` only if not the profile repo; needs `<!--START_SECTION:waka-->` markers ([readme](https://github.com/athul/waka-readme)) | Same class of tool; WakaTime API keys are write-only, so an OAuth token is required |
| capsule-render | Waving/gradient header with text | `https://capsule-render.vercel.app/api?type=waving&color=&height=&section=header&text=&fontSize=` | None | Vercel-hosted; a 300 px tall image is 300 px of scroll on every device |
| readme-typing-svg | Animated typing line | `https://readme-typing-svg.demolab.com?lines=&font=&size=` | None | Measured 5.4 KB / 1.4 s. Text is not selectable or searchable, and it is the only motion on the page |
| skillicons | One SVG row of tech logos | `https://skillicons.dev/icons?i=js,html,css&theme=light` ([readme](https://github.com/tandpfun/skill-icons)) | None | Measured **54,862 B and 6.4 s** for the 15 icons currently used — the heaviest single element in the README, for the least information |
| komarev view counter | Profile view count | `https://komarev.com/ghpvc/?username=<gh>` | None | Measured 914 B / 2.3 s; returns 3,271 for this profile. Vanity metric; also a third-party request on every page load |
| GitHub-native alternative | Anything already on the profile | Pinned repos, contribution graph, followers | None | Zero bytes, zero third parties, always in sync |

### Academic / identity components

| Identity | Official badge endpoint? | Practical implementation | Notes |
|---|---|---|---|
| Google Scholar | No — Scholar publishes no badge and no API | shields 2-segment chip, e.g. `https://img.shields.io/badge/Google%20Scholar-4285F4?logo=googlescholar&logoColor=white`, linking to the `citations?user=…` URL | A citation *count* can only be shown by scraping (fragile, against Scholar's terms) — do not build a dynamic badge on it |
| ORCID | Brand assets exist, but I could not load the [brand-guidelines page](https://info.orcid.org/brand-guidelines/) from this network (403) — no dynamic endpoint verified | shields chip with `logo=orcid` (`#A6CE39` brand colour) | Lowest-risk, highest-recognition academic link after Scholar |
| Semantic Scholar | Has an [official API](https://api.semanticscholar.org/api-docs/), no badge service | shields chip, or a `badge/dynamic/json` count if you really want a live number | API is free but rate-limited and occasionally slow — a static chip is safer |
| arXiv | [Official API](https://info.arxiv.org/help/api/index.html), no badge | shields chip `arXiv-<id>-B31B1B` linking to the abstract page | Per-paper links in a publications list beat one arXiv badge |
| DBLP | [Official search API](https://dblp.org/faq/How+to+use+the+dblp+search+API.html), no badge | shields chip `dblp-<name>-004F9F` linking to the `pid` page | Good "publication record" link for CS; less known outside CS |
| Hugging Face | No profile badge service (shields `logo=huggingface` works) | shields chip linked to the profile | For model/dataset artefacts, link the specific repo, not the profile |
| LinkedIn | No official badge | shields chip **without** `logo=linkedin` (slug removed from simple-icons — verified) | Use the word "LinkedIn" in the label, or a custom base64 logo |
| X / Twitter | No official badge | shields chip with `logo=x` | Works; the `x` slug exists |
| Personal site | None needed | Plain Markdown link, first in the row | Cheapest credibility signal; use the canonical `https://cai991108.github.io/` |

---

## Rendering constraints (verified)

**`style` is not allowed — it is stripped, not honoured.** `github/markup` states the pipeline plainly: "The HTML is sanitized, aggressively removing things that could harm you and your kin—such as `script` tags, inline-styles, and `class` or `id` attributes" ([README](https://github.com/github/markup)). The allowlist confirms it mechanically: in [`SanitizationFilter::WHITELIST`](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb) the `:attributes` hash lists per-element attributes (`'img' => ['src','longdesc']`, `'a' => ['href']`, `'div' => ['itemscope','itemtype']`) plus a global `:all` list — and `style` appears in neither.
→ Therefore `<div style="display:flex; flex-direction:column; align-items:center; gap:20px">` (as used in the current README at [line 41](../README.md#L41)) degrades to a plain block `<div>`: no flex, no gap, no centring, no borders, no shadow. All four certificates simply stack, left-aligned, at their `width` attributes.

**Verified at render level, not just in source.** I fetched the rendered profile of a README that uses `<pre style="font-family:Menlo…">` with `<span style="color:#008080">` children: the served HTML is `<pre><span>…</span></pre>` — every `style` gone, the elements kept, and the strings `Menlo` / `font-family` absent from the page. So: `span` and `pre` survive as elements; their styling does not.

**What the allowlist does permit** (quoted from the `:elements` list): `h1 h2 h3 h4 h5 h6 h7 h8 br b i strong em a pre code img tt div ins del sup sub p ol ul table thead tbody tfoot blockquote dl dt dd kbd q samp var hr ruby rt rp li tr td th s strike summary details`. The `:all` attribute list includes `align alt width height border valign colspan rowspan title media open dir lang start`. So these are all safe: `<table>` layouts, `<details>`/`<summary>` (with `open`), `<kbd>`, `<sub>`/`<sup>`, `<img align="right" width="320" alt="…">`, `<br>`, `<hr>`. Inline `style`, `class`, `id`, `<font>`, `<iframe>` and `<script>` (contents removed) are not available to you.

| Markup | Works in a README? | Evidence |
|---|---|---|
| `<h1>`–`<h6>` (and `h7`, `h8`) | Yes | `:elements` list; also triggers the Outline menu ([docs](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)) |
| `<p align="center">` | Yes | `align` is in `:all` |
| `<img src width height alt align>` | Yes | `img → [src, longdesc]` plus global `alt/width/height/align/title` |
| `<div>` | Element yes, any styling no | `div → [itemscope, itemtype]` only |
| `style="…"` on anything | **No — stripped** | not in `:attributes`; [github/markup README](https://github.com/github/markup); verified on a live rendered page |
| `<span>` | Element survives (attributes stripped) | verified on a live rendered page, though `span` is absent from the public `:elements` list |
| `<table>` / `<tr>` / `<td valign width>` | Yes | `table/thead/tbody/tfoot/tr/td/th` in `:elements`; `valign/width/colspan/rowspan` in `:all`; [simonw](https://github.com/simonw/simonw) uses it in production |
| `<details>` / `<summary>` (+`open`) | Yes | both in `:elements`; `open` in `:all` |
| `<kbd>`, `<sub>`, `<sup>`, `<br>`, `<hr>`, `<s>`, `<ins>`, `<del>` | Yes | all in `:elements` |
| `<picture>` / `<source media>` | Yes | [GitHub Blog](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/) — absent from the public allowlist, so treat that list as a lower bound |
| `<iframe>`, `<video>`, `<script>`, `<style>`, `<font>` | No | not in `:elements`; `script` contents are removed entirely. `<video>` is likewise absent — GitHub's own uploaded-video player is its asset pipeline, not raw HTML |
| Markdown `![alt](relative/path.png)` | Yes | [relative links docs](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) |

**`<picture>` + `media="(prefers-color-scheme: dark)"` works and is GitHub's own recommendation.** A first-party GitHub Blog post gives the exact snippet for READMEs: `<picture> <source media="(prefers-color-scheme: dark)" srcset="dark-mode-image.png"> <source media="(prefers-color-scheme: light)" srcset="light-mode-image.png"> <img alt="Fallback image description" src="default-image.png"> </picture>` ([GitHub Blog](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/)). *Caveat, stated honestly:* the public `SanitizationFilter` allowlist above does **not** list `picture` or `source`, and it also does not list `span` — yet `span` demonstrably survives in production. Treat the public allowlist as a lower bound on what GitHub allows, and the blog post as the authoritative word on `<picture>`. The parallel `#gh-dark-mode-only` fragment trick (used by e.g. [skill-icons' own README](https://github.com/tandpfun/skill-icons)) is widely used but I found **no** first-party documentation for it; prefer `<picture>`.

**Relative image paths are supported and resolved against the branch being viewed.** "GitHub will automatically transform your relative link or image path based on whatever branch you're currently on… Links starting with `/` will be relative to the repository root. You can use all relative link operands, such as `./` and `../`" ([docs](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)). The same page's Images section recommends relative links for images that live in the repo.
→ `nv-cert2.png` and `Completed 5-Day Gen AI Intensive.png` resolve, but a filename with spaces in an `src` is fragile (it must be percent-encoded to be correct); `raw.githubusercontent.com` links also work but bypass the repo-relative resolution and pin a branch name.

**External images are proxied and cached.** GitHub serves images through Camo: "Camo generates an anonymous URL proxy for each file… On GitHub.com, the URL starts `https://<subdomain>.githubusercontent.com/`" ([About anonymized URLs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-anonymized-urls)). Consequences: external images are fetched on GitHub's side (a slow origin = a slow image), the URL you write is not the URL the browser loads, and caching means an updated image at the same URL may not appear immediately.

**Body text has one font, images have theirs.** Because `style`/`class`/`<font>` are unavailable, all Markdown text renders in GitHub's system UI font stack — you cannot set a typeface. Badge and card SVGs are pre-rendered images with their own baked fonts (the WakaTime badge SVG declares `font-family="DejaVu Sans,Verdana,Geneva,sans-serif"`). So: match *colour and height* across badges, not typefaces.

**Sizes and limits that are documented.** Files: 50 MiB triggers a Git warning, 100 MiB is blocked, 25 MiB max when adding a file via the browser ([About large files](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)). Workflows: files "must have either a `.yml` or `.yaml` file extension" and must live in `.github/workflows` ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)). I found **no** documented per-image or per-README byte limit for Markdown rendering — so the practical budget is set by mobile patience, not by a hard cap.

**Widths (observational).** GitHub's layout tokens are `xsmall 320 px, small 544 px, medium 768 px, large 1012 px, xlarge 1280 px, xxlarge 1400 px`, with 16 px content padding (24 px at ≥ xlarge) ([Primer layout](https://primer.style/product/getting-started/foundations/layout/)). The desktop README column lands in the 700–1000 px band depending on viewport and sidebar; a 360–400 px phone gives roughly 330–370 px of usable width after padding. Design rule that follows: anything wider than ~800 px is dead pixels on desktop, and anything wider than ~330 px is scaled down (and unreadable if it contains text) on a phone.

**One caveat about this machine:** `github-readme-stats.vercel.app`, `capsule-render.vercel.app` and `github-readme-activity-graph.vercel.app` resolve to sinkhole IPs here and could not be fetched, so their live output for this account is unverified. `streak-stats.demolab.com`, `komarev.com`, `skillicons.dev`, `readme-typing-svg.demolab.com` and `wakatime.com/badge/...` all answered 200 and were measured (numbers in the catalog table).

---

## Anti-patterns

1. **Empty daily bot commits.** This repo has **861 commits, 827 of them titled "Updated with Dev Metrics" (96 %)**, one per day, and `git log --name-only` over the most recent 25 of them reports **0 files touched**; `README.md` itself was last modified 2025-04-27. This is the textbook "auto-commit noise" pattern: it inflates the contribution graph with green squares that carry no work. It also has a second cause here — `.github/workflows/waka_readme` has **no `.yml` extension**, so it never runs, while the active `wakatime.yml` has none of the `SHOW_*` flags and so has nothing to write.
2. **Badge walls.** 40+ badges in one block encode no ranking, so the reader takes none of them in; the community has argued this bluntly for years ("Profile badges are patronising, ugly and might be used as a signal of superiority", [GitHub community discussion #28686](https://github.com/orgs/community/discussions/28686)). A useful ceiling: 6–8 badges, ≤ 2 rows.
3. **Two animated headers, no content above the fold.** A 300 px waving banner plus a typing SVG is ~350 px of motion before the first sentence, and the typing text is invisible to search engines, screen readers and copy-paste.
4. **Certificates as the main content.** Four images (495 KB + 68 KB + 57 KB local PNGs plus a 57 KB remote WebP ≈ 693 KB) for one line of information. The big one is 4800 × 2960 px (read from the PNG IHDR header) rendered at 470 CSS px wide — ~5× linear oversampling even on a 2× display, and ~24× the pixels actually needed. On mobile all four are scaled to ~330 px, where the certificate text is unreadable; and the whole block sits at the *bottom* of the page, after six stats cards — the least-read position for the most bytes.
5. **Skillicons row with a fixed theme.** The row pins `theme=light`; the service documents that theme as controlling the icons' background ("Some icons have a dark and light themed background", [readme](https://github.com/tandpfun/skill-icons)), so light-mode icon tiles sit on the dark GitHub canvas. It is also the heaviest element measured: 54.8 KB / 6.4 s for 15 logos.
6. **Ambient-metric stacking.** Stats card + streak + activity graph + WakaTime languages + top-langs donut + view counter = six images answering one question the profile page already answers with its contribution graph. Each is a third-party request, a layout shift, and a theme mismatch risk (`theme=transparent` bakes a fixed text colour that must be legible on both `#ffffff` and a dark canvas).
7. **Public personal data in a badge.** `WeChat&Tel-13543750006` publishes a phone number, permanently, in an image-alt-less badge, on the most-scraped page a developer owns.
8. **Dead or decorative links.** `HuggingFace-jimchoi` is a bare `<img>` with no anchor; `GitHub-CAI991108` links to the page the reader is already on. A link that goes nowhere is worse than no link.
9. **Stale signal.** The README's content last changed 2025-04-27 while 827 bot commits landed since — the visible artefact and the activity graph tell different stories.

---

## Aesthetic system

**Three colours, chosen by role.** Pick one accent, one neutral, one semantic — the same roles GitHub itself uses (`accent` for links/selection, plus `success`/`attention`/`danger`/`open`/`done`, per [Primer colour usage](https://primer.style/product/getting-started/foundations/color-usage/)). For a research profile, an accent in the blue/violet family reads as academic and matches `scholar`/`arxiv` branding; a warm second colour should be reserved for exactly one thing (e.g. "open to internships"). Then use the accent's hex in every badge's `color=` / `labelColor=` so the badges look like one set.

**Height parity is the whole trick.** Measured SVG heights for the five shields styles: `flat` 20 px, `flat-square` 20 px, `social` 20 px, `plastic` 18 px, `for-the-badge` 28 px. The official WakaTime badge is exactly 20 px tall (measured: `height="20"`). So: keep every badge in a row at 20 px (`flat`, `flat-square` or `social` — never mix in `plastic` or `for-the-badge`), and size any hand-placed `<img>` (social icons, header) to 20/24/28 px multiples rather than arbitrary values like `height="40"`.

**Dark/light parity.** Cards and badges are static images with baked colours, so they cannot follow GitHub's theme. Two options: (a) `<picture>` with `prefers-color-scheme` for anything you control (header, diagrams, photos) — documented [here](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/); (b) choose badge colours with enough contrast on both `#ffffff` and a near-black canvas, and prefer `transparent` backgrounds only when the text colour is mid-tone (a dark navy label on transparent disappears in dark mode).

**Typography.** You get one text font (GitHub's UI stack) — so hierarchy must come from size (heading level), weight (`**bold**`) and spacing (blank lines, `<br>`), not from fonts. Inside SVGs you can have any font, which is exactly why mixing "designed" banners with plain text often looks worse than plain text.

**Consistency rules worth writing down:** one badge row per section, max 6; the same accent hex everywhere; `style=flat` for text chips and `flat-square` only if used consistently; logos only when the logo is a domain the reader recognises (see next point); no more than two colours of motion on the page (ideally none).

**Logo gotcha, verified against simple-icons:** `logo=` takes a simple-icons *slug*, and some slugs no longer exist. Checked by fetching the icon files directly: `googlescholar`, `orcid`, `semanticscholar`, `arxiv`, `dblp`, `huggingface`, `x`, `wechat`, `kaggle`, `researchgate`, `github`, `wakatime`, `pytorch`, `tensorflow`, `scikitlearn`, `r`, `latex`, and `vuedotjs` all exist; **`linkedin`, `matlab`, `windows` and `vue` do not** (`vue` is now `vuedotjs`). A badge with a missing logo either renders without it or errors — [shields docs](https://shields.io/docs/logos) also note that new simple-icons may not yet be available on shields. For academic links, prefer the two-segment chip (`/badge/Google%20Scholar-4285F4`) with an explicit `logo=googlescholar` only after checking it renders.

---

## Specific findings for CAI991108's current README

| # | Finding | Evidence | Verdict |
|---|---|---|---|
| 1 | 827 of 861 commits are daily empty "Updated with Dev Metrics" | `git log --name-only`: 0 files touched in the last 25 commits; `git rev-list --count` for the totals | Delete the workflow; keep WakaTime out of the commit history |
| 2 | `.github/workflows/waka_readme` is inert (no `.yml`) | [workflow syntax docs](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) | Delete; it is a decoy |
| 3 | The flex `<div>` does nothing and is never closed | [sanitizer allowlist](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb); [README.md:41](../README.md#L41) opens, only the inner div closes at [:47](../README.md#L47) | Replace with `<p align="center">` or a `<table>` |
| 4 | Certificates: 693 KB total, incl. a 4800 × 2960 PNG shown at 470 px | PNG header dimensions + file sizes; [About large files](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github) | Re-encode to ~100 KB WebP at 2× display width, or move behind `<details>`, or move to the personal site |
| 5 | Two animated widgets before any text | [README.md:1-9](../README.md#L1-L9) | Keep at most one, ≤ 120 px, or replace with a `<picture>` name card |
| 6 | Six overlapping stats artefacts | lines 13–23 | Keep one, ideally Action-generated (no per-view third-party call) |
| 7 | WakaTime card is queried with the **GitHub** handle | line 21 vs the card's docs ("Change the `?username=` value to your WakaTime username"); the account also has a UUID badge (`0a1a641a-…`) | Verify in a browser; if the WakaTime username differs, the card is showing an error or someone else's data |
| 8 | WakaTime card requires two public toggles | [github-readme-stats readme](https://github.com/anuraghazra/github-readme-stats) | Confirm both are on, or drop the card |
| 9 | skillicons `theme=light` on a dark canvas, 54.8 KB | [skill-icons readme](https://github.com/tandpfun/skill-icons); measured | Replace with a text row or 6–8 flat badges |
| 10 | Phone number published in a badge | line 35 | Remove immediately; use a mailto or the site's contact form |
| 11 | `HuggingFace-jimchoi` is unlinked; GitHub badge self-links | lines 32–34 | Wrap every badge in an `<a href>` to the real destination |
| 12 | Content stale since 2025-04-27 | `git log -- README.md` | Rewrite; then let a single Action update only real data |

---

## Recommended shortlist

**Hero (cheapest high-impact change).** One `<h1 align="center">` or a `<picture>`-based name card, then one line: *"PhD student in AI, CUHK-Shenzhen — [research area]. [site] · [Scholar] · [email]"*. No 300 px banner, no typing SVG. Everything a reader needs is then within the first 120 px.

**Section skeleton (six headings → clean Outline):** `## Now` · `## Research` · `## Selected work` · `## Publications` · `## Toolbox` · `## Contact`.

**Components to keep, in order of value per byte:**

| Keep | Why | How |
|---|---|---|
| Plain-text link row (site, Scholar, ORCID, Semantic Scholar, HF, GitHub, email) | Highest information per pixel; works on mobile; accessible | Markdown links, `<br>`-separated, or 2-segment shields chips |
| `## Now` block (3 bullets: current project, advisor/lab, what you want next) | Answers the only question a reader has | Text |
| Selected work table (repo/paper, one-line "why", language) | Proof-of-work, scannable, no third party | `<table>` with 2–3 columns |
| Publications list (3–5, full citations + links) | The academic equivalent of stars | Text + links; `arXiv`/`DBLP`/`Scholar` links |
| One stats artefact, generated by an Action | Fresh without bot commits and without per-view API calls | `readme-tools/github-readme-stats-action`, or self-hosted Vercel |
| `<details>` block holding certificates/awards | Zero cost until expanded; keeps the page short | `<details><summary>Awards & certificates</summary>…</details>` |
| `<picture>` name/banner with light+dark variants | Only documented way to be theme-correct | [GitHub Blog snippet](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/) |

**Drop:** capsule-render header, typing SVG, streak card, activity graph, top-langs donut, WakaTime language card (keep at most the official 20 px WakaTime badge if the total-hours number is meaningful), skillicons row, komarev counter, the `WeChat&Tel` badge, and the bot workflow. Net effect: from ~693 KB of images and 6 third-party renderers down to ≤ 2 images, and from 350 px of animation to 0.

**If a dashboard is wanted anyway,** the honest version is: self-host the stats card (as [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) does), put it in one row with the official WakaTime badge at matching 20 px height, keep both at `style=flat`, and hide everything else in `<details>`.

---

## Sources

**GitHub, first-party**
- [Managing your profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme) — prerequisites, creation flow, template.
- [Basic writing and formatting syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) — Outline menu from headings; relative links/paths and `/` resolution; images and alt text; uploading assets.
- [github/markup README](https://github.com/github/markup) — "inline-styles, and `class` or `id` attributes" are removed during sanitization.
- [`html-pipeline` `SanitizationFilter`](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb) — the element and attribute allowlists quoted above; `WHITELIST[:attributes]` and `:all`.
- [GitHub Blog: images that adapt to dark/light mode](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/) — the `<picture>` + `media="(prefers-color-scheme: …)"` snippet for READMEs.
- [About anonymized URLs (Camo)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-anonymized-urls) — image proxying and `githubusercontent.com` URLs.
- [About large files](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github) — 25 MiB browser upload, 50 MiB warning, 100 MiB block.
- [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) — `.yml`/`.yaml` requirement and `.github/workflows` location.
- [Primer layout](https://primer.style/product/getting-started/foundations/layout/) — breakpoints 320/544/768/1012/1280/1400 and content padding.
- [Primer colour usage](https://primer.style/product/getting-started/foundations/color-usage/) — accent/success/attention/danger/open/done roles.

**Badges**
- [shields.io static badge](https://shields.io/badges/static-badge) — URL grammar, `_`/`__`/`--` escaping, `style` values, `logo`, `logoColor`, `logoSize`, `label`, `labelColor`, `cacheSeconds`.
- [shields.io logos](https://shields.io/docs/logos) — simple-icons slugs, base64 custom logos, `logoColor`.
- [shields.io dynamic badges](https://shields.io/badges/dynamic-json-badge) — the JSON endpoint family.
- [simple-icons `slugs.md`](https://github.com/simple-icons/simple-icons/blob/master/slugs.md) — slug source of truth; per-slug existence checked via `raw.githubusercontent.com/simple-icons/simple-icons/master/icons/<slug>.svg`.

**Stats / WakaTime**
- [anuraghazra/github-readme-stats](https://github.com/anuraghazra/github-readme-stats) — public-instance reliability warning, self-hosting and Actions guidance, `cache_seconds` bounds (21600–86400), per-card default caches, WakaTime public-profile requirement, `username` is the WakaTime username.
- [WakaTime API docs](https://wakatime.com/developers) — API keys are write-only; OAuth tokens for read access; "Do NOT use the API with your secret key on a public website"; ~10 req/s guidance.
- [anmol098/waka-readme-stats](https://github.com/anmol098/waka-readme-stats) — `WAKATIME_API_KEY` + `GH_TOKEN`, cron-driven commits, `SHOW_*` flags, PAT `user` scope caveat.
- [athul/waka-readme](https://github.com/athul/waka-readme) — marker-comment model; OAuth token vs API key; `GH_TOKEN` only when not the profile repo.
- Live probe: `https://wakatime.com/badge/user/<uuid>.svg` → 200, `image/svg+xml`, `max-age=3600`, 190.4 × 20.

**Profiles referenced**
[rasbt](https://github.com/rasbt/rasbt) · [simonw](https://github.com/simonw/simonw) · [rusty1s](https://github.com/rusty1s/rusty1s) · [xinychen](https://github.com/xinychen/xinychen) · [NielsRogge](https://github.com/NielsRogge/NielsRogge) · [Vaibhavs10](https://github.com/Vaibhavs10/Vaibhavs10) · [DenverCoder1](https://github.com/DenverCoder1/DenverCoder1) · [thmsgbrt](https://github.com/thmsgbrt/thmsgbrt) · [anuraghazra](https://github.com/anuraghazra/anuraghazra) · [blackcater](https://github.com/blackcater/blackcater)

**Community / secondary**
- [GitHub community discussion #28686, "Profile badges are patronising, ugly…"](https://github.com/orgs/community/discussions/28686) — sentiment on badge walls (community thread, not GitHub policy).
- [NN/g, F-shaped pattern](https://www.nngroup.com/articles/f-shaped-pattern-reading-web-content/) — scanning behaviour behind the "five-second test" (research source, not GitHub-specific).

**Academic link targets (for the badges/links to point at)**
- [arXiv API](https://info.arxiv.org/help/api/index.html) · [Semantic Scholar API](https://api.semanticscholar.org/api-docs/) · [dblp search API](https://dblp.org/faq/How+to+use+the+dblp+search+API.html) — all three have official APIs, so a live citation/publication count is possible via a shields dynamic-JSON badge. Google Scholar publishes no API and no official badge endpoint; ORCID publishes brand assets but I could not load `info.orcid.org/brand-guidelines` from this network (403), so treat "official ORCID badge" as unverified — the reliable route is a shields chip with `logo=orcid`.

### Unverified / caveated

- **`*.vercel.app` is unreachable from this machine** (it resolves to sinkhole IPs), so I could not see live output for `github-readme-stats`, `capsule-render` or `github-readme-activity-graph`. Their URL grammar, caches and requirements come from their own docs/READMEs, which I did read; only the rendered result for this account is unverified.
- **Whether the WakaTime card currently shows data for `username=CAI991108`** — unverified for the same reason, and because `wakatime.com/@CAI991108` and `wakatime.com/@0a1a641a-…` both return HTTP 200 (so 200 does not prove the handle exists).
- **`#gh-dark-mode-only`** — I found no first-party documentation; only third-party usage (e.g. skill-icons' README). Use `<picture>` instead.
- **Google Scholar "no API, no badge"** is an absence-of-evidence claim: I found no Google-published API or badge endpoint, but I have no Google page stating the negative.
- **ORCID's official badge** — `info.orcid.org/brand-guidelines` returned 403 from this network, so the existence/URL of an official ORCID badge is unverified.
- **The exact production sanitizer allowlist** — GitHub does not publish it; `github/html-pipeline` is the closest public artefact and demonstrably lags production (`span` and `picture` work live but are not in its `:elements`). Everything in the table above is either in that list or verified on a live page; anything else should be tested before shipping.
- **README column width** — the 700–1000 px desktop figure is an observation from the layout tokens, not a documented number.
- **Timings** (skillicons 6.4 s, streak 1.9 s, etc.) are single probes from this machine and include local network variance; byte counts are exact.

*Researched on 2026-10-24.*
