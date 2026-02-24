# Golf App — Project Context

## Project Goal

A mobile-first web-based golf course decision-making game. The player makes strategic shot decisions (club, aim) and the app simulates realistic outcomes based on their handicap and actual club distances. The goal is to improve real-life course management for high handicap golfers.

---

## File Structure

| File | Purpose |
|---|---|
| `golf-hole.html` | Main 3D hole renderer and shot simulator — the only file needed to run the full game |
| `golf-profile-step1.html` | Original standalone profile setup UI — superseded, retained for reference |
| `golf-poc.html` | Earlier proof-of-concept; retained for reference |
| `Course map.png` | Reference image of the real-world course layout |
| `Scorecard for yardage reference.png` | Hole yardages used for scaling and layout decisions |
| `Visualization example.png` | Visual reference for the target low-poly aesthetic |
| `original golf project conversation.txt` | Archive of early design decisions and scope discussion |
| `CONTEXT.md` | This file |

---

## What's Been Built

### `golf-hole.html` (active — single file, full game)

**3D scene**
- Low-poly terrain, fairway (L-shaped dogleg right), green, rough ground
- Trees lining both fairway sections, water hazard at dogleg right edge, sand bunker beside the green, flag/pin on green
- White ball on yellow tee at tee box

**Camera system**
- Behind-ball camera (execution view) and overhead camera (strategic view), toggled by Switch View button
- On tee shot: camera automatically positioned behind ball aimed straight down the fairway centre
- After each shot: camera repositions behind new ball position, automatically aimed at the pin

**Aim mechanic**
- Tap/click on the fairway in behind-ball view to set aim point
- Yellow aim line and ring marker appear; camera rotates to face aim direction
- Tee shot and post-shot both auto-set aim toward the next logical target (fairway / pin)

**Shot physics**
- Selected club's distance drives shot length (converted via YARDS_PER_UNIT scale: 110 world units ≈ 360 yards)
- Distance variance: ±5% random on every shot
- Handicap-based lateral dispersion: Gaussian model — sigma interpolated from anchors (+10 hcp → σ≈1.2 yd, 10 → σ≈4.8 yd, 20 → σ≈9.1 yd, 30 → σ≈15 yd); produces realistic occasional big misses
- Ball flight: parabolic arc (apex at 40% of distance), two bounces, short roll

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

**Stroke flow**
- Fully playable loop: profile setup → tee shot → aim → hit → land → auto-aim at pin → repeat

**Not yet implemented**
- Stroke counter and scoring display
- Lie detection (rough, water, bunker penalties and messaging)
- Overhead view improvements (distance to pin, ball position indicator)
- Post-hole summary screen
- Putting mechanics for on-green play

---

### `golf-profile-step1.html` (superseded)
- Original standalone profile setup UI — all logic now embedded in `golf-hole.html`
- Play button still shows placeholder alert; not wired to game

### `golf-poc.html`
- Earlier prototype; placeholder for historical reference only

---

## Key Technical Decisions

- **Three.js via CDN, single HTML files, no build process** — keeps the project portable and easy to open on any device
- **Low-poly flat-shaded aesthetic, no textures** — consistent visual style, performant on mobile
- **Mobile-first: tap to aim, large touch targets** — primary input model is touch; mouse works as a fallback
- **Two camera views: overhead (strategic) and behind-ball (execution)** — overhead for course management decisions, behind-ball for aim execution
- **Auto-aim on every shot** — tee shot aims down fairway centre; post-shot aims at pin; player can override by tapping elsewhere
- **`const`/`let` TDZ ordering discipline** — game init variables must be fully declared before any function that references them is called; `applyProfile()` and auto-aim are deferred until after all `const` declarations
- **Canvas z-index** — Three.js canvas styled `position:absolute; top:0; left:0; z-index:0` so it sits behind the HUD (z-index 10) and profile overlay (z-index 100)
- **Player profile: handicap drives dispersion only; player inputs their own club distances** — real distances matter more than algorithm-guessed ones
- **Shot distance from club, not click position** — ball travels the selected club's distance in the aimed direction, not to wherever the player clicked

---

## Next Steps (priority order)

1. Add stroke counter and scoring display (HUD element, updates each shot)
2. Add lie detection — when ball lands in rough, water, or bunker, apply appropriate penalties and messaging
3. Overhead view improvements — distance to pin indicator, show ball position clearly
4. Wire Play button in `golf-profile-step1.html` to launch `golf-hole.html`
5. Add post-hole summary screen showing strokes taken and decision feedback
6. Begin building out remaining Province Lake holes
