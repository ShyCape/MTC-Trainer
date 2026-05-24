# MTC Trainer Development Log

This document tracks the development of the MTC Trainer, a web-based tank training simulator built with Three.js.

## Project Overview
MTC Trainer is a single-file web application that provides a 3D environment for training in various tank roles (Gunner, Driver, Commander, Loader) across different eras (WW2, Cold War, Modern).

## Current Status (as of May 18, 2026)

### Core Technologies
- **Frontend:** HTML5, CSS3, JavaScript (ES6)
- **3D Engine:** Three.js (r128)
- **Assets:** `MainMenuBackGround.png` (used for the main menu background)

### Implemented Features

#### 1. Main Menu & Navigation
- **Era Selection:** UI for selecting WW2, Cold War, or Modern eras.
- **Role Selection:** UI for selecting Gunner, Driver, Commander, or Loader roles.
- **Intro/Loading Sequences:** Basic fade-in and loading messages.

#### 2. WW2 Gunner Role (Fully Functional Prototype)
- **3D Environment:**
  - Procedural terrain generation using noise functions.
  - Environment details: Trees, wall ruins, sandbags, and craters.
  - Fog and lighting (Ambient + Directional with shadows).
- **Player Tank:**
  - Basic 3D model of a tank with turret and barrel movement.
  - Gunner's perspective (above the turret).
- **Combat Mechanics:**
  - **Aiming Mode:** Precision aiming with a custom SVG scope overlay and FOV zoom (Toggle with 'X').
  - **Firing:** Projectile-based firing system with muzzle flashes and point lights.
  - **Ammo Types:** Implementation of four shell types (APHE, HE, APCR, APHECBC) with varying speed, splash, and reload times.
  - **Reloading:** Timed reload bar logic.
- **Targeting System:**
  - Enemy tanks spawn and move in the valley.
  - Target acquisition indicator when enemies are within 80m.
  - Hit markers and kill logging.
- **HUD:**
  - Comprehensive HUD showing Era, Role, Kills, Bearing (BRG), and Elevation (ELEV).
  - Ammo counter and shell selection UI.
  - Controls reference panel.

#### 3. Controls
- **X:** Toggle Aim Mode
- **F:** Fire
- **W / S:** Elevation (Inverted)
- **A / D:** Traverse
- **1 - 4:** Load specific shell types
- **ESC:** Exit to menu

## Development History (Internal)
- Initial prototype established with Three.js integration.
- Terrain and environment rendering optimized for single-file deployment.
- Gunner HUD and scope overlay implemented using SVG and CSS.
- Basic AI for enemy tank movement and respawn waves.

---

## Session Log — May 18, 2026 (Claude via MCP)

### Changes Made This Session

#### Three.js Loading & Stability Fixes
- Replaced single CDN script tag with a dual-CDN loader: tries `unpkg.com` first, falls back to `cdnjs.cloudflare.com`, with a visible error message if both fail.
- Added a `window.THREE` polling guard in `launchWW2Gunner()` — scene init no longer fires until Three.js is confirmed loaded.
- Fixed `#intro-fade` starting at `opacity: 1` (was blocking the entire screen on load) — now starts at `opacity: 0`.
- Fixed race condition in scene launch: fade-to-black now completes before the canvas is made visible, and a double `requestAnimationFrame` defers `initWW2Scene()` until the canvas is fully laid out in the DOM.
- Added try/catch around `WebGLRenderer` init with console error on failure.

#### Hill / Player Position
- Camera placed on a raised hill (`y = 8`, `z = 10`) overlooking a valley.
- Terrain generation updated: Gaussian hill bump added at player position, valley floor flattened in front for clear sightlines.
- Sandbag emplacements added around the hill crest to suggest a defensive firing position.
- Trees added flanking the hill sides; cleared a forward arc for unobstructed view into the valley.
- A dirt road added running through the valley centre — enemy tanks now advance along it.

#### Controls Overhaul
- Removed pointer lock / mouse click firing system entirely.
- **X (hold)** — Enter aim mode: scope overlay fades in, FOV zooms from 55° to 18°, mouse look activates.
- **F** — Fire current shell.
- **W / S** — Barrel elevation up/down (continuous, delta-time based).
- **1–9** — Select from 9 shell types (see below).
- **ESC** — Exit to menu.
- Mouse movement only moves the camera while X is held; releases cleanly on key-up.
- Added `window blur` listener to clear key state and drop aim mode if focus is lost.

#### Shell Type System (9 Types)
Full shell selector bar added to HUD bottom-centre. Each shell has unique colour, muzzle velocity, damage multiplier, and reload time:

| # | Name  | Speed | Damage | Reload |
|---|-------|-------|--------|--------|
| 1 | AP    | 2.2   | 1.0×   | 2.5s   |
| 2 | APHE  | 2.0   | 1.5×   | 3.0s   |
| 3 | APCR  | 2.8   | 0.8×   | 2.0s   |
| 4 | HE    | 1.6   | 2.0×   | 3.5s   |
| 5 | HEAT  | 1.8   | 1.8×   | 3.2s   |
| 6 | SMOKE | 1.4   | 0×     | 2.0s   |
| 7 | APCBC | 2.1   | 1.2×   | 2.8s   |
| 8 | HEP   | 1.7   | 1.6×   | 3.0s   |
| 9 | APDS  | 3.0   | 0.7×   | 1.8s   |

Each shell type has its own ammo count. Kill log now records shell type used per kill.

#### HUD Updates
- Added always-visible small crosshair dot (replaces scope when not aiming).
- Added `[ AIMING ]` indicator that appears at top-centre when X is held.
- Elevation readout (`ELEV`) now live in left-side panel.
- Shell bar highlights active slot; each slot shows key number and shell name.
- Bearing display remains live during gameplay.
- Enemy tanks now advance toward the player hill and respawn in waves of 4.
- Kill distance now calculated from player hill position rather than world origin.

#### Waypoint System (Middle-Click Move)
- **MMB** — Issues a move order to the clicked location.
- Added a green marker with pulsing ring and vertical pole to indicate the waypoint.
- Tank hull now automatically navigates to the waypoint, tilting to match terrain and facing the movement direction.
- **Fixed Turret/Camera Sync**: Turret and camera now stay aligned with world bearing even when the hull turns.
- **Standardized Controls**: W (Up), S (Down) elevation.
- **Enemy Pursuit**: Enemies now always track and pursue the player's position.

---

## Session Log — May 18, 2026 (Gemini CLI)

### Changes & Fixes
- **Reverted to Stationary Training**: Removed manual WASD driving to focus exclusively on gunner training. WASD and Arrow Keys are now both mapped to turret traverse and elevation.
- **Fixed Camera-Turret Rotational Coupling**: Ensured the camera's rotation (view) stays perfectly synchronized with the turret's aim, even when stationary or moving via waypoint.
- **Refined Waypoint Navigation**: Tank hull now correctly orients to the movement vector and snaps to terrain slope while moving.
- **Turret/Camera Decoupling**: Fixed the regression where camera and turret would misalign when the hull rotated. Turret world rotation now tracks the absolute camera yaw.
- **Elevation Logic Fix**: Corrected elevation to match standard expectations (W=Up, S=Down) and fixed the world-space barrel tilt to account for hull pitch.
- **Enemy AI Correction**: Enemies now correctly target the player's current position as their primary objective.
- **Code Stability**: Cleaned up the animation loop to ensure consistent camera following and terrain snapping.
