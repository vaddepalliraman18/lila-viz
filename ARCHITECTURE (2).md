# ARCHITECTURE.md

## What I Built and Why

**Single-file static HTML app** — Everything lives in `index.html`. I went back and forth on this — first instinct was React + Vite since that's what I reach for on UI-heavy stuff. But I kept thinking about the person actually evaluating this. A Level Designer shouldn't need to clone a repo and run `npm install` just to open a tool. One file, shareable link, done. That felt more useful.

**Canvas 2D for rendering** — I briefly tried doing player paths as SVG elements because hover events are easier to attach. It got sluggish quickly with hundreds of path nodes in the DOM. Switched to Canvas and it was immediately smoother. The heatmap in particular — radial gradients per-point on Canvas is much cleaner than anything I would've hacked together in SVG. Kept Canvas after that.

**In-browser data generation (demo mode)** — The actual `player_data.zip` wasn't available during the build, so I wrote a seeded random generator that mirrors the schema from the assignment README. It's not perfectly realistic — real player movement has more directional momentum than what I simulated — but it was enough to build and validate every UI feature. Loading real data is a one-step JSON conversion away.

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

A few things weren't clear from the README so I made calls and noted them:

1. `is_bot` is a boolean or 0/1 integer — reading it directly from the data rather than inferring bot-ness from movement patterns (I considered that approach but it felt too fragile to be reliable).
2. Timestamps are seconds from match start (0 = drop). If the real data uses Unix timestamps, subtract match start time during parsing — easy fix.
3. `event_type` values: `move`, `kill`, `death`, `loot`, `storm_death`. Anything else defaults to `move` rather than erroring out.
4. Each row is a snapshot event, not a position delta. I draw lines between consecutive rows per player per match to form the path.
5. Bytes-encoded strings get decoded as UTF-8. The README flagged this as a nuance so I made sure to handle it.
