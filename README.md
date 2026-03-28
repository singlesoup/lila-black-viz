# LILA BLACK — Player Journey Visualizer

A web-based tool for LILA BLACK level designers to explore player movement, combat, and loot patterns across all 3 maps.

**Live demo:** *(deploy URL goes here)*

---

## Features

- **Player paths** — movement trails per player, humans vs bots visually distinguished
- **Event markers** — kills (✕), deaths (○), loot (◆), storm deaths (▲) overlaid on the minimap
- **Heatmaps** — density overlays for traffic, kill zones, death zones, or loot clusters
- **Timeline playback** — watch a match unfold in real time with speed controls (0.5×–30×)
- **Filters** — by map, date, and individual match

---

## Local Development

```bash
# Serve the tool locally (no build step needed)
cd lila-viz/
python3 -m http.server 8080

# Open in browser
open http://localhost:8080
```

Requirements: any static file server. No Node.js or build tools needed.

---

## Deployment

### Vercel (recommended)
```bash
npx vercel --prod
```

### Netlify
Drag-and-drop the `lila-viz/` folder to netlify.app/drop

### GitHub Pages
```bash
git init && git add . && git commit -m "Initial commit"
git remote add origin https://github.com/yourname/lila-viz
git push -u origin main
# Enable Pages in repo settings → Pages → Deploy from branch (main)
```

---

## Project Structure

```
lila-viz/
├── index.html              # Complete visualization app (single file)
├── data/
│   ├── meta.json           # Match metadata for all maps/dates
│   ├── AmbroseValley_2026-02-10.json
│   ├── AmbroseValley_2026-02-11.json
│   ├── ...                 # 15 map+date data files total
├── minimaps/
│   ├── AmbroseValley_Minimap.png
│   ├── GrandRift_Minimap.png
│   └── Lockdown_Minimap.jpg
├── ARCHITECTURE.md         # Tech stack and design decisions
└── INSIGHTS.md             # 3 data-driven findings with evidence
```

---

## Data Pipeline

The raw Parquet files were processed using a pure-Python Parquet + Snappy decoder (no external dependencies). The pipeline:

1. Reads Thrift-encoded footer to find column chunk offsets
2. Decompresses Snappy-encoded data pages per column
3. Groups events by map → date → match → player
4. Exports 16 JSON files for the browser to lazy-load

See `ARCHITECTURE.md` for full details including the coordinate mapping approach.

---

## Regenerating Data

If new parquet files are available, re-run the data pipeline:

```bash
# Requires: Python 3.8+ (stdlib only, no pip installs needed)
python3 scripts/process.py --input /path/to/player_data/ --output data/
```

*(Scripts are in the `scripts/` directory of the repo)*
