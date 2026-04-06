# INSIGHTS.md

Three things I noticed while actually using the tool. I spent most of my exploration time on Ashveld since it had the clearest zone separation, then cross-checked patterns on the other two maps.

---

## Insight 1 — The Ravine is quietly killing more people than it should

I almost missed this one. I was looking at the kill heatmap on Ashveld and The Ravine looked pretty normal — decent engagement, mid-tier traffic. But when I switched to storm deaths only, it lit up way more than I expected compared to zones with similar or higher traffic. That's what made me dig in.

The numbers: The Ravine is pulling in roughly 18% of total Ashveld traffic but accounting for about 29% of storm deaths. Salt Flats has more traffic and way fewer storm deaths. The ratio held across all 5 days, so it's not just one bad session.

My read on why: The Ravine has good loot which makes people want to stay and fight. But it's also a natural depression with limited exits — basically once a fight breaks out there, you're committed. By the time it ends, the storm window has closed. Players aren't dying because they ignored the storm, they just couldn't leave fast enough.

**What I'd do about it:** Add even one additional exit path on the north side. Doesn't need to be obvious — a narrow break in the terrain would do it. Also worth looking at whether loot density there is pulling people in longer than other comparable zones.

**Metrics to watch:** Storm-death-to-traffic ratio per zone. If The Ravine is at 1.6x and everything else is under 1.2x, that gap is the problem.

**Why a level designer should care:** Storm deaths feel cheap to players. You didn't outplay them, the clock did. A zone that keeps generating those is slowly eroding trust in the map's fairness — even if players can't articulate why they stopped landing there.

---

## Insight 2 — Bot and human traffic look the same at a glance but tell very different stories

This one surprised me honestly. I assumed bots would land in different zones or behave obviously differently. They don't — they land in roughly the same popular spots humans do. The difference only shows up when you isolate the path overlays.

Bots barely move. Their paths form these tight little loops around wherever they spawned. Humans spread out, push toward storm center, rotate. Bots just... stay. Kill events for bots are almost all within a small radius of their starting position. Human kills scatter across the whole map.

The practical problem: if you're using the blended traffic heatmap to decide where to add loot or adjust zone size, you might be reacting to bot congregation, not actual player behavior. A zone can look popular because 15 bots are patrolling it in circles.

**What I'd do about it:** Two things — first, the bot rotation logic needs storm-awareness, they should be moving toward safety like humans do around the 8–10 minute mark. Second, I'd make Human-only view the default in this tool. It's too easy to accidentally design around bot traffic.

**Metrics to watch:** Median distance from spawn point at T+10 min, split by bot vs. human. Right now bots are barely moving. That gap shouldn't be as large as it is.

**Why a level designer should care:** The map is designed for human players. If 28% of your traffic data is bots standing still, your heatmaps are lying to you.

---

## Insight 3 — There's a dead stretch in the mid-game where basically nobody is looting

I noticed this while scrubbing through the Crestfall timeline. In the first 8 minutes loot events (yellow triangles) are everywhere. Then around T+8 they drop off fast. By T+12–15 it's almost completely quiet on loot — players are alive, they're moving, but nothing is being picked up. Then there's a small spike near the extract zones late game.

Running the numbers: about 63% of all loot events happen in the first 8 minutes. Between T+12–15, loot per minute drops around 84% from peak. That dead zone is consistent across all 5 days.

What's happening is pretty straightforward — the map gets stripped. Early rotators clean out the loot and by mid-game there's nothing left. The late spike near extract zones is players grabbing supply boxes right before they leave, those apparently have either a longer respawn or just nobody touches them early.

**What I'd do about it:** Some kind of mid-game resupply — care packages, a second-wave spawn, something that drops into the T+10–14 window specifically. Also worth checking Glacier Pass and Research Lab on Crestfall — they're positioned well for mid-game rotation but barely show any loot events, which suggests players are passing through without finding anything worth stopping for.

**Metrics to watch:** Loot events per minute by 3-minute time bucket. Right now there's a cliff at T+8. Ideally that curve flattens rather than drops.

**Why a level designer should care:** That 12–15 minute window is where players who aren't in a fight have nothing to do. They're just running. That's the window where people alt-tab or mentally check out. It's a pacing problem that looks like a loot problem but is really an engagement problem.
