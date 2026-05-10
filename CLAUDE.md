# CLAUDE.md

Guidance for AI assistants working in this repo. Read this before editing.

## What this repo is

A small collection of **standalone, single-file HTML dashboards** for TFNYC Production. Each dashboard is one self-contained `.html` file (inline CSS + inline JS, no build step, no framework, no package manager) hosted as a static site via **GitHub Pages** and linked from the relevant **Airtable Interface** so staff reach it from inside their normal nav.

Live URLs:
- Hub: `https://doug-hue.github.io/tfnyc-dashboards/`
- Heat Map: `https://doug-hue.github.io/tfnyc-dashboards/heatmap.html`

## Repo layout

```
index.html                    Hub page — cards linking to every dashboard
heatmap.html                  Capacity Heat Map (the canonical, live dashboard)
TFNYC_Capacity_Heatmap.html   Older standalone copy of the heat map; legacy
README.md                     Short human-facing overview
.gitattributes
```

When adding a new dashboard:
1. Create a new `.html` file at the repo root (no subfolders — keep URLs flat).
2. Add a card linking to it in `index.html` under the appropriate section.
3. Commit; GitHub Pages auto-deploys within ~60s.
4. Doug links the new URL on the relevant Airtable Interface page.

## Aesthetic standard — "Bloomberg terminal"

Every dashboard in this repo must follow the same look. The CSS variable block at the top of `heatmap.html` and `index.html` is the source of truth — copy it into new dashboards. Key rules:

- **Fonts**: IBM Plex Mono (body, numbers), Bebas Neue (display headings, KPIs, clock), Barlow (longer prose). Loaded from Google Fonts via `<link>`.
- **Palette**: amber `#F5C400` as the only primary accent on near-black panels (`--black:#0c0f1a`, `--steel:#141828`, `--panel:#1c2236`). Semantic colors: green `#0057A8`, orange `#E07000`, red `#CC2200`. Dim slate text (`--text:#8898b8`, `--dim:#485070`) for secondary content; bright (`--bright:#dce4f8`) only for primary values.
- **Grain overlay**: SVG fractal-noise `body::after` pseudo-element at low opacity.
- **Header**: TFNYC wordmark in Bebas Neue + amber, sticky live clock + green pulse indicator on the right.
- **Heat / RAG tiers**: empty / green / amber / red / red-hot using the `--c-*` variables; install/strike work always rendered in `--c-install` (orange) regardless of department.

Keep everything tight, monospaced, dark, and quiet. Avoid emojis, gradients, and any rounded "friendly" UI chrome.

## Source data: Airtable

All live data comes from the **TFNYC Production v1** base: `appKKB4SwDK9cDcSc`.

Tables currently consumed:
- `Departments` — fields used: `Code`, `Department Name`, `Total Available Hrs/Week`
- `Projects` — fields used: `Project Number`, `Shop Name`, `Status`, `Companies` (linked → client name)
- `Work Orders` — fields used: `WO ID`, `Shop Name` / `Name`, `Status`, `Planned Start`, `Planned End`, `Estimated Hours`, `Department` (linked), `Project` (linked), `% Complete (Lead)` (with legacy `% Complete` fallback)

Live-pull pattern (see `heatmap.html` lines ~1049–1176):
- User clicks "Pull Live", pastes a **Personal Access Token** with `data.records:read` scope on the base.
- Token is stored in `localStorage` under key `tfnyc.pat` — never sent anywhere except `api.airtable.com`.
- `airtableFetchAll` paginates via the `offset` param.
- Linked-record fields arrive as either an array of record-ID strings or an array of `{id, name}` objects depending on the API version — handle both.
- Last successful pull is cached under `tfnyc.heatmap.data.v1` so reloads don't fall back to the embedded snapshot.
- An **embedded `SNAPSHOT` constant** at the top of the script holds the last hand-pulled data so the dashboard renders instantly on first load with no PAT.

When a dashboard's field requirements change (renames, new columns), update both the snapshot and the live-pull mapping; keep a fallback to the legacy field name for one cycle (see the `% Complete (Lead)` / `% Complete` pattern at lines ~1131–1133).

## Heat Map — domain rules to preserve

These encode operational logic; do not "simplify" them away without checking with Doug:

- **Allocation** = `Estimated Hours × (1 − %Complete (Lead))` spread uniformly across **working days** (Mon–Fri) in the window `[max(today, planned_start), planned_end]`. Past hours are sunk; only remaining work is laid out.
- **Overdue compression**: if `planned_end < today` and the WO isn't 100% complete, pile all remaining hours onto today as a single red spike — that's the signal that something is overcapacity *now*.
- **Per-day ceiling** = `weeklyCapacity / 5`. RAG tiers: `>1.50` red-hot, `>1.00` red, `>0.80` amber, else green.
- **Install / Strike / Pack-Load** WOs are detected by name keywords (`INSTALL_KEYWORDS`) and always rendered in orange under a synthetic `__INSTALL__` bucket, regardless of which department they're booked against.
- **PM** department is excluded from the overhire-needed rollup (`NO_OVERHIRE_DEPTS`).
- WOs with status outside `{Pending, Active, QC, Blocked, On Hold}` are dropped on live pull.

## Conventions for editing

- **No build step, no dependencies.** Do not introduce npm, bundlers, frameworks, TypeScript, or external JS libs. Everything is hand-written vanilla HTML/CSS/JS in a single file per dashboard. Google Fonts via `<link>` is the only allowed external resource.
- **Self-contained files.** Don't extract shared CSS or JS into separate files — each dashboard must work as a single `.html` you can email or open from disk. Duplicating the `:root` palette block across dashboards is intentional.
- **UTC dates.** All date math uses `Date.UTC` and `getUTC*` to avoid TZ drift between the user's machine and Airtable. See helpers `ymd`, `parseYMD`, `addDays`, `todayUTC` in `heatmap.html`.
- **Escape user-derived strings** before inserting into HTML — use the existing `escapeHtml` helper.
- **Don't commit secrets.** Never hardcode an Airtable PAT. Tokens live in the user's `localStorage` only.
- **Snapshot freshness.** When you pull new data into a dashboard's embedded `SNAPSHOT` for testing/demo purposes, update the `asOf` string and the "v1 · YYYY-MM-DD" badge on the matching card in `index.html`.
- **No README/docs files unless asked.** This `CLAUDE.md` and the existing `README.md` are the only docs; don't add new `.md` files speculatively.

## Workflow

- Branch: develop on `claude/add-claude-documentation-8mOy8` per the current task assignment, or whatever branch the task spec names. Never push directly to `main` without explicit permission.
- Doug commits/pushes from GitHub Desktop on his end; Claude commits and pushes from sessions when explicitly asked. Use `git push -u origin <branch>` and retry network failures with exponential backoff.
- Do not open pull requests unless explicitly requested.

## Future direction (context, not a current task)

These standalone dashboards are interim. The eventual path is rebuilding them as **Airtable Apps** custom blocks (Enterprise plan) so they embed natively in Interfaces using the SDK with no PAT round-trip. Don't refactor toward that today unless the task explicitly says so — the current single-file pattern is what's deployable right now.
