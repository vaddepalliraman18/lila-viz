# LILA — Player Journey Visualization Tool

> A browser-based telemetry visualization tool for the LILA BLACK extraction shooter. Turns raw gameplay data into interactive spatial maps that Level Designers can use to understand player behavior.

**Live Demo →** `https://lila-viz.vercel.app` *(deploy your own via instructions below)*

---

## Screenshots

The tool provides:
- Player paths (cyan = human, orange = bot) rendered over procedural minimaps
- Kill/death/loot/storm-death event markers
- Kill, death, and traffic heatmap overlays
- Match timeline playback with phase tracking
- Per-zone activity breakdown in the stats panel

---

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| Frontend | Vanilla HTML/CSS/JS | Zero build step — deploy a single file to any static host |
| Rendering | Canvas 2D API | Direct pixel control for paths, heatmaps, and overlays |
| Fonts | Google Fonts (Orbitron + Share Tech Mono) | Gaming aesthetic, loaded from CDN |
| Data (demo) | Procedurally generated in-browser | Faithfully mirrors real parquet schema |
| Hosting | Vercel / Netlify (static) | One-click deploy, free tier, shareable URL |

No npm, no bundler, no backend required.

---

## Project Structure

```
lila-viz/
├── index.html        ← entire application (self-contained)
├── README.md         ← this file
├── ARCHITECTURE.md   ← system design + coordinate mapping
└── INSIGHTS.md       ← three data-backed level design findings
```

---

## Setup & Local Development

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/lila-viz.git
cd lila-viz

# Run locally (no build needed — just a file server)
npx serve .
# or
python3 -m http.server 8080
```

Open `http://localhost:8080` in your browser.

---

## Deploying

### Vercel (recommended)
```bash
npm i -g vercel
vercel --prod
```
Vercel detects the static site automatically. No config needed.

### Netlify
Drag and drop the `lila-viz/` folder onto [netlify.com/drop](https://netlify.com/drop).

### GitHub Pages
```bash
git push origin main
# Enable GitHub Pages in repo Settings → Pages → Source: main / root
```

---

## Loading Real Parquet Data

The tool ships with synthetic demo data that mirrors the production schema. To load real data:

1. Convert your `.parquet` files to JSON using Python:
   ```python
   import pandas as pd, json
   df = pd.read_parquet('player_data.parquet')
   df.to_json('player_data.json', orient='records')
   ```
2. Click **Upload Parquet Data** in the sidebar and select the JSON file.
3. The tool parses and renders it automatically.

> **Schema expected** (from README in `player_data.zip`):
> `player_id`, `match_id`, `map_id`, `date`, `x`, `y`, `timestamp_s`, `event_type`, `is_bot`

---

## Environment Variables

None. The tool is fully client-side with no API keys or secrets.

---

## Features Checklist

- ✅ Player paths rendered on correct minimap per map
- ✅ Human vs bot visually distinct (cyan / orange)
- ✅ Kill, death, loot, storm-death markers
- ✅ Filter by map, date, match
- ✅ Timeline playback with phase labels + speed control
- ✅ Kill / death / traffic heatmap overlays
- ✅ Coordinate system: world (0–8000) → canvas pixels
- ✅ Per-zone activity bars
- ✅ Hover tooltips with player/event details
- ✅ Fully self-contained — no build, no server

---

## Author

Built as part of the LILA APM Written Test assignment.

---

## If I Had More Time

The one thing I really wanted to add but didn't get to: a "player spotlight" mode where you click on any path on the map and it isolates that single player's full journey — fades everything else out, shows their complete timeline, marks every decision point. Right now you can hover to identify a player but you can't really follow one person's game. That felt like it would be genuinely useful for a Level Designer trying to understand why someone died where they did.

Also the heatmap math is pretty basic (radial gradient per point). With real data volume I'd swap that for a proper KDE grid so the density representation is more accurate at the edges of clusters.
