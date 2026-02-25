# Golf App — Project Context

## Project Goal

A mobile-first web-based golf course decision-making game. The player makes strategic shot decisions (club, aim) and the app simulates realistic outcomes based on their handicap and actual club distances. The goal is to improve real-life course management for high handicap golfers.

---

## File Structure

| File | Purpose |
|---|---|
| `index.html` | Main 3D hole renderer and shot simulator — the only file needed to run the full game (renamed from `golf-hole.html`) |
| `golf-profile-step1.html` | Original standalone profile setup UI — superseded, retained for reference |
| `golf-poc.html` | Earlier proof-of-concept; retained for reference |
| `Course map.png` | Reference image of the real-world course layout |
| `Scorecard for yardage reference.png` | Hole yardages used for scaling and layout decisions |
| `Visualization example.png` | Visual reference for the target low-poly aesthetic |
| `original golf project conversation.txt` | Archive of early design decisions and scope discussion |
| `CONTEXT.md` | This file |

---

## What's Been Built

### `index.html` (active — single file, full game; renamed from `golf-hole.html`)

**3D scene**
- Low-poly terrain, fairway (L-shaped dogleg right), green, rough ground
- Trees lining both fairway sections, water hazard at dogleg right edge, sand bunker beside the green, flag/pin on green
- White ball on yellow tee at tee box
- Ball radius defined by constant `BALL_R = 0.16` world units (~1/25th of the 8-unit flagstick) for realistic scale; both `SphereGeometry` and landing `flightEnd.y` use this constant

**Camera system**
- Behind-ball camera (execution view) and overhead camera (strategic view), toggled by Switch View button
- On tee shot: camera automatically positioned behind ball aimed straight down the fairway centre
- After each shot: camera repositions behind new ball position, automatically aimed at the pin
- Tree occlusion culling: `updateTreeVisibility(camX, camZ, bx, bz)` hides any tree whose trunk falls within 5.5 world units of the camera→ball line segment (trees beyond the ball or behind the camera are always shown); called every time the behind-ball camera repositions

**Aim mechanic**
- Tap/click on the fairway in behind-ball view to set aim point
- Yellow aim line and ring marker appear; camera rotates to face aim direction
- Tee shot and post-shot both auto-set aim toward the next logical target (fairway / pin)

**Shot physics**
- Selected club's distance drives shot length (converted via YARDS_PER_UNIT scale: 110 world units ≈ 360 yards)
- Distance variance: ±5% random on every shot
- Handicap-based lateral dispersion: Gaussian model — sigma interpolated from anchors (+10 hcp → σ≈1.2 yd, 10 → σ≈4.8 yd, 20 → σ≈9.1 yd, 30 → σ≈15 yd); produces realistic occasional big misses
- Ball flight: parabolic arc (apex at 40% of distance), two bounces, short roll
- Bunker shots apply a 35% distance reduction to the following shot

**Player profile**
- Fullscreen profile overlay shown on first load (no localStorage profile found); skipped if profile exists
- Profile flow: Step 1 (handicap slider, +10 to 30) → Step 2 (club distances, pre-populated from handicap) → Confirmation → Play
- Edit Profile button in overhead view re-opens overlay; club panel rebuilds live after saving
- Saved to localStorage as `golf_player_profile: { handicap, clubs: [{name, distance}] }`
- Defaults if no profile: handicap 20, Driver 210 yd, 3-Wood 190 yd, 5-Iron 160 yd, 7-Iron 140 yd, 9-Iron 120 yd, PW 100 yd, SW 70 yd

**Club selection**
- Scrollable panel on left side of behind-ball view (130px wide, 52px min row height, mobile touch targets)
- Hidden in overhead view
- Selected club highlighted in yellow; distance shown in yds
- Clubs over 170 yds are greyed out and unclickable when ball is in the rough

**Swing power selector**
- Segmented control docked at the bottom of the club panel (always visible in behind-ball view)
- Four options: Full (100%), 3/4 (~78%), 1/2 (~53%), 1/4 (~28%); selected option highlighted yellow
- Distance multipliers: Full=1.0×, 3/4=0.775×, 1/2=0.525×, 1/4=0.275×
- Dispersion multipliers (tighter for partial swings): Full=1.0×, 3/4=0.80×, 1/2=0.65×, 1/4=0.55×
- When a partial swing is selected, club list shows adjusted carry estimates (e.g. "~171 yds" instead of "220 yds")
- Applied in `startFlight()` via `getSwingOpt()` — multiplies both `distMultiplier` and lateral dispersion

**Stroke counter & scoring**
- Stroke count increments on every Hit press and displayed in a centred HUD at the top of the screen
- Water hazard penalty strokes also increment the counter
- **Auto-complete on green**: when ball comes to rest on the green (`detectLie` returns `'green'`), calculated putts are added automatically: ≤8 yds = 1 putt, ≤25 yds = 2 putts, 25+ yds = 3 putts; hole completes after 1.2s delay
- Legacy hole-in check still triggers if ball stops within 5 yards of pin but not classified as green
- Hole complete state locks the Hit button and shows the post-hole summary overlay
- Par is hard-coded at 4 for the current hole

**Lie detection & penalties**
- After each shot rolls to rest, `detectLie(x, z)` classifies the ball position:
  - **Fairway** — point-in-polygon test for the L-shaped fairway shape
  - **Green** — circle test (radius 13 units) around the pin
  - **Water** — rotated ellipse test (rx 11, rz 6, rotated to match the mesh)
  - **Bunker** — axis-aligned ellipse test (rx 9, rz 6)
  - **Rough** — everything else
- Penalties applied:
  - *Rough*: clubs over 170 yds restricted in the club panel; lie banner shown
  - *Water*: +1 penalty stroke, ball respawned at last safe position (saved before each shot), lie banner shown
  - *Bunker*: `bunkerPenaltyActive` flag set; next shot distance multiplied by 0.65; flag cleared after use
- Lie status shown in a banner HUD below the score display; hidden when switching to overhead view

**Overhead view improvements**
- Distance-to-pin indicator (in yards) shown in overhead view, updates after each shot
- Ball tracked by a pulsing yellow RingGeometry marker (scales sinusoidally) visible from above
- Ring hidden in behind-ball view

**Post-hole summary overlay**
- Appears over the 3D scene (z-index 200) when the hole-complete condition is met
- Displays: strokes taken, par, score vs par (E / +1 / −1 etc.), score name (Birdie / Bogey etc.), emoji icon
- Decision-quality note based on water hazard hits and stroke count
- "Play Next Hole" button reloads the page (placeholder until more holes are built)

**Stroke flow**
- Fully playable loop: profile setup → tee shot → aim → hit → land → lie check → auto-aim at pin → repeat → hole complete → summary

---

### `golf-profile-step1.html` (superseded)
- Original standalone profile setup UI — all logic now embedded in `index.html`
- Play button wired to navigate to `index.html`

### `golf-poc.html`
- Earlier prototype; placeholder for historical reference only

---

## Key Technical Decisions

- **Hosted on GitHub Pages at `https://reid421.github.io/golf-course-game/`** — `index.html` serves as the entry point; repo is public at `https://github.com/reid421/golf-course-game`
- **Three.js via CDN, single HTML files, no build process** — keeps the project portable and easy to open on any device
- **Low-poly flat-shaded aesthetic, no textures** — consistent visual style, performant on mobile
- **Mobile-first: tap to aim, large touch targets** — primary input model is touch; mouse works as a fallback
- **Two camera views: overhead (strategic) and behind-ball (execution)** — overhead for course management decisions, behind-ball for aim execution
- **Auto-aim on every shot** — tee shot aims down fairway centre; post-shot aims at pin; player can override by tapping elsewhere
- **`const`/`let` TDZ ordering discipline** — game init variables must be fully declared before any function that references them is called; `applyProfile()` and auto-aim are deferred until after all `const` declarations
- **Canvas z-index** — Three.js canvas styled `position:absolute; top:0; left:0; z-index:0` so it sits behind the HUD (z-index 10) and profile overlay (z-index 100)
- **Player profile: handicap drives dispersion only; player inputs their own club distances** — real distances matter more than algorithm-guessed ones
- **Shot distance from club, not click position** — ball travels the selected club's distance in the aimed direction, not to wherever the player clicked
- **Lie detection uses geometric zone tests matching the visual meshes** — water and bunker ellipses use the same center/radii as the Three.js geometry; fairway uses point-in-polygon against the Shape vertices
- **`lastSafePosition` saved before every shot** — used for water hazard respawn; guarantees a valid position is always available when `onBallAtRest` runs
- **`animDur` guarded with `Math.max(..., 0.05)`** — prevents divide-by-zero if flight distance is ever zero
- **`BALL_R` constant** — single source of truth for golf ball radius (0.16 units); used for both `SphereGeometry` radius and the ball's resting Y position (`GROUND_Y + BALL_R`) so they never drift out of sync
- **Tree occlusion culling** — all tree groups stored in `treeGroups[]` at creation; `updateTreeVisibility()` uses point-to-segment projection to hide only trees between the camera and ball (not in front of the ball), keeping the foreground clear without removing any scenery trees
- **Swing power via `SWING_POWER_OPTS` + `swingPower` state** — multipliers applied in `startFlight()` via `getSwingOpt()`; partial swings also reduce lateral dispersion so shorter shots are proportionally more accurate
- **Auto-putt on green** — `calcPutts(distToPin)` converts world units to yards and returns 1/2/3 based on distance brackets; called from `onBallAtRest()` whenever `detectLie` returns `'green'`

---

## Repository & Deployment

- **GitHub repo:** `https://github.com/reid421/golf-course-game` (public)
- **Git user:** `reid421` / `reidstewart421@gmail.com`
- **GitHub CLI:** installed at `C:\Program Files\GitHub CLI\gh.exe` — not on PATH by default, invoke via full path or open a new terminal after install
- **`.gitignore` excludes:** `node_modules/`, `.env*`, `.DS_Store`, `Thumbs.db`, `desktop.ini`, `.claude/`, common editor dirs
- **`index.html`** is the deployment entry point (renamed from `golf-hole.html` for GitHub Pages compatibility)

---

## Next Steps (priority order)

1. Begin building out remaining Province Lake holes
2. Add a multi-hole scorecard / round tracker
3. Sound effects for shot, water splash, hole-in
4. Mobile polish — swipe gestures, haptic feedback on hit
