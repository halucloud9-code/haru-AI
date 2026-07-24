# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

**α共鳴シート (Alpha Resonance Sheet)** — a single-page, offline-first web app for
recording peer notes during a ~23-person pitch/retreat event
(*Retreat α · 2026-05-13*). While each presenter pitches, the user quickly records
what talents they saw, who/what they could connect the presenter to, what they can
offer, and how much they want to talk afterward. Everything is stored locally in the
browser and can be exported to CSV/JSON.

The entire app — markup, CSS, data, and logic — lives in **one file**: `index.html`.
The UI is in **Japanese**.

## Repository layout

```
.
├── index.html    # The entire app: HTML + <style> + <script> + seed data (~1200 lines)
├── README.md     # User-facing usage notes (Japanese)
└── .gitkeep
```

There is **no build system, no package manager, no dependencies, no tests, and no
framework**. Do not add any of these unless the user explicitly asks. Keep the app a
self-contained static file that runs by opening it in a browser.

## Running / previewing

Just open the file — there is nothing to compile:

```bash
# Simplest: open index.html directly in a browser.
# Or serve it (useful in a remote/headless environment):
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Because state persists in `localStorage`, testing a clean run means clearing it
(the in-app **全消去 / Clear all** button, or clearing site data / a private window).

## Architecture of `index.html`

Three regions in one file:

1. **`<style>` (top)** — all CSS. The design system is driven by CSS custom
   properties under `:root` (`--bg`, `--ink`, `--gold`, `--crimson`, etc.). It's a
   warm cream/sepia/gold "editorial" aesthetic, light-mode only
   (`color-scheme: light`). The Japanese serif font stack (Yu Mincho / Hiragino
   Mincho) is deliberate. Change colors/spacing via the variables, not scattered
   literals.

2. **`<body>` markup** — a two-column grid (`.layout`): a sticky `<aside>` sidebar
   (participant name, progress, presenter list, TOP 3, export/import actions) and a
   `<main>` panel showing the current presenter's profile plus four input sections
   (Talent Seen, Connection, Offering, Priority) and a save/navigation bar. The
   markup is a static shell; lists and dynamic content are populated by JS into
   elements referenced by `id`.

3. **`<script>` (bottom)** — plain vanilla ES (no modules, no libraries). All state
   and behavior lives here.

### Data model (constants at the top of the script)

- **`PRESENTERS`** — the seed array of ~23 presenter objects. This is the master
  data. Each object:
  ```js
  {
    no, name, status,        // status ∈ 両方取得済 / 才覚のみ / UAAMのみ
    realm,                   // one-line "領域" statement
    values[], talents[], passions[],  // 才覚領域 (may be empty [])
    uaam: { will, knowledge, skill, drive },  // 0–100 or null (未受験)
    type,                    // archetype label (may be "")
    note                     // free note shown as ※ (may be "")
  }
  ```
- **`UAAM_PILLARS`** — the four UAAM axes (志/知/技/衝 → WHY/THINK/HOW/ACT), each
  with four `tags` (16 total). These tags are the selectable chips in the
  "Talent Seen" section.
- **`TALENT_MAX = 3`** — max talent chips selectable per presenter.
- **`STORAGE_KEY = 'alpha-resonance-sheet-v2'`** — the `localStorage` key. Bump the
  version suffix only when a schema change would break old saved data.

### Runtime state

`state = { participantName, activeIndex, entries }`, persisted to `localStorage` on
every change. `entries` is keyed by presenter `no`; each entry is an `emptyEntry()`:
`{ talents[], talentFree, connection, offering, priority, updatedAt }`. `PRESENTERS`
is static seed data and is **not** stored — only the per-presenter `entries` and the
participant name are.

### Key functions

- **`loadState()` / `saveState()`** — read/write `localStorage`, with defensive
  parsing (bad data falls back to empty state).
- **`render()`** — repaints the whole main panel for `currentIdx`; delegates to
  `renderAxes`, `renderUaam`, `renderTagPool`, `renderPriority`, `renderSidebar`.
- **`captureTexts()`** — pulls the textarea/input values into the current entry;
  called on input/blur and before navigation.
- **`toggleTag()` / `setPriority()`** — mutate the current entry and re-render.
- **`goTo(i)` / prev / next** — navigate presenters (also bound to ←/→ arrow keys
  when not typing in a field).
- **`exportCsv()` / `exportJson()` / `importJson()`** — data I/O via the `download()`
  helper (Blob + object URL). CSV is UTF-8 with a BOM for Excel; values go through
  `csvEscape()`. Always use `escapeHtml()` when injecting presenter/user text into
  `innerHTML`.

## Conventions

- **Single file, zero dependencies.** All additions stay inline in `index.html`.
  Don't introduce npm, bundlers, CDNs, or external assets.
- **Vanilla DOM.** Manipulate via `document.getElementById` / `createElement`; no
  reactive framework. When adding dynamic content, follow the existing
  `render*` + event-listener pattern.
- **Persist through `state` + `saveState()`.** Never write to `localStorage`
  directly outside the load/save helpers.
- **Escape user/seed text** with `escapeHtml()` (HTML) or `csvEscape()` (CSV) at
  every injection point.
- **Japanese UI copy.** Match the existing tone and terminology (才覚 / 共鳴 /
  UAAM 16軸). Keep labels bilingual where the design already pairs JP · EN.
- **Style via CSS variables** in `:root`; the app is intentionally light-mode only.
- **Editing seed data:** to change participants, edit the `PRESENTERS` array. Keep
  `no` values contiguous and the object shape complete (empty `[]`/`""`/`null` for
  missing data, as existing rows show). The sidebar count, progress bar, and CSV all
  derive from this array automatically.

## Git workflow

- Active development branch: **`claude/claude-md-documentation-ujeddr`**. Develop,
  commit, and push here; push with `git push -u origin claude/claude-md-documentation-ujeddr`.
- Default branch is `main`. Do not push to it without explicit permission.
- Do not open a pull request unless the user explicitly asks.
- Commit history shows work lands via PRs merged into `main` from short-lived
  `claude/*` and `codex/*` branches.
