# Architecture — LILA BLACK Player Journey Visualizer

## What I Built

A browser-based, static visualization tool for LILA BLACK telemetry data. Level Designers can explore player movement paths, event markers, and density heatmaps across 3 maps, 5 days, and 796 matches — with timeline playback.

**Tech stack:** Vanilla HTML/CSS/JS (no framework) · Python (data pipeline) · Static JSON files

---

## Tech Stack Choices & Why

| Layer | Choice | Rationale |
|---|---|---|
| Data pipeline | Pure Python (stdlib only) | No network access to install pyarrow/fastparquet, so I wrote a minimal Parquet + Snappy decoder from scratch |
| Data storage | Pre-processed JSON, split by map+date | 15 files at 10–640KB each. Fast lazy loading — user only downloads what they're viewing |
| Frontend | Vanilla HTML5 Canvas + JS | No build step needed, zero dependencies, fastest iteration. Canvas is ideal for map rendering and heatmap pixel manipulation |
| Hosting | Static (GitHub Pages / Netlify / Vercel) | No server required — all processing happens at build time |

---

## Data Flow

```
Raw parquet files (1,243 files)
    │
    ▼
Python parser (pure stdlib)
  ├── Reads Thrift-encoded Parquet footer → column chunk offsets
  ├── Reads Snappy-compressed data pages per column
  └── Decodes: PLAIN floats, INT64 timestamps, RLE_DICTIONARY strings
    │
    ▼
Structured Python dicts
  ├── Groups events by: map → date → match_id → player
  └── Separates movement (Position/BotPosition) from events (kills, loot, etc.)
    │
    ▼
JSON export (16 files)
  ├── meta.json          — match counts and stats per map/date
  └── {Map}_{date}.json  — all match data for that map+date chunk
    │
    ▼
Browser (index.html)
  ├── Fetches meta.json on load
  ├── Fetches map+date chunk when user selects a date
  └── Renders on HTML5 Canvas: minimap → paths → event markers → heatmap
```

---

## Coordinate Mapping Approach

The game uses a 3D world coordinate system (x, y, z) where:
- `x` = horizontal position (east-west)
- `y` = elevation (unused for 2D mapping)
- `z` = depth (north-south)
- Origin is at `(0, 0, 0)` in the world

Each map has a documented `scale` and `origin (ox, oz)`:

```
u = (x - origin_x) / scale          → 0..1 normalized
v = 1 - (z - origin_z) / scale      → 0..1 normalized, Y-flipped

pixel_x = u × canvas_width
pixel_y = v × canvas_height
```

The Y-flip (`1 - v`) is critical: minimap images have origin at top-left, but game world Z increases northward (upward). Without the flip, all positions appear mirrored vertically.

**Verification:** I confirmed the mapping by checking that `Position` event coordinates for known maps remained within the 0–1 UV range across all files.

---

## Assumptions Made

| Ambiguity | Assumption |
|---|---|
| `ts` column meaning | README says "time elapsed within match" but values look like ms since Unix epoch (~Jan 1970). I treat them as relative timestamps within each match and normalize to 0–1 for timeline display. |
| Snappy compression format | parquet-go writes raw Snappy (no Hadoop framing), with a 2-byte prefix for rep/def level bit-widths (both 0 for flat required schema). Verified by checking decompressed size matches `uncompressed_page_size`. |
| Dataset completeness | Only ~1.56 files/match (vs expected 50 players/match), so this is a sampled dataset. Individual match views show 1–2 players. Heatmaps (all-match aggregate) are more informative. |
| Bot detection | UUID user_id = human, numeric user_id = bot. Validated against event types (bots only produce `BotPosition`/`BotKill`/`BotKilled`). |
| No `.parquet` extension | Files are valid Parquet despite the `.nakama-0` extension. Standard Parquet magic bytes (`PAR1`) confirmed. |

---

## Major Tradeoffs

| Tradeoff | What I chose | What I didn't |
|---|---|---|
| Pure-Python Parquet reader | Full control, no dependencies | Slower build (~2 min), more fragile |
| Pre-processed JSON vs. real-time backend | Zero hosting cost, simple deploy | Can't query across dates dynamically |
| Canvas rendering vs. WebGL/Deck.gl | Simpler code, works everywhere | Lower performance ceiling for very large datasets |
| Single-file HTML | Easy to share/deploy | Harder to maintain as it grows |

---

## What I'd Do Differently With More Time

1. **Use PyArrow or DuckDB** — my pure-Python Parquet reader works but is brittle. With package access, a 10-line DuckDB query replaces ~400 lines.

2. **Player identity tracking** — currently paths are per-file. I'd link multiple files from the same user_id to show a player's journey *across matches*.

3. **Named POI overlay** — add labeled zones ("Hilltop","Market", etc.) so level designers can reference areas by name, not grid coordinates.

4. **Better timeline resolution** — aggregate matches by real clock time (not just match-relative) to see how behavior changes across a day or week.

5. **Export functionality** — let designers download a screenshot or CSV of the current heatmap view for use in documentation.
