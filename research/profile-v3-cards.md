# Profile v3 — making the third-party cards lighter

Scope: the seven images the profile embeds from `github-readme-stats` (stats, top-langs, 4 × pin) and `github-readme-streak-stats` (streak), asked about in the v3 "cards are too heavy" thread. Everything below is read from the projects' own READMEs and source at `master`/`main` on 2026-10-25, plus WCAG arithmetic run locally in Node.

**This machine sinkholes `*.vercel.app`**, so `github-readme-stats.vercel.app`, `github-stats-extended.vercel.app` and `streak-stats.demolab.com` were not fetched and no card was rendered. Every behavioural claim is derived from the code path that produces the SVG (quoted below) and is marked *unrendered* where a browser could still surprise us. GitHub-side rendering (Camo re-hosting, theme tags) is quoted from the projects' docs and GitHub's changelog.

## 1. Transparent / borderless parameters

### 1.1 `anuraghazra/github-readme-stats` — 62 themes

Documented common options ([`readme.md` L292–L303](https://github.com/anuraghazra/github-readme-stats/blob/master/readme.md#L292-L303)):

| Param | Documented as | Default | What the code does |
|---|---|---|---|
| `bg_color` | "Card's background color" — hex or gradient | `fffefe` | goes into the background `<rect fill>` verbatim |
| `hide_border` | "Hides the card's border" | `false` | keeps the rect but sets `stroke-opacity="0"` |
| `border_radius` | "Corner rounding on the card" | `4.5` | into `rx` on the same rect, unvalidated |
| `border_color` | "Does not apply when `hide_border` is enabled" | `e4e2e2` | `stroke` on the rect |
| `theme` | name from `themes/README.md` | `default` | table lookup, per-colour params override it |

**Yes, `00000000` is documented as transparent**, in its own section ([`readme.md` L219–L224](https://github.com/anuraghazra/github-readme-stats/blob/master/readme.md#L219-L224)):

> You can use the `bg_color` parameter to make any of the available themes transparent. This is done by setting the `bg_color` to a color with a transparent alpha channel (i.e. `bg_color=00000000`)

There is also a built-in `theme=transparent` ([L204–L207](https://github.com/anuraghazra/github-readme-stats/blob/master/readme.md#L204-L207)), "a `transparent` theme that has a transparent background … optimized to look good on GitHub's dark and light default themes". Source grounding:

- **8-digit hex is accepted and normalised.** `isValidHexColor` matches `8|6|3|4` digits and `fallbackColor` prepends `#` ([`src/common/color.js` L11–L15, L37–L49](https://github.com/anuraghazra/github-readme-stats/blob/master/src/common/color.js#L11-L49)) — so `bg_color=00000000` reaches the renderer as `#00000000`, and `bg_color=0000` works too.
- **The card draws exactly one background shape.** `<rect data-testid="card-bg" rx="{border_radius}" stroke="{borderColor}" fill="{bgColor}" stroke-opacity="{hide_border ? 0 : 1}">` ([`src/common/Card.js` L243–L257](https://github.com/anuraghazra/github-readme-stats/blob/master/src/common/Card.js#L243-L257)). No second panel, no header bar — so "transparent" removes 100 % of the card's own fill.
- **`hide_border` is opacity, not removal.** The stroke attribute is still emitted; `hideBorder` is set from the query param at [`src/cards/stats.js` L521](https://github.com/anuraghazra/github-readme-stats/blob/master/src/cards/stats.js#L521). Consequence for anyone inspecting the SVG: a `e4e2e2` stroke is present but invisible. It costs nothing and has no side effect.
- **A theme without `border_color` inherits `e4e2e2`.** `defaultBorderColor = "border_color" in selectedTheme ? selectedTheme.border_color : defaultTheme.border_color` ([`color.js` L91–L95](https://github.com/anuraghazra/github-readme-stats/blob/master/src/common/color.js#L91-L95)). `transparent` ([`themes/index.js` L15–L20](https://github.com/anuraghazra/github-readme-stats/blob/master/themes/index.js#L15-L20)) and both Catppuccin themes have no `border_color`, so **`theme=transparent` on its own still draws a light-grey `#e4e2e2` border** — on GitHub dark that border is 14.67:1 against `#0d1117`, i.e. the loudest thing on the card. Always pair transparency with `hide_border=true`.
- **The built-in `transparent` theme is not neutral.** It ships blue text: title `006AFF`, icon `0579C3`, text `417E87`, bg `ffffff00`. Measured on `#0d1117`: 4.06 / 4.08 / 4.11 — all below the 4.5:1 AA threshold for normal text. It is "optimised" by eye, not by contrast.

**Catppuccin exists, under underscores** ([`themes/index.js` L398–L409](https://github.com/anuraghazra/github-readme-stats/blob/master/themes/index.js#L398-L409)):

| Theme | title | icon | text | bg |
|---|---|---|---|---|
| `catppuccin_mocha` | `94e2d5` | `cba6f7` | `cdd6f4` | `1e1e2e` |
| `catppuccin_latte` | `137980` | `8839ef` | `4c4f69` | `eff1f5` |

Note the title is teal `94e2d5`, **not** the `89b4fa` blue the README currently uses by hand. There is no `catppuccin_frappe`/`macchiato` here. The theme list is closed: "we paused the addition of new themes".

### 1.2 `stats-organization/github-stats-extended` — the successor, 66 themes

Drop-in compatible: same parameter names, same defaults, same section "GitHub-Stats-Extended aims to be fully compatible with github-readme-stats". The colour pipeline is a TypeScript port of the same file (`packages/core/src/common/color.ts`, same `8|6|4|3`-digit regex at L3–L5, same fallback at L183–L212), the `transparent` and `catppuccin_*` theme blocks are identical, `hide_border` is still `stroke-opacity` ([`Card.ts` L362–L367](https://github.com/stats-organization/github-stats-extended/blob/master/packages/core/src/common/Card.ts#L362-L367)). Transparency is documented twice, in the same words plus "Any of the available themes turns transparent when `bg_color` carries an alpha channel".

Where it differs, and it matters here: **one image can carry both palettes.** It adds `theme_light` / `theme_dark` and `*_light` / `*_dark` for every colour param, emitting a single SVG whose `<style>` contains `@media (prefers-color-scheme: dark) { .header { fill: … } .card-bg { fill: …; stroke: … } … }` ([`Card.ts` L269–L294](https://github.com/stats-organization/github-stats-extended/blob/master/packages/core/src/common/Card.ts#L269-L294)). Their docs: "`theme_light` / `theme_dark` and the `*_light` / `*_dark` color parameters put both modes in a single card URL, which then follows the viewer's browser or OS setting … GitHub serves the card from its CDN, so a GitHub theme that differs from the browser/OS setting cannot be detected by this approach." The four extra themes are `light_github`, `dark_github`, `light_github_repocard`, `dark_github_repocard`.

### 1.3 `DenverCoder1/github-readme-streak-stats`

Documented options ([`README.md` L138–L145](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/README.md#L138-L145)):

| Param | Documented as | Default |
|---|---|---|
| `theme` | "The theme to apply" | `default` |
| `hide_border` | "Make the border transparent" | `false` |
| `border_radius` | "Number `0` (sharp corners) to `248` (ellipse)" | `4.5` |
| `background` | "hex code without `#`, css color, or gradient in the form `angle,start_color,...,end_color`" | theme |
| `border` | "hex code without `#` or css color" | theme |
| `stroke` | "Stroke line color between sections" | theme |
| `ring` / `fire` / `currStreakNum` / `sideNums` / `currStreakLabel` / `sideLabels` / `dates` | one colour each | theme |

Transparency here is documented by *type*, not by example: `background` accepts a CSS colour, and `transparent` is in the colour whitelist ([`src/colors.php` L146](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/src/colors.php#L146)). The parser also accepts 8-digit hex and converts it ([`src/card.php` L147](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/src/card.php#L147), L707–L760):

```php
} elseif (strlen($color) === 8) {
    return ["color" => "#" . substr($color, 0, 6),
            "opacity" => hexdec(substr($color, 6, 2)) / 255];
```

so `background=00000000` emits `fill="#000000" fill-opacity="0"`, and `background=transparent` is rewritten to `#0000`. Both are supported; neither appears as a literal example in the README.

Gotchas, from source:

- **`hide_border` sets `$theme["border"] = "#0000"`** ([`card.php` L164–L167](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/src/card.php#L164-L167)) — genuinely transparent, not opacity.
- **The `transparent` theme still draws a border and the two dividers**: bg `#0000` but `border` and `stroke` are `#E4E2E2` ([`src/themes.php` L44–L56](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/src/themes.php#L44-L56)). On dark that is a bright frame plus two 1 px white lines; with `hide_border=true` the *stroke* lines stay.
- **Catppuccin is hyphenated here**: `catppuccin-latte`, `-frappe`, `-macchiato`, `-mocha` (four variants). `theme=catppuccin_mocha` also works — `normalizeThemeName` lowercases and replaces `_` with `-` ([`card.php` L110–L113](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/src/card.php#L110-L113)) — so the same spelling works on all three hosts.
- **Partial overrides keep the theme's other colours.** The merge loop only replaces properties you name ([`card.php` L139–L160](https://github.com/DenverCoder1/github-readme-streak-stats/blob/main/src/card.php#L139-L160)). With no `theme=`, the base is `default`: background `#FFFEFE`, **`currStreakNum` and `sideNums` `#151515`**, labels `#151515`, dates `#464646`. So the current Signals URL, which sets `background=1e1e2e` and leaves the numbers alone, renders the three big numbers `#151515` on `#1e1e2e` = **1.11:1**. Source-predicted, unrendered, but that is the entire point of the card disappearing.

### 1.4 What "transparent" means under GitHub light vs dark

There is no server-side theme detection: the image is one fixed SVG, Camo re-hosts it, and the same bytes go to every viewer ("we can not infer the browser/GitHub theme on the server side", [`readme.md` L202](https://github.com/anuraghazra/github-readme-stats/blob/master/readme.md#L202)). So a transparent card is literal: the page canvas shows through (`#ffffff` light, `#0d1117` dark), while **title, text, icons and chart colours stay fixed**. A Mocha-text transparent card is legible on dark and almost blank on light (`cdd6f4` is 1.45:1 on white); a Latte-text card is the reverse (`4c4f69` is 2.37:1 on `#0d1117`).

Three switches exist, and they are not equivalent: (1) `#gh-dark-mode-only` / `#gh-light-mode-only`, GitHub's own tag, follows the **GitHub** theme ([changelog](https://github.blog/changelog/2021-11-24-specify-theme-context-for-images-in-markdown/)) at one image per mode; (2) `<picture>` + `prefers-color-scheme`, which follows the **OS/browser**, not the GitHub theme — mismatch is the known failure mode in [HANDOFF §3](../.scratch/profile-redesign/HANDOFF.md); (3) the successor's `theme_light`/`theme_dark`, same OS signal as (2) but inside one SVG, so no README duplication — whether GitHub's Camo'd `<img>` honours an SVG's internal `prefers-color-scheme` is **unverified here**.

## 2. Pinned repo cards (`/api/pin/`)

`src/cards/repo.js` reads the same five knobs and calls the same `getCardColors` ([L68–L79, L113–L120](https://github.com/anuraghazra/github-readme-stats/blob/master/src/cards/repo.js#L68-L120)): `hide_border = false, bg_color, theme = "default_repocard", border_radius, border_color`.

So: `bg_color` is the panel, `border_color` the 1 px edge, `hide_border` zeroes the edge opacity, `border_radius` the corners, and `theme` defaults to `default_repocard` (same as `default` but icon colour `586069`). **A transparent pin card is supported** — `bg_color=00000000`, exactly as on the stats card, with the same `e4e2e2`-border caveat if you rely on `theme=transparent` instead. The successor's pin card is identical plus `light_github_repocard` / `dark_github_repocard` ("differ only in icon colour").

What a pin card actually draws, for the transparency trade-off: repository **name as title**, a description wrapped to at most three lines (`DESCRIPTION_MAX_LINES = 3`, [`repo.js` L20](https://github.com/anuraghazra/github-readme-stats/blob/master/src/cards/repo.js#L20)), a language dot + name, star count, fork count. No chart, no icon column. Remove the panel and four of these in a 2 × 2 table become eight islands of text whose only grouping is the `<td>` grid — and on this page the table already contributes GitHub's own cell rules ([HANDOFF §3](../.scratch/profile-redesign/HANDOFF.md)), so the cards would lose their fill but not their frame.

## 3. Colour contrast (WCAG 2.1)

Computed locally in Node: sRGB linearisation (`c ≤ 0.03928 ? c/12.92 : ((c+0.055)/1.055)^2.4`), `L = 0.2126R + 0.7152G + 0.0722B`, ratio `(Lmax + 0.05) / (Lmin + 0.05)`. Thresholds: **4.5:1** normal text (1.4.3 AA), **3:1** large text (≥ 24 px, or ≥ 18.66 px bold) and non-text UI/graphics (1.4.11).

### Catppuccin Mocha

| Colour | on `#ffffff` | on `#0d1117` | Verdict (light / dark) |
|---|---|---|---|
| `89b4fa` blue | 2.11:1 | 8.99:1 | fail both / AA text |
| `cba6f7` mauve | 2.03:1 | 9.32:1 | fail both / AA text |
| `94e2d5` teal | 1.49:1 | 12.71:1 | fail both / AA text |
| `a6e3a1` green | 1.49:1 | 12.73:1 | fail both / AA text |
| `fab387` peach | 1.77:1 | 10.69:1 | fail both / AA text |
| `f2cdcd` flamingo | 1.46:1 | 12.97:1 | fail both / AA text |
| `cdd6f4` text | 1.45:1 | 13.09:1 | fail both / AA text |
| `6c7086` overlay0 | 4.88:1 | 3.87:1 | AA text / large-UI only |
| `1e1e2e` base | 16.40:1 | 1.15:1 | AA text / fail both |

**No Mocha accent reaches even 3:1 on white**; the best is `89b4fa` at 2.11:1. On `#0d1117` every accent clears 4.5:1 with room to spare.

### Catppuccin Latte

| Colour | on `#ffffff` | on `#0d1117` | Verdict (light / dark) |
|---|---|---|---|
| `1e66f5` blue | 4.91:1 | 3.85:1 | AA text / large-UI only |
| `8839ef` mauve | 5.41:1 | 3.50:1 | AA text / large-UI only |
| `179299` teal | 3.74:1 | 5.05:1 | large-UI only / AA text |
| `40a02b` green | 3.34:1 | 5.66:1 | large-UI only / AA text |
| `fe640b` peach | 2.98:1 | 6.35:1 | **fail both** / AA text |
| `dc8a78` rosewater | 2.64:1 | 7.16:1 | **fail both** / AA text |
| `4c4f69` text | 7.99:1 | 2.37:1 | AA text / fail both |
| `9ca0b0` overlay0 | 2.60:1 | 7.27:1 | **fail both** / AA text |

### Against the cards' own canvas

| Pair | Ratio | Reading |
|---|---|---|
| `cdd6f4` on `1e1e2e` | 11.34:1 | the current stats/pin text is fine |
| `6c7086` on `1e1e2e` | 3.36:1 | current streak `dates` at 12 px — **fails AA text** |
| `a6adc8` on `1e1e2e` | 7.37:1 | Mocha subtext0 would fix the dates |
| `151515` on `1e1e2e` | 1.11:1 | streak's default `sideNums`/`currStreakNum` — **invisible** |
| `e4e2e2` on `#0d1117` | 14.67:1 | the default border is a beacon on a transparent dark card |
| `e4e2e2` on `1e1e2e` | 12.71:1 | same border on the current solid card |

Takeaways: on dark, Mocha is the right palette and nothing needs changing; on light, **Mocha supplies only two usable text colours (`6c7086` at 4.88:1 and `1e1e2e` at 16.40:1) and no accent above 3:1**, and Latte itself has three tones (`fe640b`, `dc8a78`, `9ca0b0`) that fail even the 3:1 UI floor on white.

## 4. Recommendation

### Options

| Option | Panel | Legible on dark | Legible on light | Cost |
|---|---|---|---|---|
| **A. Solid, one palette** (today) | `1e1e2e` everywhere | yes | yes | the heaviness being complained about |
| **B. Tinted, one palette** | `181825`/`11111b` | yes | yes | still a dark slab on white — no lighter than A |
| **C. Transparent, one palette** | none | yes (Mocha) | **no** — text 1.45:1 | breaks half the audience |
| **D. Transparent, per-mode images** | none | yes | yes | 2× images, README duplication (or migrate to the successor) |

**Recommendation: D for the three Signals cards, A for the four pin cards — and only if the per-mode mechanism can be verified; otherwise stay on A.**

- **Pinned stays solid.** Pin cards are text, not charts: name, up to three description lines, language, stars, forks. With no panel they stop reading as cards and become a wall of descriptions, and their `cdd6f4` text is 1.45:1 on white. Their fill is doing real work — separating two repo columns — so transparency buys nothing and costs structure. Keep `bg_color=1e1e2e`, `hide_border=true`, `border_radius=6` (up from 4.5: on a 2 × 2 grid a slightly rounder corner reads as deliberate, where 4.5 reads as unstyled), `title_color=89b4fa`, `text_color=cdd6f4`, `icon_color=cba6f7`.
- **Signals is where transparency pays.** Stats, top-langs and streak are charts — a rank ring, three progress-bar rows, a ring + fire + numbers. Removing their panels turns 3 slabs into 3 floating diagrams, and the table already groups them. But a transparent Mocha card is illegible on light, so it has to be D: one image per mode via `#gh-dark-mode-only` / `#gh-light-mode-only`, Mocha colours in the dark image, Latte colours in the light one, `hide_border=true` in both.
- **Tinted (B) is a non-option, and one palette is enough only while opaque.** Any tint dark enough to sit on GitHub's dark canvas is a slab on the light canvas, because the background is baked into the SVG. One palette is correct as long as the background is opaque — the card carries its own canvas, so `1e1e2e` + Mocha works in both GitHub modes. The moment alpha < 1, two palettes are mandatory, and even then Latte needs substitutions: on white, `fe640b`, `dc8a78` and `9ca0b0` fail 3:1, so a light-mode Signals block should use `1e66f5` / `8839ef` / `179299` for accents and `4c4f69` / `6c6f85` for text, never Latte peach or rosewater as small text.
- **If migrating is on the table**, the successor turns D into one image: `theme_light=light_github&theme_dark=dark_github&bg_color=00000000` keeps the background transparent in both and swaps title/text/icon colours per mode. That is the only single-`<img>` route to a transparent card that is correct in both themes, and it is a domain swap, not a rewrite. It is also the one recommendation here that depends on unverified browser behaviour.

### What would look sparse or illegible if everything but the accents went transparent

- **Streak** loses its three-column structure. The only dividers are two 1 px `stroke` lines, default `#E4E2E2` — 14.67:1 on `#0d1117`, brighter than any accent. On a transparent card the frame becomes the loudest element unless `stroke` is set near the background (e.g. `313244`) or dropped.
- **Stats** keeps its rank ring and icons but loses the panel that anchors the ring; the percent label inside the ring sits directly on the page, so it needs ≥ 4.5:1 against the *page*, not against `1e1e2e`.
- **Top-langs** survives best: its bars are filled with GitHub's per-language colours, not theme accents, so the chart still has colour. But the language names and percentages are `text_color`, and `cdd6f4` on white is invisible.
- **Pin cards** lose the most and gain the least: four descriptions with no panel, no chart, and a title that no longer looks like a heading.

### Concrete fixes to the current URLs (independent of the above)

1. Streak numbers are unset, so they render `#151515` on `#1e1e2e` (1.11:1). Add `currStreakNum=cdd6f4&sideNums=cdd6f4`, or switch to `theme=catppuccin-mocha`.
2. `dates=6c7086` is 3.36:1 at 12 px. Use `a6adc8` (7.37:1).
3. If any card goes transparent, add `hide_border=true` **and** set `border_color`/`border` to the page background — `theme=transparent` alone still draws `#e4e2e2` on all three hosts.

Could not verify locally:

- Any rendered card. `*.vercel.app` and `streak-stats.demolab.com` are sinkholed on this machine, so the actual SVG output, its byte size, and its appearance on the live profile were never observed — all behaviour is from source.
- Whether GitHub's Camo'd `<img>` honours an SVG's internal `@media (prefers-color-scheme: dark)` (the successor's `theme_light`/`theme_dark` mechanism). The docs assert it; no render was possible here.
- GitHub's exact canvas colours. `#ffffff` and `#0d1117` are taken as given by the brief, not re-measured from GitHub's stylesheet.
- The `td` cell borders on the Pinned/Signals tables — taken from [HANDOFF §3](../.scratch/profile-redesign/HANDOFF.md), not re-observed here.
- Live cache/rate-limit behaviour of the two hosts, and whether `github-readme-stats.vercel.app` currently serves the profile's cards at all.

**Verified on 2026-10-25.**

## Addendum — "just drop the background" is not the mechanism (measured 2026-10-25)

The v1 profile (`cf36e4e`, `d0686ca`) did ship transparent cards, but through `theme=transparent`, not through `bg_color=00000000` alone. Measured on the live streak endpoint:

| variant | text fills | on `#ffffff` | on `#0d1117` |
|---|---|---|---|
| `theme=transparent` | `006AFF` / `417E87` / `0579C3` | 4.66 / 4.61 / 4.64 | 4.06 / 4.11 / 4.08 |
| `background=00000000`, default colours | `151515` / `464646` | 18.26 / 9.44 | **1.04 / 2.01** |
| `background=00000000` + Mocha | `cdd6f4` / `a6adc8` | **1.45 / 2.23** | 13.09 / 8.50 |
| mid-tone Mocha (shipped) | `5d7aa9` / `6f7787` | 4.35 / 4.50 | 4.35 / 4.20 |

Alpha only removes the panel. Legibility is decided entirely by the text fill, which is baked into the SVG. A single palette can clear both canvases only near the luminance crossover, and there the ceiling is **4.35:1** — no colour exceeds it (brute-forced over the grey axis). The shipped palette sits on that ceiling, which is why small card text lands at 4.2–4.5: just under AA 4.5 on one side.

`theme=transparent` reaches 4.06–4.66 by using the same trick with a GitHub-blue palette; it is brighter on white and dimmer on dark than the shipped mid-tone set.
