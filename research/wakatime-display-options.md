# WakaTime display options — what @CAI991108 can actually put on GitHub

The profile currently shows exactly one WakaTime image: the all-time badge. This note answers what else
exists, which URL patterns are real, and what the "public stats range" setting blocks. Every line marked
**probe** was fetched from this machine on **2026-10-25** with Node's `fetch` (this box cannot do
`Invoke-WebRequest`/`curl.exe`); GitHub rendering rules are inherited from
[github-profile-readme.md](./github-profile-readme.md) and
[engineering-profile-components.md](./engineering-profile-components.md) and are not re-derived here.

---

## API endpoints that matter

The docs advertise the prefix `https://api.wakatime.com/api/v1/`; `https://wakatime.com/api/v1/` serves the
same API and is what every probe below used ([WakaTime API docs](https://wakatime.com/developers)).

| Endpoint | Auth | Works for a public profile, no token? | What comes back |
|---|---|---|---|
| `GET /users/{user}` | none | **Yes — 200** (probe) | Identity + the five privacy flags + `public_profile_time_range`. CAI991108 returns `logged_time_public: true`, `languages_used_public: true`, `editors_used_public: true`, `categories_used_public: true`, `os_used_public: true`, `is_hireable: true` ([docs § Users](https://wakatime.com/developers#users)) |
| `GET /users/{user}/stats/{range}` | none for the public range | **Only for the one public range.** `all_time` → 200; `last_7_days`, `last_30_days`, `last_6_months`, `last_year`, `2025` → 400 `Time range not matching user's public stats range.` (probe) | `total_seconds`, `total_seconds_including_other_language`, `human_readable_total`, `daily_average`, `languages[]`, `editors[]`, `operating_systems[]`, `categories[]`, `ai_models`, `range`, `timeout`, `writes_only`, `is_up_to_date`, `percent_calculated`, `is_*_visible` ([docs § Stats](https://wakatime.com/developers#stats)) |
| `GET /users/{user}/stats` (no range) | none | **Yes — 200** (probe) | Same payload, range = the profile's public range. Docs: "When range isn't present, the user's public profile range is used" ([§ Stats](https://wakatime.com/developers#stats)) |
| `GET /users/{user}/summaries?start=&end=` | token, `read_summaries` | **No — 401** (probe) | Array of per-day summaries with the same language/editor/category breakdown ([§ Summaries](https://wakatime.com/developers#summaries)) |
| `GET /users/{user}/status_bar/today` | token, `read_summaries` | **No — 401** (probe) | Today's `grand_total` from cache, for IDE status bars ([§ Status Bar](https://wakatime.com/developers#status_bar)) |
| `GET /users/{user}/all_time_since_today` | token, `read_stats` | **No — 401** (probe), even though `stats/all_time` is public | `text`, `decimal`, `digital`, `total_seconds`, `range` ([§ All Time Since Today](https://wakatime.com/developers#all_time_since_today)) |
| `GET /users/current` (+ `/stats/{range}`) | token | **No — 401** (probe) | Your own account, **any** range, no public-range restriction ([§ Users](https://wakatime.com/developers#users)) |
| `GET /users/{user}/insights/{type}/{range}` | token, `read_summaries` | **No — 401** (probe) | `stats`, `weekdays`, `days`, `languages`, `editors`, `categories`, `machines`, `operating_systems` insights ([§ Insights](https://wakatime.com/developers#insights)) |
| `GET /users/{user}/goals`, `/projects` | token | **No — 401** (probe) | Goals / project list ([§ Goals](https://wakatime.com/developers#goals), [§ Projects](https://wakatime.com/developers#projects)) |
| `GET /leaders` | none | **Yes — 200** (probe) | Public leaderboard rows with `running_total` ([§ Leaders](https://wakatime.com/developers#leaders)) |
| `GET /stats/{range}` | none | **Yes — 200** (probe) | Site-wide aggregate for `last_7_days` or a past year ([§ Stats Aggregated](https://wakatime.com/developers#stats_aggregated)) |

Two divergences worth knowing: the docs attach a scope (`read_stats`, `read_summaries`) to endpoints that
nevertheless answer anonymously for a public range, and `all_time_since_today` — the endpoint that sounds
most public — is **not** one of them.

What the `stats` payload actually contained for CAI991108 (`all_time`, probe): `total_seconds` **1,809,380**
= 502 hrs 36 mins *excluding* the `Other` language, `total_seconds_including_other_language` = **1,963 hrs 38
mins**, `days_minus_holidays` 681, plus `languages` where `Other` alone is 1,461 hrs. The badge on the README
shows the **including-Other** number (1,963 hrs 38 mins), so badge and API appear to disagree by ~4× until you
notice which field you are reading.

---

## Public range setting

The 400 is not a bug: WakaTime stores **one** public time range per account and refuses every other range to
anonymous callers.

- **Field name:** `public_profile_time_range`. It is returned by `GET /users/{user}` (probe: `"all_time"` for
  CAI991108) — but it is **undocumented**: the Users example response in the API docs does not list it
  ([§ Users](https://wakatime.com/developers#users)).
- **Where the control lives:** <https://wakatime.com/settings/profile> (login required). The setting group is
  labelled **"Display code time publicly"**; the range selector sits in that group. The same page carries the
  visibility checkboxes that `github-readme-stats` and `waka-box` both tell users to enable
  ("Display languages, editors, os, categories publicly") — see the
  [github-readme-stats WakaTime card docs](https://github.com/anuraghazra/github-readme-stats#wakatime-stats-card)
  and the [waka-box setup steps](https://github.com/matchai/waka-box).
- **What "one range" means, measured.** Three public profiles, five ranges each (probe):

| Profile | `public_profile_time_range` | `all_time` | `last_7_days` | `last_30_days` | `last_6_months` | `last_year` |
|---|---|---|---|---|---|---|
| CAI991108 | `all_time` | **200** | 400 | 400 | 400 | 400 |
| Hakula | `last_7_days` | 400 | **202** | 400 | 400 | 400 |
| ffflabs | `last_year` | 400 | 400 | 400 | 400 | **200** |

  (`202` = "Accepted", the code the docs say `stats` returns while a range is being calculated. Either way,
  exactly one range answers; every other range is the same 400.)

- **What to change to expose 7 or 30 days:** set that selector to `Last 7 Days` (or `Last 30 Days`). That is
  the documented-enough behaviour confirmed by the card maintainers in
  [github-readme-stats#2698](https://github.com/anuraghazra/github-readme-stats/issues/2698): the setting is
  the thing at `/settings/profile`, and the current value can be read from `public_profile_time_range`.
  The trade is symmetric — switching to 7 days makes `stats/all_time` return 400 to anonymous callers.
- **The badge survives the switch.** Hakula's profile is `last_7_days`, yet
  `https://wakatime.com/badge/user/f4a35a1f-…svg` still returned **200** with the all-time total
  (7,794 hrs 23 mins). The relevant field is `share_all_time_badge: true`, which CAI991108 also has (probe).
- **A token bypasses the restriction entirely.** `athul/waka-readme` exposes `TIME_RANGE` with all five values
  and documents that the secret must be an **OAuth token**, not the plugin API key
  ([README](https://github.com/athul/waka-readme)) — i.e. an authenticated job reads any range regardless of
  what the public profile exposes.

---

## Official embeds

The share hub (<https://wakatime.com/share>) enumerates the first-party offers: **Repo Badges**, **Project
Badges**, **Embeddable Charts**, Goals, Insights, Animated GIFs, Email Reports, JSON API. Only three of those
are image URLs you can drop into a README — user badge, repo badge, project badge, embeddable chart — and one
of them has no public instance for this account yet.

| Embed | URL pattern | Live check (2026-10-25) |
|---|---|---|
| User badge (SVG) | `https://wakatime.com/badge/user/<user-uuid>.svg` | **200** · `image/svg+xml` · **190.4 × 20** · 3,114 B · `cache-control: max-age=3600` · text "1,963 hrs 38 mins" |
| User badge (PNG) | same, `.png` | **200** · `image/png` · **190 × 20** · 3,526 B |
| Repo badge | `https://wakatime.com/badge/github/<owner>/<repo>.svg` | **200** for `wakatime/wakatime-cli` (179.9 × 20, 3,131 B, "588 hrs 30 mins"); **404** with an SVG body ("not found", 3,096 B) for a repo without linked WakaTime data |
| Project badge | `https://wakatime.com/badge/user/<user-uuid>/project/<project>.svg` | Route exists; **404** SVG "not found" for a project that is not public (3,096 B) |
| Embeddable chart (SVG) | `https://wakatime.com/@<user>/<chart-id>.svg` → **302** → `https://wakatime.com/share/embeddable/<user>/<chart-id>.svg` | **202 → 200** · `image/svg+xml` · **800 × 600** · 15,058 B on the 202, 28,781 B once cached |
| Embeddable chart (PNG) | same, `.png` | **200** · `image/png` · **800 × 600** · 22,445 B |
| Embeddable chart (JSON) | same, `.json` | **404** for the chart id tried — JSON shares are separate share objects minted in the same UI |
| Profile social image | `https://wakatime.com/photo/<user-uuid>` (the profile's `og:image`) | **200** · `image/png` · **420 × 420** · 145,593 B — it is the avatar, *not* a stats card |
| Share index | `https://wakatime.com/share/@CAI991108` | **404** — this account has no public share list |
| Share / embed UI | `https://wakatime.com/share/embed` | **302 → /login** — a chart id can only be minted while signed in; probe user `robertwe`'s id was used for the rows above |

Details that decide usability:

- **The embed is big and white.** 800 × 600 is 2.4× a phone's ~330 px text column, and `?theme=dark` and
  `?theme=light` returned **byte-identical** SVG with `background-color:#ffffff` hard-coded (probe). There is
  no dark variant and no `prefers-color-scheme` support on the chart endpoint.
- **The badge is 20 px and light.** `rx=3`, label `#555`, value `#007ec6`, background `#fff` (read out of the
  SVG), so it reads as a normal pill on both GitHub themes. `?style=flat` returns a slightly different
  191 × 20 render (3,469 B) — still 20 px, so it mixes with shields `flat`/`flat-square` at the same height.
- **The badge counts `Other`; the API total does not.** See the field comparison above.

---

## Third-party renderers

| Renderer | What it needs | What it renders | Update cadence | Failure modes |
|---|---|---|---|---|
| [anmol098/waka-readme-stats](https://github.com/anmol098/waka-readme-stats) | `WAKATIME_API_KEY` + `GH_TOKEN` (PAT with `repo` + `user` for the commit sections) | **Text/emoji block** written between `<!--START_SECTION:waka-->` markers: weekly total, languages, editors, OS, projects, AI-coding split | Cron; the README's example is `30 18 * * *` (daily) | It commits to your repo every run — this repo already carries 827 "Updated with Dev Metrics" commits; its README still says "WakaTime API Key", but WakaTime keys are **write-only** ([docs § Authentication](https://wakatime.com/developers#authentication)), so an OAuth token is required in practice |
| [athul/waka-readme](https://github.com/athul/waka-readme) | `WAKATIME_API_KEY` = **OAuth token** (stated explicitly), workflow `contents: write`; `GH_TOKEN` only outside the profile repo | **ASCII bar chart in a fenced code block** between markers; `BLOCKS` picks the glyphs (`░▒▓█`, `⣀⣄…`), `TIME_RANGE` any of the five ranges, `LANG_COUNT`, `SHOW_TIME` | Cron; README example `0 0 * * *` (daily) | Needs Actions write permission or the push silently fails; output is plain text, so it inherits GitHub's font and cannot be themed; every run is a commit |
| [github-readme-stats `api/wakatime`](https://github.com/anuraghazra/github-readme-stats#wakatime-stats-card) | Public WakaTime profile with **both** "Display code time publicly" and "Display languages, editors, os, categories publicly" enabled; nothing else | `<img>`-able **card**, default width **495 px** (`DEFAULT_CARD_WIDTH` in [`src/cards/wakatime.js`](https://github.com/anuraghazra/github-readme-stats/blob/master/src/cards/wakatime.js)); `layout=compact`, `hide_progress`, `langs_count`, `display_format=percent` | 24 h server cache ([README](https://github.com/anuraghazra/github-readme-stats)) | Upstream now opens with **"This repository is no longer maintained"** (successor: `stats-organization/github-stats-extended`); the fetcher calls `/users/{u}/stats?is_including_today=true` with no `range`, so the card silently shows whatever range the profile exposes, and *any* non-2xx is rewritten as `Could not resolve to a User with the login of '<u>'` ([`src/fetchers/wakatime.js`](https://github.com/anuraghazra/github-readme-stats/blob/master/src/fetchers/wakatime.js)); `github-readme-stats.vercel.app` is unreachable from this machine (fetch failed) |
| [matchai/waka-box](https://github.com/matchai/waka-box) | GitHub token with **`gist`** scope, WakaTime credential, and the two public toggles at `/settings/profile` | Text inside a **pinned gist** — not in the README; the README can only link to it | Weekly stats in a gist, driven by cron | Repo is **archived** (GitHub API: `archived: true`, last push 2024-07-27) — treat as dead; a gist is not renderable inline, so its value on a profile is one link |
| [lowlighter/metrics — WakaTime plugin](https://github.com/lowlighter/metrics/blob/main/source/plugins/wakatime/README.md) | Action run + `plugin_wakatime_token`; sections `time`, `projects`, `languages`, `editors`, `os` (+ `-graphs` variants); `plugin_wakatime_days` ∈ `7/30/180/365` | **SVG committed to the repo** (example output: **480 × 231**, 12,642 B) | Cron Action | Token required; the plugin is explicitly "not affiliated" with WakaTime; the shared instance (`metrics.lecoq.io`) returned **HTTP 500** today (probe), so only the Action path is trustworthy |
| [mamboer/waka-charts](https://github.com/mamboer/waka-charts) | GitHub Action + `WAKATIME_API_KEY` | **SVG committed to the repo**, light **and** dark variants | Cron; README example `0 0 * * *` | Same class as metrics but smaller; assets must be referenced repo-relative or via `raw.githubusercontent.com`, and the action is a one-maintainer project |

The pattern across the table: **anything token-based can read any range; anything anonymous is pinned to the
one public range.** That is the single fact that decides the architecture below.

---

## What renders in a README

GitHub accepts images and text, nothing else. `style=` is stripped rather than ignored, `<iframe>`, `<video>`,
`<script>` are removed, while `<img>`, `<picture>`, `<table>`, `<details>`, `<kbd>`, `<sub>`/`<sup>` survive
([github/markup](https://github.com/github/markup), [`html-pipeline`
`SanitizationFilter`](https://github.com/github/html-pipeline/blob/master/lib/html/pipeline/sanitization_filter.rb),
and the render-level checks in [github-profile-readme.md](./github-profile-readme.md)). External images are
proxied and cached by Camo ([GitHub docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-anonymized-urls)),
which is why an external card can be slow or stale but a committed SVG cannot.

| Artefact | Renders as | Pixels | Bytes | Theme behaviour |
|---|---|---|---|---|
| WakaTime user badge | external `<img>` | 190 × 20 | ~3.1 KB | light pill, legible on both themes; fixed |
| WakaTime embeddable chart | external `<img>` | 800 × 600 | 15–29 KB | white background, no dark variant |
| metrics / waka-charts SVG | repo-relative `<img>` (or `<picture>` for the light/dark pair) | 480 × 231 (metrics example) | 12.6 KB measured | chosen at build time, so it matches the page |
| github-readme-stats WakaTime card | external `<img>` | 495 px default width | not measurable here (Vercel unreachable) | `theme=` parameter |
| `waka-readme` / `waka-readme-stats` output | Markdown text / fenced block | — | < 1 KB | inherits GitHub's one text font and both themes |
| `waka-box` gist | a link, not an image | — | — | n/a |

Budgets that apply on top of that (from the repo's earlier audits): **≤ 150 KB per image, ≤ 400 KB total**,
and a phone gives a README **330–370 px** of usable width — so a 480 px SVG is already ~1.4× too wide and an
800 px chart is 2.4× too wide, scaled down to unreadable text. A 20 px badge is the only element in this
document that is natively phone-sized.

---

## Recommended setup for @CAI991108

Given: public range is `all_time` only, a weekly stats workflow exists but is parked, the page is going
engineering/geek, and the image budget is tight.

1. **Keep the public range at `all_time`.** It keeps the anonymous stats API on the same number the badge
   shows, so any card built on it (the `github-readme-stats` languages card, for instance) reports all-time
   languages rather than a week. The badge itself survives either choice — see Hakula above — so this is a
   preference about what anonymous clients may read, not about whether the page keeps working.
   Changing it to `Last 7 Days` buys anonymous 7-day data and costs anonymous all-time data — a straight
   swap, not an upgrade.
2. **Keep the official badge** (`https://wakatime.com/badge/user/0a1a641a-17e3-4bcf-8339-f02a8fb3a069.svg`):
   20 px, 3.1 KB, zero maintenance, theme-safe, and it already renders the including-Other total. Nothing else
   in this document beats it on bytes-per-credibility.
3. **Enable the parked weekly workflow with an OAuth token, not the API key.** Scope `read_stats` is enough;
   a token reads any range, so the job can compute "last 7 days" while the public profile stays `all_time`.
   Have it commit **one** SVG — `lowlighter/metrics` with `plugin_wakatime: yes`,
   `plugin_wakatime_days: 7`, `plugin_wakatime_sections: time, languages` produces a 480 × 231, ~12.6 KB file
   — and reference it repo-relative, so no viewer ever hits a third party.
4. **Put it where it does not dominate.** Either a two-cell `<table>` (card left, three bullets right) or
   inside `<details><summary>coding time</summary>`. At 480 px it is wider than a phone; behind `<details>` it
   costs nothing until opened.
5. **Do not put the 800 × 600 embeddable chart in the README.** It is a hard white box on a dark page, 2.4×
   too wide, and needs a manual chart id minted at `/share/embed`. Link to it instead if you want the full
   chart.
6. **Do not enable `waka-readme-stats` on a daily cron.** It writes text, which is fine, but every run is a
   commit — the exact pattern that produced 827 empty "Updated with Dev Metrics" commits here. If text is
   wanted, run `athul/waka-readme` weekly with `TIME_RANGE: last_7_days` and a `BLOCKS` glyph set that matches
   the page's monospace look.

Sketch of the whole WakaTime surface, after the above:

```markdown
<p align="center">
  <a href="https://wakatime.com/@CAI991108"><img alt="Total coding time on WakaTime"
    src="https://wakatime.com/badge/user/0a1a641a-17e3-4bcf-8339-f02a8fb3a069.svg"></a>
</p>

<details><summary>Last 7 days (updated weekly)</summary>

<img alt="Coding time by language, last 7 days" src="./assets/wakatime-week.svg" width="480">

</details>
```

Cost: ~3.1 KB external + ~12.6 KB committed, both inside the 150 KB per-image budget, and one scheduled
commit per week instead of 365.

---

## Sources

**WakaTime, first-party**
- [API docs](https://wakatime.com/developers) — base URL, status codes, authentication ("API keys are
  **write-only**", OAuth tokens for reads), scopes, rate guidance.
- [§ Stats](https://wakatime.com/developers#stats) — `:range` vocabulary, "when range isn't present, the
  user's public profile range is used", full response schema, 202 semantics.
- [§ Summaries](https://wakatime.com/developers#summaries) · [§ Status
  Bar](https://wakatime.com/developers#status_bar) · [§ All Time Since
  Today](https://wakatime.com/developers#all_time_since_today) · [§ Insights](https://wakatime.com/developers#insights)
  · [§ Goals](https://wakatime.com/developers#goals) · [§ Projects](https://wakatime.com/developers#projects)
  — required scopes and response fields.
- [§ Users](https://wakatime.com/developers#users) — identity and the `*_public` flags (note:
  `public_profile_time_range` is **not** in the documented example).
- [§ Leaders](https://wakatime.com/developers#leaders) · [§ Stats
  Aggregated](https://wakatime.com/developers#stats_aggregated) — the two anonymous endpoints that return 200.
- [Share hub](https://wakatime.com/share) — the official list of share types (repo badges, project badges,
  embeddable charts, goals, insights, animated GIFs, email reports, JSON API).
- [FAQ](https://wakatime.com/faq) — public profile timeout/formula, "edit your profile and check the boxes",
  "Settings > Profile".
- [Settings → Profile](https://wakatime.com/settings/profile) — the "Display code time publicly" group and its
  range selector (login-gated; confirmed by the two third-party READMEs below).

**Third-party, read at source**
- [anuraghazra/github-readme-stats](https://github.com/anuraghazra/github-readme-stats) — unmaintained
  CAUTION, WakaTime card requirements/options, 24 h cache, [`src/cards/wakatime.js`](https://github.com/anuraghazra/github-readme-stats/blob/master/src/cards/wakatime.js)
  (495 px default) and [`src/fetchers/wakatime.js`](https://github.com/anuraghazra/github-readme-stats/blob/master/src/fetchers/wakatime.js)
  (no `range`, generic error message).
- [Issue #2698](https://github.com/anuraghazra/github-readme-stats/issues/2698) — maintainer confirmation that
  `/settings/profile` decides the served range and that `public_profile_time_range` reveals it.
- [anmol098/waka-readme-stats](https://github.com/anmol098/waka-readme-stats) — secrets, markers, `SHOW_*`
  flags, cron example, "API Key" wording.
- [athul/waka-readme](https://github.com/athul/waka-readme) — OAuth-token requirement, markers, `TIME_RANGE`,
  `BLOCKS`, cron example.
- [matchai/waka-box](https://github.com/matchai/waka-box) — gist model, `gist` scope, the two public toggles;
  GitHub API confirms `archived: true`, last push 2024-07-27.
- [lowlighter/metrics WakaTime plugin](https://github.com/lowlighter/metrics/blob/main/source/plugins/wakatime/README.md)
  — options and allowed values; example SVG measured at 480 × 231 / 12,642 B.
- [mamboer/waka-charts](https://github.com/mamboer/waka-charts) — committed-SVG Action, light/dark variants.

**Probes (this machine, 2026-10-25)** — every status, content-type, byte count and pixel dimension above.
Endpoints: `users/CAI991108`, `users/{CAI991108,Hakula,ffflabs}/{stats/{all_time,last_7_days,last_30_days,last_6_months,last_year},stats,summaries,status_bar/today,all_time_since_today,insights/...,goals,projects}`,
`users/current`, `leaders`, `stats/last_7_days`; images: `badge/user/<uuid>.svg|.png`, `badge/github/wakatime/wakatime-cli.svg`,
`badge/user/<uuid>/project/<name>.svg`, `@robertwe/<chart>.svg` → `share/embeddable/...svg|.png|.json`, `photo/<uuid>`.

**Unverified / caveated**
- `github-readme-stats.vercel.app` is unreachable from here (connection failure), so no card dimensions or
  live render for this account; `metrics.lecoq.io` answers 500.
- No embeddable chart exists for CAI991108 (`/share/@CAI991108` → 404; the logged-in UI is needed to mint
  one), so the chart row above is verified through another user's still-live chart id, not this account's.
- The exact dropdown labels in "Display code time publicly" (e.g. "Last 7 Days" vs "last_7_days") are not
  published by WakaTime; only the stored value vocabulary (`all_time`, `last_7_days`, …) is confirmed.
- JSON and animated-GIF shares were listed by the share hub but could not be minted or fetched without login.
- Whether `public_profile_time_range` is writable through any public API is unknown; it appears to be a
  settings-page-only control.

*Researched and probed on 2026-10-25.*
