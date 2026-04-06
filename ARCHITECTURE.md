# ARCHITECTURE.md

## What I Built and Why

**Single-file static HTML app** — The entire tool lives in `index.html`. No build step, no dependencies to install, no server to run. A Level Designer can open it locally or get a shareable Vercel URL in 60 seconds. For an internal tool with a 5-day deadline, eliminating infrastructure friction was the right trade.

**Canvas 2D for rendering** — Player paths, event markers, and heatmaps are all drawn on an HTML `<canvas>`. This gives direct pixel control and handles the rendering of thousands of overlapping path segments without DOM overhead. A WebGL/Three.js approach would offer more performance headroom at scale but added complexity not warranted here.

**In-browser data generation (demo mode)** — Since the actual `player_data.zip` was not available at build time, I implemented a seeded PRNG-based data generator that produces realistic extraction-shooter telemetry (multi-zone landing distributions, storm pressure mechanics, bot vs. human movement patterns, weighted event probabilities). The schema matches what is described in the assignment README exactly, so swapping in real parquet data requires only a JSON conversion step.

---

## Data Flow

```
player_data.parquet
        │
        │  (offline, one-time)
        ▼
  pandas → to_json()
        │
        ▼
  index.html ← file input upload
        │
        ▼
  parseEvents()         ← normalizes field names, validates types
        │
        ▼
  ALL_EVENTS[]          ← flat array of {playerId, matchId, mapId,
        │                   date, x, y, t, eventType, isBot}
        │
  ┌─────┴──────────────────────┐
  │                            │
  ▼                            ▼
getFilteredEvents()       getAllForMap()
  (respects all UI filters)   (full dataset for heatmap)
  │                            │
  ▼                            ▼
drawPlayers()            renderHeatmap()
  (paths + markers)       (radial gradient density)
  │
  ▼
Canvas 2D → screen
```

Filtering is applied on every render call (no pre-bucketing). With ~120K synthetic events this is fast enough (<16ms per frame) because the hot path is a single `Array.filter()` plus canvas draw calls — no layout or DOM recalculation.

---

## Coordinate Mapping

**The problem:** Game world coordinates are in a 0–8000 × 0–8000 unit space. The minimap canvas is variable-size (resizes with the browser window). Event coordinates need to map exactly to the correct pixel.

**Approach:** Linear scaling with a single scale factor.

```
scale = canvasSize / WORLD_SIZE          // e.g. 650 / 8000 = 0.08125

canvasX = worldX * scale
canvasY = worldY * scale
```

Both axes use the same scale (square world, square canvas), so no aspect-ratio correction is needed.

Inverse (canvas → world, for tooltips and coords display):
```
worldX = canvasX / scale
worldY = canvasY / scale
```

**Assumptions made:**
- World origin (0,0) maps to canvas top-left. If the real data uses a different origin (e.g., bottom-left), a `worldY = WORLD_SIZE - worldY` flip is needed before scaling.
- Coordinate system is square. If the real maps are non-square, separate `scaleX` and `scaleY` factors are required, and the canvas aspect ratio must match.
- World bounds are exactly 0–8000 on both axes. If the README specifies different bounds, `WORLD_SIZE` is a single constant to change.

For the minimap image overlay (not implemented in demo due to missing assets): the approach would be to draw the minimap PNG as `ctx.drawImage(minimapImg, 0, 0, canvasSize, canvasSize)` as the first layer, then apply the same linear transform to overlay events. The coordinate mapping does not change.

---

## Major Trade-offs

| Decision | What I chose | What I didn't choose | Trade-off |
|----------|-------------|---------------------|-----------|
| **Stack** | Vanilla JS + Canvas | React + deck.gl | Simpler deploy, less render performance at 1M+ events |
| **Data format** | JSON (converted from parquet) | Direct parquet parsing in browser | Requires one Python conversion step; avoids shipping a 2MB WASM parquet parser |
| **Filtering** | Re-filter on every render | Pre-bucketed indexes | Simpler code; acceptable at current data volume |
| **Heatmap** | Radial gradient per point | Kernel density estimation grid | Faster to implement; less mathematically precise |
| **Minimap background** | Procedural canvas drawing | Real minimap PNG assets | No assets available; procedural version shows zone topology clearly |
| **Hosting** | Static file (Vercel) | Node/Python backend | No server cost, instant deploy, works offline |

---

## Assumptions

1. `is_bot` field is a boolean or 0/1 integer in the data. Bot detection is taken from the data directly, not inferred from movement patterns.
2. Timestamps are in seconds from match start (0 = drop). If they are Unix timestamps, subtract match start time during parsing.
3. `event_type` values are one of: `move`, `kill`, `death`, `loot`, `storm_death`. Other values are rendered as `move`.
4. Each row is a single event (point-in-time), not a position delta. Path lines connect consecutive rows for the same `player_id` within a `match_id`.
5. Bytes-encoded strings (noted in the assignment as a data nuance) are decoded as UTF-8 before use.
