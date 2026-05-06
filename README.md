# tfnyc-dashboards

Custom HTML dashboards for TFNYC Production. Each lives here as a static file, served via GitHub Pages, linked from the relevant Airtable Interface so staff reach it from inside the operational nav — never as a loose file in Drive.

## Live URLs

- **Hub** — `https://YOUR-USERNAME.github.io/tfnyc-dashboards/`
- **Capacity Heat Map** — `https://YOUR-USERNAME.github.io/tfnyc-dashboards/heatmap.html`

*(Replace `YOUR-USERNAME` with the GitHub account or org hosting this repo.)*

## What's in the repo

| File | Audience | Description |
|---|---|---|
| `index.html` | Anyone | Hub page listing all dashboards. Top-level entry point. |
| `heatmap.html` | Drew (Shop Manager), PMs, Director | Per-day load by project × department. RAG vs daily ceiling, install/strike in orange, shop total + overhire-needed row at bottom. Click "Pull Live" to refresh from the Airtable base. |

## How dashboards get added

1. Drop a new `.html` file into this repo (Claude or anyone authoring custom dashboards)
2. Add a card to `index.html` linking to the new file
3. Commit + push from GitHub Desktop
4. Add a link to the new dashboard's URL on the relevant Airtable Interface page (e.g. on the Director or Shop Manager Interface)

## Aesthetic standard

All TFNYC dashboards in this repo follow the **Bloomberg-terminal aesthetic** locked in `_CONTEXT/feedback_dashboard_aesthetic.md`:

- IBM Plex Mono (body/numbers) + Bebas Neue (display) + Barlow (longer prose)
- Amber primary (`#F5C400`) on near-black panels
- Grain overlay
- Sticky live clock + pulse indicator
- Reference dashboards in `G:\My Drive\CLAUDE\TFNYC PRODUCTION\dashboard examples\`

## Source data

Dashboards pull from the **TFNYC Production v1** Airtable base (`appKKB4SwDK9cDcSc`). Live refresh requires a Personal Access Token with `data.records:read` scope on that base. Token is stored in browser localStorage on first refresh — never sent anywhere except Airtable's API.

## Future direction

These standalone dashboards are interim. Path A is rebuilding as Airtable Apps custom blocks (Enterprise plan required) so they embed natively in Interfaces with no PAT and live SDK data. Spec at `G:\My Drive\CLAUDE\TFNYC PRODUCTION\_CONTEXT\airtable-apps-sdk-migration-spec.md`.

## Workflow for updates

- **Claude writes** files into the local clone of this repo (Drive-synced folder)
- **Doug commits + pushes** via GitHub Desktop
- **GitHub Pages auto-deploys** within ~60 seconds
- **URL stays stable forever** — no broken bookmarks, no re-uploading
