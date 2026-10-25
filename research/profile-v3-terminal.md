# Terminal block on a GitHub profile README — what the fence can actually do (v3)

Scope: how to make a plain ```` ```console ```` block on `github.com/CAI991108` look better, without editing `README.md`.
Everything below was checked on **2026-10-25** from this machine. Evidence tags: **[M]** measured here (GitHub's own renderer/CSS, a live page, or Chromium), **[D]** primary document/source read but not re-tested, **[?]** could not verify.

Method:
- Server-side HTML: `POST http://127.0.0.1:8124/api/render` (the repo's workbench, which proxies `https://api.github.com/markdown` with `mode:gfm`, `context:CAI991108/CAI991108`). **[M]**
- Colours: the CSS GitHub served for `github.com/CAI991108` (`github.githubassets.com/assets/{light,dark,...}.css`, `global-*.css`), plus `getComputedStyle` on the live page in headless Chromium. **[M]**
- Language registry: [`github-linguist/linguist` `languages.yml`](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml) at `main`, and the vendored [`shell-session.cson`](https://github.com/atom/language-shellscript/blob/master/grammars/shell-session.cson). **[M]** (fetched), **[D]** (the repo owns the data).
- Animation: Chromium 156 headless (Edge 156 engine) via CDP, plus the live `github.com/DenverCoder1` page. **[M]**

---

## 1. Highlighting ceiling of code fences

### (a) Which fence languages exist, and how `console` maps to a lexer

GitHub's docs say the language list *is* Linguist's: "We use Linguist to perform language detection and to select third-party grammars for syntax highlighting. You can find out which keywords are valid in the languages YAML file." ([Creating and highlighting code blocks](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks)) **[D]**

Matching is by language **name**, **alias**, or **extension**, case-insensitively; an identifier that matches nothing renders as `<pre lang="…"><code>` with **no highlight wrapper at all** — identical to ```` ```text ````. **[M]**

`console` is an **alias of `ShellSession`** ([languages.yml](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml)): **[M]**

```yaml
ShellSession:
  type: programming
  extensions: [".sh-session"]
  aliases: [bash session, console]
  tm_scope: text.shell-session
```

`text.shell-session` → wrapper class `highlight-text-shell-session`; the grammar is vendored from `atom/language-shellscript`. Grammar shape ([source](https://github.com/atom/language-shellscript/blob/master/grammars/shell-session.cson)): a line starting with `$ # % > ❯ ➜` (optionally prefixed by `user@host:path`) is a prompt + `source.shell` (the *whole* shell grammar, nested); every other line is `meta.output`. That explicit output rule is unique to this grammar — it is why output lines get a colour here and get nothing under `bash`.

Probed through GitHub's renderer (realistic 7-line session, `$ cd ~/work … echo "hi" # greet`): **[M]**

| fence | wrapper | token classes emitted |
|---|---|---|
| `console`, `sh-session`, `shellsession` | `highlight-text-shell-session` | `pl-s1` `pl-c1` `pl-k` `pl-s` `pl-pds` `pl-c` (+`pl-e` for `user@host`, `pl-smi` for `$VAR`) |
| `bash`, `sh`, `shell`, `zsh`, `ksh`, `csh`, `tcsh` | `highlight-source-shell` | `pl-c1` `pl-k` `pl-s` `pl-pds` `pl-c` — output lines get **nothing** |
| `fish` | `highlight-source-fish` | `pl-c1` `pl-s` `pl-pds` `pl-c` |
| `nushell` | `highlight-source-nushell` | `pl-k` `pl-s` `pl-v` `pl-en` `pl-pds` `pl-c` |
| `powershell` / `ps1` / `pwsh` | `highlight-source-powershell` | `pl-k` `pl-c1` `pl-s` `pl-pds` `pl-c` |
| `bat` / `cmd` / `batch` | `highlight-source-batchfile` | `pl-k` `pl-c1` `pl-s` `pl-pds` |
| `ini`, `yaml`, `json`, `dockerfile` | own `highlight-source-*` | unrelated tokens |
| `text`, `plaintext`, `nohighlight`, `terminal`, **`shell-session`**, anything unknown | *none* — bare `<pre lang>` | — |
| `txt` | `highlight-text-adblock` ⚠ | `pl-k` — `.txt` is an extension of Linguist's **Adblock Filter List** |

Two traps: **`shell-session` (with the hyphen) does not work** even though the scope is `text.shell-session` — it is neither name, alias, nor extension, so you get plain text; **`txt` is not `text`** and highlights as Adblock. **[M]**

There are 836 entries in `languages.yml`, 773 with a TextMate scope (63 are `tm_scope: none`). **[M]** counted from the fetched file. GitHub resolves that registry; it does not publish an independent whitelist.

### (b) What a `console` block actually emits, and its colours

Current README lines 17–27, rendered by GitHub: **[M]**

```html
<div class="highlight highlight-text-shell-session" dir="auto"><pre class="notranslate">$ <span class="pl-s1">whoami</span>
<span class="pl-c1">zijin — phd.ai @ cuhksz</span>
$ <span class="pl-s1">ls <span class="pl-k">~</span>/work</span>
<span class="pl-c1">omarchy-NTE  dsh-studios</span>…
```

Class → Primer token → colour, with computed values from the live page: **[M]**

| class | Primer token | light | dark | appears as |
|---|---|---|---|---|
| `pl-s1` (top level) | none — inherits `--fgColor-default` | `#1f2328` | `#f0f6fc` | the command text, i.e. **plain body colour** |
| `pl-c1` | `--color-prettylights-syntax-constant` | `#0550ae` | `#79c0ff` | **every output line**, and shell builtins (`cd`, `echo`) |
| `pl-k` | `…-keyword` | `#cf222e` | `#ff7b72` | `~`, operators |
| `pl-s` / `pl-pds` | `…-string` | `#0a3069` | `#a5d6ff` | quoted strings and their quotes |
| `pl-c` | `…-comment` | `#59636e` | `#9198a1` | `#` comments |
| `pl-e` / `pl-en` | `…-entity` | `#6639ba` | `#d2a8ff` | `user@host:path` prompt prefix |
| `pl-smi` | `…-storage-modifier-import` | `#1f2328` | `#f0f6fc` | `$HOME`-style variables |
| `pl-v` | `…-variable` | `#953800` | `#ffa657` | not emitted by `console` in any probe; shows up in other grammars |
| `<pre>` background | `--bgColor-muted` | `#f6f8fa` | `#151b23` | the code-card background |

The counter-intuitive part: **`pl-s1` has no standalone CSS rule** (only `.pl-smi,.pl-s .pl-s1` exists), so top-level command text is the default foreground; the visible colour in a `console` block comes from *output lines* (`pl-c1`, blue) and the odd `~`/string/comment. Note also that the prompt glyph (`$ `, `❯ `, `% `) is **not wrapped in any span** — it is always default colour. **[M]**
Font is GitHub's UI mono stack (`ui-monospace, SFMono-Regular, …`), so no terminal typeface. **[M]** High-contrast and colour-blind themes shift every value (e.g. light-HC constant `#023b95`); the table is the default light/dark pair. **[M]** (values in `light-*.css` / `dark-*.css`).

### (c) Is there a richer fence?

No. Consequences of the above, all **[M]**:

- Only `console`/`sh-session`/`shellsession` have a *session* grammar, i.e. only they colour output lines and `user@host` prompts.
- `bash`/`shell`/`zsh` colour the *commands* (`cd`, `~`, quotes, comments) but leave output plain — a different look, not a richer one; for `$ whoami` there is nothing to colour, which is exactly what you measured.
- `fish`/`nushell`/`powershell` are worse fits for a Unix-flavoured card (different keywords, no session mode).
- No fence gets ANSI colours, a dark card, a window chrome, a prompt separator, or a custom font. The ceiling is ~6 token classes on GitHub's own muted code-card background. Anything more must be an image (see §2).

---

## 2. Typewriter / animated terminal

### (a) `readme-typing-svg` (`DenverCoder1/readme-typing-svg`)

Mechanism, from the service's source and a live response: it emits **SMIL**, not CSS — one `<text>` per line bound by `textPath` to a `<path>` whose `d` is animated (`<animate attributeName="d" values="m0,80 h0 ; m0,80 h480 ; …">`), chained with `begin="d0.end"`. No `@media`, no `prefers-reduced-motion`, no `prefers-color-scheme` in the output. **[M]** ([source](https://github.com/DenverCoder1/readme-typing-svg/blob/main/src/models/RendererModel.php), [README options table](https://github.com/DenverCoder1/readme-typing-svg#-options))

| question | answer |
|---|---|
| Animates on github.com today? | **Yes.** Live `github.com/DenverCoder1` page, cropped screenshots 1.5 s apart: 716 of 19,800 px changed, next frame 2,260 px (near-blank mid-cycle). **[M]** |
| Cost | 5-line, 480×160 card: **3,337 B** with default `monospace`; **12,608 B** with `font=Fira Code`, of which 9,120 B is a base64 WOFF subset. One request (`<img>` → camo), no sub-resources. **[M]** |
| Theme | One `color` + optional `background` per URL. Light/dark needs `<picture>` with two URLs (2 files, 1 fetched). **[D]** per project FAQ, **[M]** that `<picture>` survives GitHub's renderer |
| Accessibility | Text is a path, so **not selectable and not in the DOM**; screen readers get only `alt`; the generated SVG cannot honour reduced motion; DenverCoder1's own profile ships it with `alt=""`. **[M]** |
| Operational risk | Third-party host, `cache-control: no-cache, must-revalidate`, camo in front; self-hosting instructions are Heroku-era. **[M]** headers, **[D]** README |

### (b) Animated SVG committed to the repo (`assets/*.svg`, loaded via `<img>`)

- **Animates:** yes. Both SMIL (`<animate>`) and CSS `@keyframes` moved pixels between frames in an `<img>` in Chromium 156. **[M]** And on a real page: this profile's own `output/snake.svg` (28,602 B, 36 CSS keyframes, 11 KB of `<style>`, no media queries) animates on `github.com/CAI991108` — 1,040 px changed between two frames 1.5 s apart. **[M]**
- **Serving:** a relative `src` in the README is rewritten by the page to `/<user>/<repo>/raw/<branch>/<path>`, which redirects to `raw.githubusercontent.com` with `content-type: image/svg+xml` (+`nosniff`). Not camo'd, so no third party in the path. **[M]** (measured on a live profile's committed assets; note the `/markdown` API leaves relative `src` untouched — the rewrite is a page-layer step)
- **Reduced motion:** a `@media (prefers-reduced-motion: reduce)` block **inside** the SVG *is* honoured in `<img>` context — verified with Chromium's `--force-prefers-reduced-motion`: the media block applied and the CSS animation froze. ⚠ Caveat: CDP `Emulation.setEmulatedMedia` does **not** reach image documents, so an emulation-only test gives a false negative. **[M]**
- **Theme:** `@media (prefers-color-scheme: dark)` inside the SVG *did* respond in Chromium, but cross-browser support is **[?]** — GitHub's documented route is `<picture><source media="(prefers-color-scheme: dark)" srcset=…>` ([GitHub blog, first-party](https://github.blog/developer-skills/github/how-to-make-your-images-in-markdown-on-github-adjust-for-dark-mode-and-light-mode/)), which survives rendering (GitHub wraps it in `<themed-picture>`). **[M]**
- **Cost:** hand-authored 7-line cards measured **853 B** static, **1,409 B** with a 7-step reveal animation. A per-character typing effect like the service's needs path-length SMIL and lands near the service's 3.3 KB; the real-world snake (much more elaborate) is 28.6 KB. **[M]**

### (c) Animated GIF committed to the repo

- **Animates:** yes, everywhere, no CSS/SMIL involved; it cannot be paused and no media query can stop it. **[D]** (GIF semantics) / **[M]** it renders in `<img>`.
- **Cost:** 480×160, 7 lines, 70 frames at 140 ms ≈ 11 s loop: **78,029 B** (optimised palette). Static PNG of the same card: **9,982 B**. **[M]**
- **Theme:** baked in. Light/dark = two files (or one that fits neither). **Copy/selection:** none; `alt` is the only text. Reduced motion: impossible. **[D]**

---

## 3. Recommendation for a 7-line terminal block

Scores 1–5 (5 best). Bytes are measured for this exact 7-line card.

| option | look | bytes | theme fit | maintenance | break risk |
|---|---|---|---|---|---|
| (a) plain `console`, as today | 2 — blue output, plain commands, GitHub mono on grey | 0 | 5 — GitHub's own tokens | 5 — none | 5 |
| (b) `console` + content/fence tuned | 3.5 — `user@host` purple, `~` red, `#` grey, output blue | 0 | 5 | 5 | 5 |
| (c) committed animated SVG | 4.5 — pixel-exact dark card, typing, custom colours | 1.4 KB (up to ~3.3 KB) | 4 — `<picture>` pair or in-SVG media query | 3 — hand-edit the SVG | 4 — file must stay valid; no sanitisation, no scripts |
| (d) `readme-typing-svg` | 4 — smooth typing, but one colour, no syntax | 3.3 KB (12.6 KB with a font) | 2 — two URLs for light/dark, no auto-fit | 4 — URL params only | 2 — third-party host, camo, no reduced motion |
| (e) static PNG/SVG under `assets/` | 4.5 — pixel-exact | 1–2 KB SVG, 10 KB PNG | 4 — two files, or one SVG that adapts | 3 — re-export on every copy change | 4.5 |

**Recommendation: (b).** Keep the fence and spend the effort on content: a `user@host:~/work` prompt (gets `pl-e`), `~` and quoted paths (get colour), `#` comment lines (grey), and 3 short output lines instead of 5 long ones. It costs nothing, inherits every theme (including colour-blind/HC), stays selectable and screen-reader friendly, and cannot rot. Its ceiling is real: output is blue, commands are body-coloured, and the "card" is GitHub's grey block.

**Fallback: (e), a committed static SVG card** (two files if the card must adapt — `assets/term-dark.svg` + `assets/term-light.svg` behind `<picture>`), built from the 853 B template in `.tmp/anim/term-static.svg`. Use (c) only if motion is the point; avoid (d) — it trades a third-party host, a 3–12 KB request, no reduced-motion support and an unselectable image for an animation that a profile page does not need.

### What is verified vs read

| claim | how |
|---|---|
| Fence → lexer mapping, emitted classes, unregistered fences | Rendered through GitHub's `/markdown` **[M]** |
| `console` = alias of `ShellSession`; 836/773 counts; `txt`→Adblock | `languages.yml` fetched at `main` **[M]** |
| Colours, `<pre>` background, font stack | GitHub's served CSS + `getComputedStyle` on the live profile **[M]** |
| Docs "we use Linguist" wording, theme-image feature | Docs page and GitHub blog fetched **[D]** |
| readme-typing-svg output shape, bytes, no media queries | Live response + source **[M]**; params/FAQ **[D]** |
| It animates on github.com | Two timed screenshots of the live page **[M]** |
| SMIL/CSS animate inside `<img>`; reduced-motion inside SVG | Chromium 156 headless **[M]** |
| raw-served committed SVG actually animates on github.com | Two timed crops of `snake.svg` on the live profile **[M]** |
| Camo byte count for a third-party image | **Could not verify** — camo returned 403 from this network; origin bytes measured instead |
| Firefox/Safari behaviour for in-SVG media queries, SMIL in `<img>` | **Not verified** — Chromium only |

### Appendix: 853 B static card, if (e) is chosen

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="480" height="160" viewBox="0 0 480 160" role="img" aria-label="terminal session">
  <rect width="480" height="160" fill="#1e1e2e" rx="6"/>
  <g font-family="ui-monospace, SFMono-Regular, Menlo, Consolas, monospace" font-size="14">
    <text x="12" y="28" fill="#89b4fa">$ <tspan fill="#cdd6f4">whoami</tspan></text>
    <text x="12" y="48" fill="#a6adc8">zijin — phd.ai @ cuhksz</text>
    <text x="12" y="68" fill="#89b4fa">$ <tspan fill="#cdd6f4">cat focus.txt</tspan></text>
    <text x="12" y="88" fill="#a6adc8">multimodal · diffusion · agents</text>
    <text x="12" y="108" fill="#89b4fa">$ <tspan fill="#cdd6f4">ls ~/work</tspan></text>
    <text x="12" y="128" fill="#a6adc8">omarchy-NTE  dsh-studios</text>
    <text x="12" y="148" fill="#a6adc8">crossledger  crossbooks</text>
  </g>
</svg>
```

Serve as `assets/term-dark.svg` behind `<picture>` with a light twin (`#f6f8fa` bg, `#1f2328` text, `#0550ae` prompt). Keep `role="img"` + `aria-label`; the text inside is not selectable, so the alt/label must carry the content.

Verified on 2026-10-25.
