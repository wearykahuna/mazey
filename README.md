# mazey
Mazey game a Son and Dad project

# Mazey Game 🎮

A browser-based maze adventure game built in a single `index.html` file. No dependencies, no build step, no server required — just open the file in a browser or deploy via GitHub Pages.

---

## Playing the game

Visit the live URL, enter the password, and use **arrow keys** to navigate through the maze.

**Password:** `oisinmazeyclub`

**Controls:**
- Arrow keys — move
- SPACE — activate magic hat teleport (if equipped)

---

## Deployment

Hosted on GitHub Pages from the `main` branch. The entry point must be called `index.html`.

**Live URL:** `https://[username].github.io/mazey-game`

To update: commit changes to `main` and GitHub Pages redeploys automatically within ~2 minutes.

---

## File structure

Everything is in one file: `index.html`

```
index.html
├── <style>          CSS — password screen, game layout, shop, mobile hint
├── Password screen  HTML shown before the game loads
├── Game screen      HTML — canvas, controls, shop panel
└── <script>
    ├── checkPassword()   Gate — checks against CORRECT_PW constant
    └── initGame()        Everything else runs inside here
        ├── getCanvasLimits()   Responsive sizing logic
        ├── localStorage helpers
        ├── BOTS / THEMES / LEVELS / SHOP   Game data
        ├── State variables
        ├── buildMaze()         Recursive backtracker maze generator
        ├── bfs()               Pathfinding (used for main path + octopus AI)
        ├── Octopus AI          40% random wander, 60% pathfind to player
        ├── setupLevel()        Builds maze, places all items, resets state
        ├── drawAll() + draw*() Canvas rendering functions
        ├── canMove()           Wall + special gate collision
        ├── processArrival()    Handles all collectibles and hazards on landing
        ├── move() / doTeleport()
        ├── renderShop()        Builds shop HTML dynamically
        └── Event listeners + animation interval
```

---

## Canvas sizing

The canvas resizes responsively using `getCanvasLimits()`:

```javascript
// Desktop (>= 768px): 90% viewport width, up to 1140px; 78% viewport height, up to 860px
// Mobile (< 768px):   screen width minus small padding; 62% viewport height

function getCanvasLimits() {
  const w = window.innerWidth, h = window.innerHeight;
  if (w < 768) return { maxW: w - 16, maxH: h * 0.62 };
  return { maxW: Math.min(w * 0.90, 1140), maxH: Math.min(h * 0.78, 860) };
}
```

Tile size is then: `Math.min(floor(maxW / cols), floor(maxH / rows))`

A debounced `resize` event listener rescales the canvas without resetting game state (positions are stored as grid coordinates, not pixels).

A mobile hint banner is shown via CSS media query at `max-width: 768px`:
> 💻 For the best experience, try this on a laptop or desktop!

---

## Saving progress

Uses `localStorage` — free, built into every browser, no account needed. Saves instantly on every relevant action. Per-device and per-browser (progress does not sync across devices).

| Key | Value | What it stores |
|-----|-------|----------------|
| `mazey_progress` | integer | Highest level unlocked (0 = only level 1 available) |
| `mazey_coins` | integer | Total coin balance |
| `mazey_inventory` | JSON object | `{ hats: [], boosters: 0, food: 0, equippedHat: null }` |

"Reset All Progress" button wipes all three keys and resets to level 1.

---

## Levels

| # | Bot | Theme | Special feature |
|---|-----|-------|-----------------|
| 1 | Mr Brown | Default | Basic maze |
| 2 | Geoffrey | Default | Larger maze (24×18) |
| 3 | Mr Obvious | Default | 2 trap doors (off main path, won't block finish) |
| 4 | Mr Lighty | Default | Cheese collectible (off path, speech bubble on collect) |
| 5 | Seeky | Default | 2 keys + 2 doors in strict order on the main path |
| 6 | Mr Brown | Default | Octopus chaser (starts off path, 40% random wander) |
| 7 | Geoffrey | Nature (green) | Rose collectible (on path midpoint, speech bubble) |
| 8 | Axy | Default | 5 coins to collect (bigcoins mode) |
| 9 | Mr Lighty | Dino (dark brown) | 4 bones to find, stone gate blocks finish until all collected |
| 10 | Seeky | Underwater (dark blue) | 3 treasure chests giving +10 coins each |
| 11 | Geoffrey | Default | 3 chickens appearing sequentially, fence gate blocks finish |

**Level progression:** Beat a level to unlock the next. A "Jump to level" dropdown lets players resume at any unlocked level.

**Coins on every level:** All levels have 3 coins (level 8 has 5) placed along the main route, avoiding other items. Coins add to the persistent balance used in the shop.

---

## Characters (Bots)

| ID | Name | Appearance | Face style |
|----|------|------------|------------|
| `brown` | Mr Brown | Brown cube, flat mouth, tie | Boring |
| `seeky` | Seeky | Yellow cube, rosy cheeks, jetpack | Cute (squidmallow) |
| `obvious` | Mr Obvious | Half red / half blue cube | Obvious |
| `geoffrey` | Geoffrey | Tan cube with giraffe spots and ossicone horns | Giraffe |
| `lighty` | Mr Lighty | Teal cube with purple + yellow helmet | Helmet |
| `axy` | Axy | Pink axolotl shape with feathery gills | Axolotl |

---

## Shop

Accessible via the 🛒 Shop button. Coin balance persists across sessions.

### Hats (buy once, equip/unequip anytime)

| Hat | Cost | Effect |
|-----|------|--------|
| 🦆 Duck Hat | 4💰 | Coins are drawn as grapes (cosmetic only, same value) |
| 🎸 Guitar Hat | 7💰 | Every coin collected counts double (treasure gives +20) |
| 🔥 Obsidian Flame Hat | 9💰 | Trap doors have no effect |
| 👻 Invisibility Hat | 11💰 | Octopus moves randomly (can't pathfind to you). Character shown as dotted outline |
| ⚡ Zoomey Hat | 14💰 | Each arrow press moves 3 squares |
| 🪄 Magic Hat | 18💰 | Press SPACE to enter teleport mode, then arrow key to jump 10 squares through walls. Can't teleport through bone/chicken gates |

### Consumables (buy multiple, used up when activated)

| Item | Cost | Effect |
|------|------|--------|
| 🚀 Booster Rockets | 5💰 | 2 extra squares per press for 10 presses. Stacks with Zoomey hat |
| 🍎 Food | 3💰 | Freezes the octopus for 6 seconds (level 6 only) |

---

## Level features in detail

### Traps (level 3)
Two purple `?` circles placed **off** the main route. Stepping on one either warps you near the goal (lucky) or back to start (unlucky). Obsidian hat grants immunity. Magic hat can teleport past them.

### Keys and doors (level 5)
Items placed along the main path in strict order:
`key 1 → door 1 → key 2 → door 2 → goal`
Doors block movement until the correct key is collected. The second key can't be picked up before door 1 is opened.

### Octopus (level 6)
- Spawns off the main path, at least 5 cells from start
- Moves every ~0.9 seconds (every other tick of a 450ms interval)
- 40% chance of random wander; 60% chance of pathfinding toward player
- Avoids immediately backtracking when wandering
- Turns blue and stops when frozen by food
- Invisibility hat makes it wander 100% of the time
- Catching the player resets the level after 1.5 seconds

### Dino gate (level 9)
- 4 bones: 2 placed off-path, 2 placed on-path
- Stone gate at second-to-last cell on main path blocks progress
- Gate shows bone counter (`0/4`) and lifts when all collected
- Magic hat cannot teleport through the gate until bones are collected

### Treasure (level 10)
- 3 treasure chests: mix of on-path and off-path
- Each gives +10 coins on collection (+20 with Guitar hat)
- Chest changes appearance when opened

### Chickens (level 11)
- 3 chickens at ~25%, 52%, 78% along the main path
- Only the current chicken is visible; next appears after tagging previous
- Wooden fence gate blocks finish until all 3 tagged
- Magic hat cannot teleport through the gate until all chickens are tagged

---

## Technical notes

### Maze generation
Uses an iterative **recursive backtracker** (depth-first search with a stack). Guaranteed to produce a perfect maze — every cell is reachable, exactly one path between any two cells.

### Pathfinding
`bfs()` (breadth-first search) is used in two places:
1. Finding the main path from start to goal at level setup (used to place keys, doors, rose, bones, chickens correctly along the route)
2. Octopus AI — recalculated every time the octopus moves

### Item placement
`offPath()` returns all cells not on the main BFS route. Traps and cheese are placed off-path. On-path items (keys, doors, rose, bones, chickens) are placed at calculated fractions of the path length. An `avoid` array prevents items overlapping each other.

### Special gates
`canMove()` checks for dino/chicken gates before checking maze walls. This means gates block movement regardless of whether there's a maze wall there.

### Animation
A `setInterval` running every 200ms handles:
- Octopus movement and tentacle wiggle animation
- Underwater bubble animation
- Obsidian hat flame flicker
- Teleport mode sparkle rotation
- Freeze countdown display

---

## Known issues / planned work (develop branch)

- Button sizes don't scale proportionally with the larger desktop maze
- Layout is too busy at the top on desktop — needs flattening
- No left-panel layout (planned: avatar + controls on the left, info in top bar)
- Mobile requires keyboard — touch D-pad controls not yet implemented
- No player name / who's played feature
- No shared leaderboard

---

## Planned future features

- Player name entry at password screen
- "Who's played" board showing names + coin totals + highest level
- More levels (12+)
- More hats

---

*Built with vanilla HTML/CSS/JS. No frameworks, no build tools, no external dependencies.*
