# BrotherDoge v1.2 UI, Mobile, Desktop, and Polish Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task after Mike approves execution. Do not make one giant edit; each phase should be implemented, verified locally, then committed.

**Goal:** Make BrotherDoge feel good on phones and desktop by fixing oversized shop/breed-select screens, enlarging and beautifying the desktop arena, and adding the gameplay/retention/polish features previously proposed.

**Architecture:** Keep the project as a self-contained browser game in `BrotherDoge.html` and mirrored `index.html`. Refactor only enough to support responsive layout, viewport scaling, camera transforms, UI screens, persistence, and polish without converting to a build system. Use small helper functions for repeated UI/render/input logic.

**Tech Stack:** Single-file HTML5 Canvas + CSS + vanilla JavaScript, Web Audio API, `localStorage`, GitHub Pages.

---

## Current Context

Repository: `/tmp/brotherdoge-work`

Current pushed commit: `0b2f0f8 Fix mobile arena scaling`

Known current state:
- Arena is now fully visible on phone by CSS-scaling the `800x600` canvas.
- Doge select and shop screens are still too large on phone.
- Desktop arena now looks too small because the canvas is capped at 1x scale.
- Arena visual style is functional but dull: dark grid, simple shapes.
- Current project has two mirrored game files:
  - `BrotherDoge.html`
  - `index.html`
- Any edit to one must be copied to the other before commit.

Primary problems to fix first:
1. Breed select screen does not fit on phones.
2. Shop screen does not fit on phones.
3. Desktop gameplay canvas should scale up and feel larger.
4. Arena background needs visual richness without hurting readability.

Secondary feature list to implement after UI foundation:
- Portrait warning.
- Real visual mobile joysticks.
- Mobile follow-camera mode.
- Settings menu.
- Tutorial sequence: move -> shoot -> collect coin.
- Lifetime stats screen.
- Achievements/unlocks.
- Better shop recommendations/comparison.
- Boss phase/readability improvements.
- Earlier enemy-introduction callouts.
- Juice/feel improvements.
- Generated Web Audio music.
- Shareable run summaries.
- Seeded/daily runs.
- README/social presentation upgrades.
- Local leaderboard, with future online leaderboard deferred.

---

## Guiding Decisions

### Responsive UI strategy

Do NOT try to solve mobile screens by simply scaling the entire DOM with `transform: scale(...)`. That made the game visible but will make menus awkward.

Use screen-specific layouts:
- Breed select: scrollable responsive card grid.
- Shop: scrollable panels/tabs on phones, 3-column desktop layout.
- End screens: scrollable overlay with max-height and compact summaries.

### Desktop arena strategy

Current `resizeCanvas()` uses:
- internal canvas = arena size
- CSS display size = min(view / arena, 1)

For desktop, allow upscale:
- max scale: `1.35` or `1.5`
- phone/tablet: keep fit-to-screen scale
- desktop: scale arena up while preserving coordinate mapping

### Camera strategy

Keep full-arena rendering as default for desktop.

For phones, add an optional follow-camera mode:
- Internal arena stays `800x600` / boss `1100x800`.
- Canvas still fits phone viewport.
- Render world through camera transform so sprites are larger.
- UI remains screen-space DOM/overlay.
- Offscreen enemy indicators become more important.

Implement camera mode after the menu fixes so we do not mix layout and world-render refactors in one patch.

---

# Phase 1: Fix Phone Breed Select Screen

## Task 1.1: Make overlay screens scroll safely

**Objective:** Ensure large overlays can scroll vertically on small screens instead of clipping.

**Files:**
- Modify: `BrotherDoge.html`
- Mirror: `index.html`

**Implementation notes:**

Update base overlay containers:
- `#breedSelect`
- `#shopOverlay`
- `#gameOver`
- `#victory`
- `#pauseOverlay`

Add these principles:
```css
overflow-y: auto;
overflow-x: hidden;
-webkit-overflow-scrolling: touch;
max-height: 100dvh;
```

For full-screen overlays, keep `position:absolute; inset:0;` but allow content wrapper scrolling.

**Verification:**
- Open in desktop browser.
- Simulate narrow viewport mentally with browser console or resize.
- Confirm no overlay content is inaccessible.
- Confirm game canvas itself remains non-scrollable during play.

---

## Task 1.2: Convert Doge select to responsive grid

**Objective:** Make all Doge cards usable on phones without requiring impossible horizontal space.

**Files:**
- Modify CSS in `BrotherDoge.html`
- Mirror to `index.html`

**Current problem:** `.breedOptions` uses cards that are too wide for phone portrait.

**Change:**

Desktop:
```css
.breedOptions {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(190px, 1fr));
  gap: 14px;
  width: min(1100px, 96vw);
}
.breedCard { width: auto; }
```

Mobile:
```css
@media (max-width: 700px), (max-height: 500px) {
  #breedSelect {
    justify-content: flex-start;
    padding: 12px 8px 24px;
  }
  #breedSelect h1 {
    font-size: 24px;
    margin: 8px 0 10px;
    position: sticky;
    top: 0;
    z-index: 2;
    background: rgba(10,10,30,0.9);
    padding: 8px;
    border-radius: 8px;
  }
  .breedOptions {
    grid-template-columns: 1fr;
    width: min(420px, 94vw);
    gap: 10px;
  }
  .breedCard {
    width: 100%;
    min-height: unset;
    padding: 10px;
  }
}
```

**Verification:**
- In browser console, check a simulated 390px-wide layout is one card wide.
- Confirm all five Doges are reachable by vertical scroll.
- Confirm cards still look good on desktop.

---

## Task 1.3: Add compact Doge cards for mobile

**Objective:** Reduce visual height of each Doge card while preserving the important choice info.

**Files:**
- Modify CSS/HTML in `BrotherDoge.html`
- Mirror to `index.html`

**Change:**
- Keep name and role visible.
- Make flavor quote smaller or hide it on very small screens.
- Keep stats but reduce line height.
- Keep perk line.

Mobile CSS:
```css
@media (max-width: 700px) {
  .breedCard .stats em { display: none; }
  .breedCard .stats { line-height: 1.35; }
  .breedCard .perk { margin-top: 6px; padding-top: 6px; }
}
```

**Verification:**
- On phone portrait, user should see at least one complete card and part of the next.
- No text should overflow off-card.

---

# Phase 2: Fix Phone Shop Screen

## Task 2.1: Replace mobile shop 3-column layout with tabbed shop

**Objective:** Make shop usable on phones by showing one section at a time.

**Files:**
- Modify HTML/CSS/JS in `BrotherDoge.html`
- Mirror to `index.html`

**Current problem:** `#shopContent` contains Weapons, Perks, Upgrades, Stats side-by-side and is too wide.

**Approach:**
- Desktop: keep multi-column layout.
- Mobile: show tab buttons at top:
  - Weapons
  - Perks
  - Upgrades
  - Stats
- Only active shop section is displayed on mobile.

**HTML addition inside `#shopOverlay`, before `#shopContent`:**
```html
<div id="shopTabs">
  <button onclick="setShopTab('weapons')">Weapons</button>
  <button onclick="setShopTab('perks')">Perks</button>
  <button onclick="setShopTab('upgrades')">Upgrades</button>
  <button onclick="setShopTab('stats')">Stats</button>
</div>
```

**JS state:**
```js
let activeShopTab = 'weapons';
function setShopTab(tab) {
  activeShopTab = tab;
  document.querySelectorAll('.shopSection').forEach(section => {
    section.classList.toggle('mobile-active', section.dataset.shopTab === tab);
  });
  document.querySelectorAll('#shopTabs button').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.shopTab === tab);
  });
}
```

Add `data-shop-tab="weapons"`, etc. to sections.

In `openShop()` and `renderShop()`, call `setShopTab(activeShopTab || 'weapons')` after rendering.

**CSS:**
```css
#shopTabs { display:none; }
@media (max-width: 700px), (max-height: 500px) {
  #shopTabs { display:flex; gap:6px; flex-wrap:wrap; margin-bottom:8px; }
  #shopTabs button.active { background:#ffd700; color:#000; }
  #shopContent { display:block; width:min(430px,94vw); transform:none; }
  .shopSection { display:none; min-width:0; width:100%; max-height:52dvh; overflow-y:auto; }
  .shopSection.mobile-active { display:block; }
}
```

**Verification:**
- On mobile width, only one shop column visible at a time.
- Tabs switch sections.
- Buy/lock/upgrade buttons still work.
- Desktop still shows all sections side-by-side.

---

## Task 2.2: Make shop buttons sticky and reachable

**Objective:** Ensure Refresh and Continue buttons are always reachable on phone.

**Files:**
- Modify CSS in both HTML files.

**Change:**
Wrap bottom shop controls or style existing buttons:
```css
@media (max-width: 700px) {
  #refreshShop, #continueBtn {
    position: sticky;
    bottom: 8px;
    z-index: 5;
    font-size: 14px;
    padding: 10px 14px;
    margin: 4px;
  }
}
```

Better version: create a `#shopActions` wrapper around both buttons.

**Verification:**
- User can continue from shop without scrolling to impossible positions.
- Buttons do not cover shop card content too badly.

---

## Task 2.3: Compact shop cards on mobile

**Objective:** Fit multiple shop offers per phone screen.

**Files:**
- Modify CSS in both HTML files.

Mobile card changes:
```css
@media (max-width: 700px) {
  .shopItemHeader .name { font-size: 13px; }
  .shopItem .desc { font-size: 11px; }
  .shopItem .price { font-size: 13px; }
  .lock-btn { min-width: 34px; min-height: 30px; }
}
```

**Verification:**
- Weapon/perk names readable.
- Lock button tappable.
- At least 2 cards visible on a typical phone portrait screen.

---

# Phase 3: Make Desktop Arena Larger and Better Looking

## Task 3.1: Allow desktop canvas upscaling

**Objective:** Make arena fill more of desktop screens while preserving whole arena visibility.

**Files:**
- Modify `resizeCanvas()` in both HTML files.

**Current behavior:** scale is capped at `1`.

**New behavior:**
```js
function getMaxCanvasScale() {
  const smallScreen = window.innerWidth < 760 || window.innerHeight < 540;
  if (smallScreen) return 1;
  if (window.innerWidth >= 1400 && window.innerHeight >= 900) return 1.45;
  return 1.25;
}

const scale = Math.min(viewW / ARENA_WIDTH, viewH / ARENA_HEIGHT, getMaxCanvasScale());
```

Important: pointer mapping through `clientToArena()` already accounts for display scale, so input should still work.

**Verification:**
- Desktop 1280x720 should display canvas around 960x720 if scale allows height fit.
- Desktop 1920x1080 should display around 1440x1080 or capped at 1.45.
- Phone should still fit whole arena.

---

## Task 3.2: Add arena background layers

**Objective:** Make arena less dull without obscuring gameplay.

**Files:**
- Modify `render()` in both HTML files.

**Add helper:**
```js
function drawArenaBackground() {
  const grad = ctx.createRadialGradient(ARENA_WIDTH/2, ARENA_HEIGHT/2, 40, ARENA_WIDTH/2, ARENA_HEIGHT/2, Math.max(ARENA_WIDTH, ARENA_HEIGHT));
  grad.addColorStop(0, '#17173a');
  grad.addColorStop(0.55, '#0f0f24');
  grad.addColorStop(1, '#070713');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, ARENA_WIDTH, ARENA_HEIGHT);

  // subtle Dogecoin moon/circuit grid
  // draw large translucent circles, diagonal market lines, and small sparkles
}
```

Replace the flat clear:
```js
ctx.fillStyle = '#0f0f1e';
ctx.fillRect(0, 0, canvas.width, canvas.height);
```
with `drawArenaBackground()`.

**Visual style:**
- Warm heroic arcade, not grimdark.
- Dogecoin gold accents.
- Subtle candlestick/market-chart ghost lines in background.
- Slight vignette.
- Keep enemies/projectiles readable.

**Verification:**
- Player/enemies still pop.
- Boss danger zones still readable.
- No performance stutter.

---

## Task 3.3: Add environmental props/parallax details

**Objective:** Add richness without asset files.

**Files:**
- Modify render helpers in both HTML files.

Add decorative elements:
- faint Doge paw prints on floor
- scattered paper scraps near edges
- gold glow border pulses on wave clear
- subtle stars/sparks in background

Use deterministic pseudo-random based on coordinates so it does not jitter every frame.

**Verification:**
- Background details are stable, not flickering.
- Performance remains smooth.

---

# Phase 4: Mobile Feel Patch

## Task 4.1: Add portrait warning

**Objective:** Tell players landscape is better while still allowing portrait play.

**Files:**
- Add DOM element and CSS/JS in both HTML files.

**HTML:**
```html
<div id="rotateHint">📱 Rotate phone sideways for bigger battle view</div>
```

**JS:**
```js
function updateRotateHint() {
  const isPhone = window.innerWidth < 760;
  const portrait = window.innerHeight > window.innerWidth;
  document.getElementById('rotateHint').style.display = isPhone && portrait && gameState === 'playing' ? 'block' : 'none';
}
```
Call in `resizeCanvas()` and `updateUI()`.

**Verification:**
- Portrait phone shows hint during play.
- Landscape and desktop hide it.

---

## Task 4.2: Draw visual mobile joysticks

**Objective:** Make touch controls discoverable.

**Files:**
- Modify render or DOM overlay in both HTML files.

**Approach:** Draw joysticks on canvas only when touch is active or likely mobile.

Add helper:
```js
function drawTouchControls() {
  if (!isTouchDevice()) return;
  // left base circle, left thumb circle
  // right aim/fire circle
}
```

Call near end of `render()` after world drawing but before overlays if appropriate.

**Verification:**
- On mobile/touch simulation, joystick visuals appear.
- They do not cover important HUD elements.

---

## Task 4.3: Add mobile follow-camera mode

**Objective:** Make arena appear larger on phones while preserving awareness.

**Files:**
- Modify render coordinate system in both HTML files.

**State:**
```js
let cameraMode = 'auto'; // auto, full, follow
let camera = { x:0, y:0, scale:1 };
```

**Logic:**
```js
function updateCamera() {
  const smallScreen = window.innerWidth < 760 || window.innerHeight < 540;
  const useFollow = cameraMode === 'follow' || (cameraMode === 'auto' && smallScreen && gameState === 'playing');
  if (!useFollow) { camera.x = 0; camera.y = 0; camera.scale = 1; return; }
  camera.scale = boss ? 1.15 : 1.45;
  const viewW = canvas.width / camera.scale;
  const viewH = canvas.height / camera.scale;
  camera.x = clamp(player.x - viewW/2, 0, ARENA_WIDTH - viewW);
  camera.y = clamp(player.y - viewH/2, 0, ARENA_HEIGHT - viewH);
}
```

Wrap world draw:
```js
ctx.save();
ctx.scale(camera.scale, camera.scale);
ctx.translate(-camera.x, -camera.y);
// world draw
ctx.restore();
// screen-space HUD/canvas overlays after restore
```

Need to adjust pointer-to-arena aim when camera is active:
```js
x = camera.x + screenX / camera.scale;
y = camera.y + screenY / camera.scale;
```

**Verification:**
- On mobile, player/enemies are larger.
- Mouse/touch aiming still hits tutorial target.
- Offscreen enemy indicators work or are updated to camera view.
- Boss mode zooms out enough.

---

# Phase 5: Settings Menu

## Task 5.1: Add settings state and UI

**Objective:** Give players toggles for mobile and accessibility features.

**Files:**
- Modify pause overlay in both HTML files.

Settings:
- Sound: on/off
- Auto-fire: on/off
- Camera: auto/full/follow
- Screen shake: low/normal/high
- Damage numbers: on/off
- Music: on/off
- Reset saved progress

Persist in `localStorage` under existing progress key or a `settings` object.

**Verification:**
- Settings persist after reload.
- Reset progress asks for confirmation.
- Pause menu still fits mobile.

---

# Phase 6: Better Tutorial Sequence

## Task 6.1: Replace one-step tutorial with staged tutorial

**Objective:** Teach move, shoot, collect before Wave 1.

**Files:**
- Modify tutorial logic in both HTML files.

States:
```js
let tutorialStep = 'move'; // move, shoot, collect, done
let tutorialCoin = null;
```

Flow:
1. Spawn glowing circle near player.
2. Player moves into circle.
3. Spawn unmoving Paper Hand.
4. Player kills it.
5. Spawn coin.
6. Player collects coin.
7. Start Wave 1.

**Verification:**
- Tutorial cannot soft-lock.
- Touch, mouse, gamepad all work.
- Skips should not be necessary.

---

# Phase 7: Persistent Stats, Achievements, and Local Leaderboard

## Task 7.1: Extend progress schema

**Objective:** Save lifetime stats and achievements.

**Files:**
- Modify progress functions in both HTML files.

Add:
```js
lifetime: {
  kills: 0,
  coins: 0,
  damage: 0,
  bossKills: 0,
  runs: 0,
  victories: 0,
  favoriteBreedCounts: {},
  weaponUseCounts: {}
},
achievements: {}
```

Migration: existing saves should still load.

**Verification:**
- Old save does not crash.
- New fields appear after one run.

---

## Task 7.2: Add achievements

**Objective:** Make persistent progress rewarding.

Initial achievements:
- First Blood: kill 1 Paper Hand
- Diamond Paws: survive/clear wave with <= 1 HP or very low HP
- Whale Hunter: beat boss
- Such Economy: collect 500 coins in one run
- No Paper Hands: reach Wave 5 without dropping below 50% HP
- Degen Confirmed: unlock Degen Mode
- Much Violence: deal 100k lifetime damage

Add `unlockAchievement(key)` helper with floating toast.

**Verification:**
- Achievements save.
- Toast appears once.
- Stats screen lists unlocked achievements.

---

## Task 7.3: Add Stats/Achievements screen on breed select

**Objective:** Let players see persistent progress.

**Files:**
- Modify breed select HTML/CSS/JS in both HTML files.

Add buttons:
- Stats
- Achievements
- Reset Progress

Use modal overlay or a section below cards.

**Verification:**
- Screen fits mobile.
- Stats are readable.
- Reset progress works with confirmation.

---

## Task 7.4: Add local leaderboard

**Objective:** Track top local runs without online backend.

**Files:**
- Modify progress/run summary in both HTML files.

Save top 10 runs by difficulty/kills/coins.

**Verification:**
- Death/victory adds entry.
- Stats screen shows top runs.

---

# Phase 8: Shop Intelligence

## Task 8.1: Add shop recommendation labels

**Objective:** Help players understand build synergy.

**Files:**
- Modify `renderShop()`, add helper functions.

Add helpers:
```js
function getPerkRecommendation(perk) { ... }
function getWeaponRecommendation(weapon) { ... }
```

Examples:
- Lifesteal recommended if high fire rate / shotgun / low HP build.
- Magnet/economy recommended early.
- Crit recommended with high damage/shotgun.
- Dodge recommended for Phantom/Streets.

**Verification:**
- Labels are helpful, not spammy.
- Mobile cards still fit.

---

## Task 8.2: Improve comparison display

**Objective:** Make buy decisions clearer.

Show:
- DPS
- Range
- Trait tags: `Pierce`, `Bounce`, `Explosive`, `Homing`, `Knockback`, `Slow`
- Up/down vs current best

**Verification:**
- No broken HTML injection.
- Cards remain readable on mobile.

---

# Phase 9: Boss and Enemy Readability

## Task 9.1: Boss phase labels

**Objective:** Make boss fight legible and exciting.

Phase names:
- Phase 1: Paper Hands
- Phase 2: Market Panic
- Phase 3: Final Sell-Off

Add phase toast when thresholds are crossed.

**Verification:**
- Toast appears once per threshold.
- Boss HP bar includes phase label.

---

## Task 9.2: Distinct boss attack warnings

**Objective:** Every boss attack should look different before it hurts.

Visual mapping:
- Red circle: MARKET CRASH slam
- Orange circles: LIQUIDATION delayed explosions
- White spiral: PAPER STORM
- Purple flash: HOSTILE TAKEOVER teleport
- Speed streak: PUMP & DUMP charge

**Verification:**
- Player can understand danger before damage.
- No excessive screen clutter.

---

## Task 9.3: Enemy intro callouts

**Objective:** Teach new enemies as they appear.

When first enemy type appears:
```text
NEW ENEMY: SHREDDER
Fast but fragile. Kite sideways.
```

Persist only per run, not forever.

**Verification:**
- Appears once per run per enemy type.
- Does not block controls.

---

# Phase 10: Juice and Feel

## Task 10.1: Crit and hit juice

**Objective:** Make combat satisfying.

Add:
- bigger crit sparks
- small hit pause on crit/big hit
- coin pickup trail sparkle
- low-health red edge pulse
- visible `DODGE!` on every dodge

**Verification:**
- Game remains responsive.
- Screen shake can be reduced in settings.

---

## Task 10.2: Wave clear and shop transition polish

**Objective:** Make loop feel polished.

Add:
- animated `WAVE CLEAR` banner
- coin vacuum finish
- shop card pop-in
- purchase flash

**Verification:**
- Shop still opens reliably.
- No timing soft-lock.

---

# Phase 11: Music

## Task 11.1: Add Web Audio music manager

**Objective:** Add simple generated music without asset files.

Music modes:
- menu
- wave
- boss
- victory
- death

Add settings toggle `musicEnabled`.

**Implementation:**
Use oscillators/gain nodes, scheduled loops, and stop/transition helpers.

**Verification:**
- Browser autoplay restrictions respected: start after user gesture only.
- Sound toggle and music toggle work separately.
- No runaway audio nodes.

---

# Phase 12: Shareable Runs and Seeded Runs

## Task 12.1: Add copy run summary button

**Objective:** Make runs shareable.

Add button to death/victory:
```text
Copy Run Summary
```

Copies:
```text
HODL OR DIE RUN
Doge: Lassy
Wave: 10
Difficulty: 2
Kills: 386
Weapon: Wow Shotgun +3
Perks: 14
Seed: DOGE-7K2M
```

**Verification:**
- Clipboard works on HTTPS Pages.
- Fallback selects text if clipboard unavailable.

---

## Task 12.2: Add seeded RNG foundation

**Objective:** Prepare daily/seeded runs.

Add seeded RNG helper, but do not replace every `Math.random()` in one massive step.

Approach:
- Add `rng()` wrapper.
- Start with shop rolls and wave spawns.
- Later migrate combat decorations if needed.

**Verification:**
- Same seed gives same shop offers and wave spawns.
- Normal random runs remain default.

---

## Task 12.3: Add Daily Run mode

**Objective:** Add replayable community challenge.

Daily seed formula:
```js
const dailySeed = 'DOGE-' + new Date().toISOString().slice(0,10);
```

Add button on breed select:
- Normal Run
- Daily Run
- Seeded Run

**Verification:**
- Daily run uses same seed all day.
- Run summary identifies daily/seed.

---

# Phase 13: README and Presentation

## Task 13.1: Add screenshot/GIF assets

**Objective:** Make repo look public-ready.

Files:
- Create `assets/screenshots/arena.png`
- Create `assets/screenshots/shop.png`
- Optionally create short GIF later.

Update README with images.

**Verification:**
- Images render on GitHub README.
- File sizes reasonable.

---

## Task 13.2: Add changelog/release notes

**Objective:** Track visible progress.

Create:
- `CHANGELOG.md`

Add entries:
- v1.1 onboarding/progress/pages
- v1.2 responsive UI/mobile/desktop polish

**Verification:**
- README links changelog.

---

# Phase 14: Future Online Leaderboard (Deferred)

Do NOT implement online leaderboard until local leaderboard and seeded runs are stable.

Possible future stack:
- Firebase anonymous auth + RTDB
- Supabase table with simple row insert
- GitHub Gist manually curated leaderboard

Risks:
- Abuse/cheating because client-side game has no authority.
- Adds setup friction and external dependency.

Recommended first step: local leaderboard only.

---

# Implementation Order and Commits

Recommended commit sequence:

1. `fix: make breed select responsive on mobile`
2. `fix: make shop usable on mobile`
3. `feat: scale desktop arena larger`
4. `style: improve arena background visuals`
5. `feat: add mobile rotate hint and touch joysticks`
6. `feat: add camera settings and mobile follow camera`
7. `feat: add settings menu`
8. `feat: expand tutorial flow`
9. `feat: add lifetime stats and achievements`
10. `feat: add local leaderboard`
11. `feat: improve shop recommendations`
12. `feat: improve boss phases and enemy callouts`
13. `feat: add combat juice and generated music`
14. `feat: add shareable summaries and seeded runs`
15. `docs: update README screenshots and changelog`

Each commit should be pushed and checked on GitHub Pages before continuing.

---

# Validation Checklist

Run after each phase:

```bash
cd /tmp/brotherdoge-work
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
class ScriptExtractor(HTMLParser):
    def __init__(self): super().__init__(); self.in_script=False; self.parts=[]
    def handle_starttag(self, tag, attrs):
        if tag.lower()=='script': self.in_script=True
    def handle_endtag(self, tag):
        if tag.lower()=='script': self.in_script=False
    def handle_data(self, data):
        if self.in_script: self.parts.append(data)
for fname in ['BrotherDoge.html','index.html']:
    parser=ScriptExtractor(); parser.feed(Path(fname).read_text(encoding='utf-8'))
    Path(f'/tmp/{fname}.js').write_text('\n'.join(parser.parts), encoding='utf-8')
PY
node --check /tmp/BrotherDoge.html.js
node --check /tmp/index.html.js
```

Manual browser checks:
- Desktop 1280x720:
  - Arena larger than current.
  - Breed select readable.
  - Shop desktop layout intact.
- Phone portrait ~390x844:
  - Breed select scrolls and cards fit.
  - Shop tabs work.
  - Continue button reachable.
  - Arena visible and playable.
- Phone landscape ~844x390:
  - Arena is bigger and playable.
  - HUD does not cover too much.
- Tutorial:
  - move -> shoot -> collect -> Wave 1 starts.
- Persistence:
  - stats/achievements survive reload.
- Pages:
  - `https://123mikeyd.github.io/Brother_Doge_VideoGame/` serves newest commit.

---

# Risks and Tradeoffs

1. **Camera mode can break aiming.**
   - Mitigation: implement after UI fixes; adjust `clientToArena()` to account for camera transform; test tutorial target hit.

2. **Shop tabs may break existing `renderShop()` assumptions.**
   - Mitigation: preserve existing DOM IDs (`weaponShop`, `perkShop`, `upgradeShop`, `shopStats`) and only change parent layout.

3. **Desktop upscaling may blur canvas.**
   - Mitigation: acceptable for canvas arcade; if too blurry, increase actual canvas backing resolution with devicePixelRatio later.

4. **Too many features in one patch increases regression risk.**
   - Mitigation: implement in phases and push/check after each commit.

5. **Mobile browser viewport changes when address bar hides.**
   - Mitigation: use `100dvh`, call `resizeCanvas()` on `resize` and `orientationchange`.

6. **Token security.**
   - The GitHub token pasted in chat should be revoked/rotated. Use `gh auth login` or a fresh token for future pushes.

---

# Definition of Done for v1.2

v1.2 is done when:
- Doge select fits and scrolls on phone portrait.
- Shop is usable on phone portrait via tabs or equivalent compact layout.
- Desktop arena is visibly larger than current GitHub Pages build.
- Arena background looks richer but gameplay remains readable.
- Touch controls have visible joysticks.
- Portrait warning exists.
- Settings menu exists.
- Tutorial teaches movement, shooting, and coin pickup.
- Lifetime stats and achievements exist.
- Local leaderboard exists.
- Boss/enemy readability upgrades exist.
- Run summaries are shareable.
- README has screenshots/changelog.
- GitHub Pages is verified live after push.
