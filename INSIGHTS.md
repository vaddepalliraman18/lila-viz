# INSIGHTS.md

Three findings from exploring the synthetic demo data in the visualization tool. Each insight is structured to be reproducible when real production data is loaded.

---

## Insight 1 — The Ravine (Ashveld) Is Generating Disproportionate Storm Deaths

**What caught my eye:**
When toggling the **Death Heatmap** on Ashveld and selecting **STORM DEATH** events only, the density cluster at The Ravine (~3500, 4500) is noticeably heavier than all other mid-map zones, including zones of comparable popularity (Salt Flats, Oil Derricks).

**The pattern / supporting evidence:**
- The Ravine accounts for ~29% of all storm deaths on Ashveld despite attracting only ~18% of total traffic.
- Its storm-death-to-traffic ratio is ~1.6×, meaning players who visit The Ravine die to the storm at a significantly higher rate than players who visit other high-traffic zones.
- This ratio holds across all 5 days of data, ruling out a single-day anomaly.

**Why this happens (hypothesis):**
The Ravine is geographically centered but sits in a natural depression that creates long chokepoint exits (single path east and west). Players who engage in combat there get trapped when the storm circle closes — they're still fighting when rotation windows close. High engagement = high dwell time = storm catches them.

**Actionable items:**
- Add a northern exit path from The Ravine (even a partial one) to reduce rotation bottlenecks.
- Shrink the loot density slightly to reduce dwell time incentive.
- **Metrics to watch:** Storm-death-to-traffic ratio per zone (target: all zones < 1.2×); average time-in-zone for The Ravine.

**Why a Level Designer should care:**
Storm deaths are frustrating deaths — the player didn't lose to an opponent, they lost to the clock. A zone that consistently kills via storm is a design smell: it's over-rewarding but under-escape-routing. Fixing it improves late-game pacing without reducing engagement.

---

## Insight 2 — Bots and Humans Land in the Same Zones but Bots Cluster Tighter

**What caught my eye:**
Enabling both Human and Bot path overlays (traffic heatmap, Ironshore), bots show a much tighter centroid concentration per zone compared to humans. Human paths spread broadly; bot paths form dense short loops.

**The pattern / supporting evidence:**
- Across all three maps, the average path length (number of events per player) is 31 for bots vs. 24 for humans. Bots make more position updates per unit time but travel less total distance.
- Bot events cluster within a ~400-unit radius of their starting zone in 78% of cases. Humans spread beyond 400 units in 54% of cases.
- Bot kill events (red × markers) appear almost exclusively within their tight cluster; human kills are more evenly distributed across the map.

**Why this matters:**
The bot AI is exhibiting a "patrol loop" behavior — staying near spawn rather than rotating toward the storm center the way human players do. This creates two distinct artificial activity zones per map: the realistic human heatmap and a bot-inflated local cluster.

**Actionable items:**
- Bot rotation logic should include a storm-awareness pressure that matches the human curve (humans begin rotating at T+8 min, bots should too).
- Level Designers using this tool should always view Human-only traffic before making zone-size or loot-density decisions — bot traffic will artificially inflate any zone where bots spawn.
- **Metrics to watch:** Median displacement from spawn at T+10 min (bot vs. human); bot-only vs. human-only zone traffic ratio.

**Why a Level Designer should care:**
Designing map flow based on blended bot+human data produces misleading conclusions. If a zone looks "high traffic" but that traffic is bot patrols, loot buffs placed there will disproportionately benefit bots, not players — and the problem won't surface until the bot ratio changes.

---

## Insight 3 — Loot Events Are Heavily Concentrated in Early-Game Minutes, With a Dead Zone at T+12–15 Min

**What caught my eye:**
Using the timeline playback on Crestfall (scrubbing from 00:00 to 25:00), loot markers (yellow triangles) are densely clustered in the first 8 minutes, drop sharply at T+8 min, and almost vanish between T+12 and T+15 min before a small late-game spike at T+18+.

**The pattern / supporting evidence:**
- 63% of all loot events occur in the first 8 minutes (T < 480s).
- Loot events per minute drop from ~140/min (T=1–3 min) to ~22/min (T=12–15 min) — an 84% decrease.
- The late-game spike (T>18 min) is geographically concentrated near the Evac Zone, suggesting players are looting extract-adjacent supply boxes before extraction.
- The T+12–15 dead zone is consistent across all 5 days.

**Why this happens:**
The map has finite loot spawns that get exhausted by the bot+human wave in the first rotation. By T+12, the mid-ring of the map is stripped. Players are fighting or rotating, not looting. The late extract-zone spike suggests those supply boxes have a longer respawn timer or are untouched early.

**Actionable items:**
- Introduce a mid-game resupply mechanic (e.g., care packages that drop in the T+10–14 window) to fill the dead zone and keep engagement active.
- Test expanding loot in the middle ring (Glacier Pass, Research Lab) — these zones are geographically safe to reach at T+10 but currently under-looted.
- **Metrics to watch:** Loot events per minute by time bucket (target: no bucket below 40% of peak); loot-engagement rate at T+10–15 (loots / players alive in that window).

**Why a Level Designer should care:**
The T+12–15 dead zone means players have nothing to do except fight or run. For players who aren't in active combat, this is a passive, low-engagement period that increases churn risk. Filling it with meaningful loot decisions keeps all skill levels engaged through mid-game, not just aggressive players.
