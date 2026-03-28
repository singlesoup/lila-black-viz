# LILA BLACK — Player Journey Insights

*Derived from 5 days of production data (Feb 10–14, 2026) across 796 matches, 89,104 telemetry events, using the Player Journey Visualization Tool.*

---

## Insight 1: The Eastern Half of Ambrose Valley Is a Ghost Town

**What caught my eye:**
The traffic heatmap for Ambrose Valley shows an immediate, striking imbalance — the eastern 30% of the map (roughly the right-hand third) registers near-zero player presence. Switching the heatmap to "Traffic" mode and zooming in reveals that **59 of 100 grid cells have less than 0.5% of total movement**, and cells in the ~75-90% x-range show *zero visits* across all 5 days of data.

**The evidence:**
Running a grid density analysis across 48,754 position events in Ambrose Valley:

| Zone | Share of total traffic |
|---|---|
| Central corridor (40–60% x, 40–60% y) | ~22% |
| Northwest quadrant | ~31% |
| East side (75%+ from left edge) | <1% |

The 10 most-visited cells hold ~40% of all movement. The 10 least-visited cells hold **combined 0%**.

**Why a level designer should care:**
This is a map utilization crisis. A meaningful chunk of the level art, collision mesh, AI navigation, and audio work is receiving no player attention — wasting engineering budget and failing to deliver the intended experience. The level design likely assumes a storm direction that pushes players toward the east, but players are extracting or dying before they ever reach it.

**Actionable items:**
- Move a high-value extraction point to the eastern area to force players to route through it
- Add a loot cache in the dead zone to justify the traverse
- Metrics to watch: % of matches where any player enters the eastern 25% of the map; average furthest x-coordinate reached per match

---

## Insight 2: Loot Drives Conflict — Kill Zones and Loot Zones Are Nearly Identical

**What caught my eye:**
Looking at the "Loot" heatmap vs. the "Kills" heatmap for Ambrose Valley, the overlays look almost like duplicates of each other. I suspected correlation but wanted to verify it quantitatively.

**The evidence:**
Cross-referencing the top 10 loot-density cells with the top 10 kill-density cells:

- **Ambrose Valley: 9 out of 10 top kill zones are also top loot zones** (90% overlap)
- **Lockdown: 6 out of 10** (60% overlap)

The top 5 loot cells alone account for **35% of all 9,949 loot pickups** in Ambrose Valley — loot is extremely concentrated, not distributed. Players converge on the same 3-5 POIs every match, and those same POIs are where all the fighting happens.

**Why a level designer should care:**
This tells you the current loot layout is creating a funnel: players who land in the top POIs get fights, and players who don't are isolated. The game *could* be encouraging broader map exploration and risk-reward tradeoffs, but instead it's rewarding the same landing spots match after match. This creates predictable, repetitive experiences.

**Actionable items:**
- Introduce secondary loot spawns in currently-underused areas to diversify landing choices
- Monitor: is the kill-loot overlap decreasing after layout changes? (target < 60% overlap)
- Consider region-locking certain high-value items to force route diversity
- Watch session time and extraction rate as proxies for whether players feel forced into early fights vs. strategic looting

---

## Insight 3: PvP Is Almost Nonexistent — The Game Is Effectively PvE

**What caught my eye:**
Filtering the event view to show only "Kill" and "Killed" events (human-vs-human combat), the markers barely appear — even when viewing all 5 days of data across hundreds of matches. Switching to "BotKill" shows abundant activity. The discrepancy is dramatic.

**The evidence:**
Across all 89,104 events in the dataset:

| Event | Count |
|---|---|
| Human kills human (K/Kd) | **4 total** (2 kills, 2 deaths) |
| Human kills bot (BK) | 2,415 |
| Human killed by bot (BKd) | 700 |
| Loot pickups | 14,885 |

Across 567 Ambrose Valley matches with at least one human player, there were **2 human-vs-human kills in total**. That's a PvP encounter rate of ~0.35%.

**Why a level designer should care:**
One of three things is happening, and all three have major design implications:

1. **Population problem**: There are so few concurrent human players that humans almost never share a match. If this is true, the level design's conflict geometry, sightlines, and cover placement are being judged only against bot AI — which has known pathfinding patterns.

2. **Avoidance behavior**: Humans are actively choosing to avoid each other and extract early. The storm or extraction timer isn't forcing enough player-to-player contact.

3. **Bot imbalance**: Matches are so bot-heavy that humans are using all their engagement budget on AI before encountering other humans.

**Actionable items:**
- Instrument a "human encounter rate" metric: % of matches where a human player came within 50m of another human
- Tighten the storm shrink timing to force player convergence
- Reduce bot density in mid-game to create a clearer "human combat zone"
- Target: at least 20–30% of matches should end with one human killing another

---

*All insights above were surfaced using the Player Journey Visualization Tool — specifically the heatmap overlay, event filter, and all-match aggregate view features.*
