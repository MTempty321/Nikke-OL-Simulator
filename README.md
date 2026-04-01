# Nikke OL Gear Optimizer

A browser-based overload gear calculator and reroll simulator for **Goddess of Victory: Nikke**. Helps you plan, optimize, and track your overload gear progression with mathematically-backed strategy recommendations.

**No install required** — runs entirely in your browser as a single HTML file. Data is saved locally via `localStorage`.

## Features

### Planner
- **Character database** — store multiple characters, each with 4 gear pieces (Helm, Gloves, Chest, Boots)
- **Flexible goal system** — each of 3 goal slots supports 1–3 acceptable attributes (OR groups), plus a minimum sub-stat tier target (T1–T15). Set 1, 2, or 3 goals per piece — unused slots are treated as wildcards
- **Current state tracking** — input your current attribute lines and sub-stat tiers. The tool detects which goals are already met (attribute + tier) vs. partially met (attribute only)
- **Unified cost calculation** — expected rock cost accounts for *both* attribute acquisition (Change Effects) and sub-stat optimization (% Reset Attributes) in a single number
- **Phase-split breakdown** — see exactly how many rocks go toward getting attribute lines vs. optimizing sub-stat tiers
- **Strategy optimizer** — compares multiple approaches (full reset, lock all matched, selective lock, deviation strategies) and recommends the cheapest path
- **Confidence thresholds** — 50%, 75%, and 90% probability estimates for total rock cost
- **Priority ranking** — all 4 gear pieces ranked by expected cost, so you know what to roll first
- **Per-piece ignore toggle** — exclude completed or skipped pieces from calculations

### Reroll Simulator
- **Practice rolling** with real game probabilities before spending resources
- **Auto-roll mode** — RNG uses actual attribute appearance rates and tier distributions
- **Manual input mode** — enter what you rolled in-game to get live strategy advice
- **Both roll types** — "Change Effects" (rerolls attrs + tiers) and "% Reset Attributes" (rerolls tiers only)
- **Lock system** — permanent rock locks and one-time custom locks (100 currency, 1-roll protection)
- **Live strategy advisor** — after each roll, recalculates expected remaining cost and recommends next action
- **Color-coded previews** — green for upgrades, red for downgrades, so you can evaluate at a glance
- **Cost tracker** — running totals of rocks and custom locks spent
- **Send from planner** — one-click transfer of goals + current state from any character's gear piece into the simulator

## Attribute Line Recommendations

For which attributes to target on each character, refer to the community guides:

- **[Prydwen Institute — OL Gear Recommendations](https://www.prydwen.gg/nikke/guides/overload-gear-recommendations)** — detailed per-character recommendations with priority ordering (Essential / Ideal / Passable)
- [Prydwen — OL Gear Rerolling Guide](https://www.prydwen.gg/nikke/guides/overload-gear-reroll/) — in-game rerolling mechanics explained
- [Prydwen — OL Gear Introduction](https://www.prydwen.gg/nikke/guides/overload-gear-intro/) — general overload system overview
- [Loot and Waifus — Overload Guide](https://lootandwaifus.com/guides/overload-gear-and-stats-priority/) — sub-stat tier table and stat priorities

## How the Math Works

### Game Mechanics

Each gear piece can have up to 3 overload attribute lines. When rolling:

| Line | Appearance Rate |
|------|----------------|
| Line 1 | 100% |
| Line 2 | 50% |
| Line 3 | 30% |

When a line appears, one of 9 attributes is selected with these weights:

| Attribute | Probability |
|-----------|------------|
| Hit Rate, Max Ammo, Charge DMG, Charge SPD, Crit Rate | 12% each |
| Crit DMG, ATK, Element DMG, DEF | 10% each |

**No duplicates** are allowed on a single piece — if Line 1 rolls ATK, then Line 2 and Line 3 draw from the remaining 8 attributes with adjusted (conditional) probabilities.

Each attribute also receives a random **sub-stat tier** (T1–T15):

| Tier Range | Combined Rate | Per Tier |
|-----------|--------------|---------|
| T1–T4 (Common) | 60% | 15% each |
| T5–T10 (Uncommon) | 35% | ~5.83% each |
| T11–T15 (Rare / Gold) | 5% | 1% each |

### Locking and Rock Costs

| Lines Locked | Cost per Roll |
|-------------|--------------|
| 0 | 1 rock |
| 1 | 2 rocks |
| 2 | 3 rocks |

Rock income averages **~1.5 per day** from daily gameplay.

### Monte Carlo Simulation

Calculating exact expected costs analytically for this system is extremely complex due to conditional probabilities (no-duplicate constraint), multiple phases (attribute acquisition → tier optimization), and branching strategy decisions (which lines to lock, when to sacrifice, when to reset).

Instead, this tool uses **Monte Carlo simulation** — it runs thousands of simulated roll sequences using the real game probabilities, then aggregates the results statistically.

For each calculation:
- **8,000 simulations** per strategy candidate (for comparison)
- **20,000 simulations** for the winning strategy (for confidence intervals)
- **1,500 simulations** for sidebar/overview quick estimates (for responsiveness)

Each simulation plays out the full strategy: locking lines as targets are hit, rerolling unlocked lines, then running the sub-stat reroll phase for any lines below their tier target. The total rock cost of each simulation is recorded, and results are sorted to extract:

- **Mean** — the expected (average) cost
- **P50** — the median cost (50% of players will finish at or below this)
- **P75** — the 75th percentile (75% of players finish at or below)
- **P90** — the 90th percentile (90% of players finish at or below)

### Strategy Optimizer

The tool doesn't just calculate cost for one approach — it evaluates multiple strategies and picks the cheapest.

**Strategies compared:**

1. **Full Reset** — ignore all current lines, treat the piece as blank. Sometimes starting fresh is cheaper than working around suboptimal existing lines.

2. **Lock All Matched** (Standard) — lock every line that has a goal attribute, regardless of its current tier. Get remaining attributes via Change Effects, then fix tiers via % Reset Attributes.

3. **Selective Lock** — only lock lines where *both* the attribute AND the tier meet their targets. Lines with the right attribute but a bad tier are left unlocked, hoping to re-roll the same attribute with a better tier naturally.

4. **Deviation Strategies** — lock only a *subset* of matched lines. For example, if Lines 1 and 2 both have goal attributes but Line 3 is empty, it might be cheaper to lock only Line 2 and reroll Lines 1+3 at 2 rocks/roll rather than locking both and paying 3 rocks/roll for just Line 3 (which only has a 30% appearance rate).

**Why "Full Reset" can win:**

If your current lines have the right attributes on Lines 1 and 2 but not Line 3, you'd need to lock 2 lines (3 rocks/roll) and roll until Line 3 appears (30% chance) with the correct attribute (~10–12%). That's ~3/(0.3 × 0.11) ≈ 91 rocks just for that one line. Starting fresh at 1 rock/roll with 3 lines all rerolling might average ~50 rocks total for all 3 attributes. The optimizer catches these cases automatically.

**Phase-Split Costing:**

Every strategy is simulated across two phases:
- **Phase 1 (Attribute Lines)** — "Change Effects" rolls to get the right attributes on all lines
- **Phase 2 (Sub-Stat Tiers)** — "% Reset Attributes" rolls to bring each line up to its tier target

The tool reports both costs separately so you can see where your rocks are actually going. High tier targets (T11+) can make Phase 2 dominate the total cost.

## Hosting

This tool is a single self-contained HTML file with no external dependencies, no build step, and no backend. It runs entirely in the browser. To host your own copy:

1. Fork or clone this repository
2. Enable GitHub Pages in Settings → Pages → Deploy from branch (main, root)
3. The tool will be live at `https://yourusername.github.io/repo-name/`

## Disclaimer

This is a community-made tool and is not affiliated with, endorsed by, or associated with Shift Up, Level Infinite, or the Goddess of Victory: Nikke development team. Game mechanics and probabilities are based on community research and may not be 100% accurate. Use at your own discretion.

All character names and game terminology are property of their respective owners.
