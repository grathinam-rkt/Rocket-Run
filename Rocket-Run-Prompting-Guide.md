# Prompting Guide: Build a Super Mario-Style Game That Runs in the Browser

## 1. Purpose and outcome

This guide shows how to prompt an AI assistant (such as Claude) to build a Mario-style platformer that:

- Is a **single `index.html` file** (HTML5 Canvas + vanilla JavaScript).
- **Runs by double-clicking the file.** No installation, build step, server, or libraries.
- Uses **no external assets**. Graphics are drawn with shapes or pixel arrays, and sound comes from the Web Audio API.

> **Result:** you save the AI's output as `index.html`, open it in Chrome, Edge, or Firefox, and play.

---

## 2. Prompting principles

| Principle | Why it matters | Example |
|---|---|---|
| **State constraints first** | The AI picks the wrong stack if you don't. | "Single HTML file, no libraries, no external images." |
| **Build in stages** | One giant prompt produces buggy, hard-to-fix code. | Movement, then collision, then enemies, then sound. |
| **Use numbers, not adjectives** | "Floaty jump" is subjective. | "Jump height 4 tiles, gravity 0.6 px/frame²." |
| **Define "done"** | Gives you a checklist to test against. | "Player can't walk through walls or fall through floors." |
| **Ask for the full file each time** | Avoids broken partial edits. | "Return the complete updated `index.html`." |
| **Report bugs precisely** | The AI can then fix the right thing. | "Expected X, got Y, steps to reproduce." |

---

## 3. Master prompt template

Copy this and fill in the `[brackets]`.

```text
Build a Super Mario-style 2D platformer game as ONE self-contained index.html file.

TECH CONSTRAINTS
- HTML5 Canvas 2D + vanilla JavaScript only. No libraries, frameworks, or CDN links.
- No external images or audio files. Draw all graphics with canvas shapes or
  small pixel-art arrays. Generate sound with the Web Audio API.
- Must work by double-clicking the file in Chrome, Edge, or Firefox.
- Use requestAnimationFrame with delta-time so speed is the same on any monitor.
- No console errors or warnings.

CORE MECHANICS
- Player: run, variable-height jump (hold to jump higher), gravity, acceleration
  and friction. Optional run button for extra speed.
- Tile-based world with solid collision on all four sides.
- Enemies: [Goomba-style walkers, Koopa-style shell enemies]. Stomp from above
  to defeat them, and touching them from the side costs a life.
- Collectibles: coins, question blocks that release coins or a power-up.
- Power-up: [mushroom grows the player, which absorbs one hit].
- Goal: flagpole at the end of the level, then show a level-complete screen.

CONTROLS
- Arrow keys or WASD to move, Space or W to jump, Shift to run.
- On-screen touch buttons on mobile.

GAME FEEL
- Coyote time of [100] ms and jump buffering of [100] ms.
- Camera follows the player horizontally and clamps to level bounds.
- Jump height of about [4] tiles, max run speed of [6] tiles per second.

LEVEL
- Define the level as an array of strings (one character per tile):
  [paste or describe level 1: ground, pits, pipes, brick rows, question blocks, enemies]

HUD AND STATES
- HUD: score, coins, lives, time remaining, world number.
- States: title screen, playing, paused (P key), game over, level complete.

OUTPUT
- Return the complete index.html in one code block.
- After the code, give a 5-line summary of how to play and the tunable constants.
```

---

## 4. Iterative prompt sequence

Use these one at a time. Test each stage before moving on.

### Stage 1: Skeleton and movement
```text
Create a single index.html with a canvas, a fixed-timestep game loop, and a
rectangle player that runs left and right with acceleration and friction and
jumps with gravity (hold to jump higher). Draw a flat ground line. Show FPS in
the corner. Return the full file.
```
**Done when:** movement feels smooth, the jump is variable-height, and FPS is steady.

### Stage 2: Tile map and collision
```text
Add a tile map defined as an array of strings. Tiles: '#' ground, 'B' brick,
'?' question block, 'P' pipe, ' ' empty. Add axis-separated AABB collision so the
player cannot pass through solid tiles. Add a horizontally scrolling camera.
Create a 100-tile-wide level with two pits. Return the full file.
```
**Done when:** the player lands on platforms, bumps their head on bricks, can't clip through walls, and falls into pits.

### Stage 3: Enemies
```text
Add Goomba-style enemies that walk, turn around at walls and ledges, and are
squashed when the player lands on them (small bounce). Side contact kills the
player and respawns them at the start with one fewer life. Return the full file.
```
**Done when:** stomping works every time and side contact is distinguishable from stomping.

### Stage 4: Items, blocks, and scoring
```text
Add coins, question blocks that pop out a coin or a mushroom when hit from below
(with a small bump animation), brick blocks that break when the player is
powered up, and score popups. Mushroom grows the player and absorbs one hit.
Return the full file.
```
**Done when:** every block type behaves correctly and the score and coin counter update.

### Stage 5: HUD and game states
```text
Add a HUD (score, coins, lives, timer, world), a title screen, pause (P),
game over, and a flagpole at the end of the level that triggers a
level-complete screen with a time bonus. Return the full file.
```
**Done when:** you can play from title to win, and from title to game over, without reloading.

### Stage 6: Sound
```text
Add sound effects using the Web Audio API only (jump, coin, stomp, power-up,
death, level complete) and a simple looping background melody. Add an M key to
mute. Audio must start only after the first user input to satisfy browser
autoplay rules. Return the full file.
```
**Done when:** all sounds play, there's no audio before the first keypress, and mute works.

### Stage 7: Multiple levels and mobile controls
```text
Add three levels in a levels[] array with increasing difficulty, and on-screen
touch controls (left, right, jump, run) that only appear on touch devices.
Scale the canvas to fit the window while keeping the aspect ratio and crisp
pixels. Return the full file.
```
**Done when:** levels advance in order and the game is playable on a phone.

### Stage 8: Polish and performance
```text
Review the code for bugs and performance. Add squash-and-stretch on jump and
land, particles for brick breaks, a death animation, and save the high score in
localStorage (wrapped in try/catch). Remove any unused code. Return the full file.
```
**Done when:** the game holds 60 FPS and the high score persists.

---

## 5. Refinement and debugging prompts

| Problem | Prompt |
|---|---|
| Player falls through floor | "The player sometimes falls through the ground at high speed. Fix collision by resolving X and Y separately and clamping the max fall speed to [N] px per frame." |
| Jump feels floaty | "The jump is too floaty. Increase gravity by 30% when falling, and cut the jump short when the key is released." |
| Enemies don't turn around | "Enemies walk off ledges and through walls. Make them reverse direction on wall contact and when no ground is ahead." |
| Game too fast on a high-refresh monitor | "Game speed depends on the monitor. Use delta-time in the loop and cap dt at 50 ms." |
| No sound | "Sound doesn't play. Create the AudioContext on the first keydown or click and call resume()." |
| Blurry graphics | "Disable image smoothing and scale the canvas using integer scale factors." |

**Bug report format** (use this every time):

```text
BUG: [one-line summary]
EXPECTED: [what should happen]
ACTUAL: [what happens instead]
STEPS: [1. ... 2. ... 3. ...]
BROWSER: [Chrome / Edge / Firefox + version]
CONSOLE ERROR: [paste if any]
```

---

## 6. Do and don't

| Do | Don't |
|---|---|
| Specify numbers (speed, jump height, timing). | Say "make it feel nice". |
| Ask for the complete file every time. | Accept partial snippets you have to merge by hand. |
| Test after every stage. | Stack five features in one prompt. |
| Name the constraints up front (single file, no libraries). | Let the AI choose the stack. |
| Keep tunable constants at the top of the file. | Bury magic numbers in the code. |
| Ask the AI to explain a bug before fixing it when it's unclear. | Re-paste the same prompt hoping for a different result. |

---

## 7. How to run and share

1. Copy the code block the AI returns into a new text file.
2. Save it as `index.html`. On Windows, make sure the extension is not `.html.txt`.
3. Double-click it. It opens in your default browser, and no install or server is needed.
4. **Sharing inside Rocket:** use Rocket Spin, which is the approved internal host. The game contains no client data, so there is nothing sensitive to worry about. Don't add any.

---

## 8. Intellectual property note

"Super Mario" characters, music, and sprites belong to Nintendo.

- Draw **original** art and compose **original** sound.
- Call the game "Mario-style", or give it an original title and character name.
- Use it for learning and internal demos, and don't distribute it publicly.

---

## 9. Verification checklist

- [ ] Opens by double-click, with no server or install.
- [ ] Controls respond (keyboard, and touch on a phone).
- [ ] No clipping through floors or walls.
- [ ] Stomp and side-hit are handled correctly.
- [ ] Every block and item behaves as described.
- [ ] Title, play, pause, game over, and win all work.
- [ ] Sound starts after the first input, and mute works.
- [ ] Steady 60 FPS, with no console errors.
- [ ] High score persists after a refresh.

AI can be wrong, so play-test the output before sharing it.

---

## Appendix: One-shot quick-demo prompt

```text
Create a complete, playable Super Mario-style platformer in ONE index.html file
with HTML5 Canvas and vanilla JS only (no libraries, no external assets).
Include: a player with acceleration and variable-height jump, a tile map
level about 120 tiles wide with pits, pipes, bricks and question blocks,
Goomba-style enemies you can stomp, coins, a mushroom power-up, a flagpole goal,
HUD (score, coins, lives, time), title/game-over/win screens, and Web Audio
sound effects. Controls: arrows or WASD plus Space; Shift to run. Use
delta-time. Keep tunable constants at the top. Return only the full file in one
code block, then a 3-line how-to-play note.
```
