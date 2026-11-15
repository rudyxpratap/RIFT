# RIFT: Space Command

**Resourceful Intelligence for Flight & Technology**

A first-person 3D space mission simulation where you step into the role of a Space Mission Director. Manage budgets, configure spacecraft, select launch vehicles, plan trajectories, and watch your mission launch — all while balancing risk, cost, and scientific yield.

---

## 🎮 Game Overview

**Genre:** Simulation / Strategy / First-Person Exploration  
**Engine:** Godot 4.4 (Forward+ Rendering)  
**Language:** GDScript  
**Platform:** Desktop (Windows/Linux/macOS)  

### The Fantasy
You are the newly appointed Mission Director at RIFT Space Command. The former director hands you classified files, a sanctioned government budget, and a team of engineers and scientists. Your job: design, build, and launch space missions that push the boundaries of human knowledge — without going over budget or blowing up on the pad.

---

## 🚀 Core Gameplay Loop

### 1. **Boardroom Briefing** (Scene 1.2)
Meet the former Mission Director. Learn the ropes: budget constraints, equipment tiers, launch vehicle risk, trajectory planning, and the career progression system.

### 2. **Mission Dossier** (Scene 2)
Read your government-sanctioned budget. Each mission tier (Easy/Medium/Hard) has a different budget range. Your credibility score tracks career progress.

### 3. **Engineering Bay** (Scene 3)
Configure your spacecraft within budget:
- **Part 1 — Spacecraft Bus & Power**: Minimum (+5%), Recommended (+25%), Best (+50%)
- **Part 2 — Scientific Payload & Avionics**: Same three tiers
- **Launch Vehicle (LV)**: Heavy (5% risk), Standard (15% risk), Light (25% risk)

Every choice costs money and affects your **Success Rate Counter** (starts at 0%, max 99%).

### 4. **Control Center** (Scene 4)
Work with the Head Scientist to select your orbital trajectory:
- **Best Trajectory**: 10% risk deduction
- **Moderate Trajectory**: 25% risk deduction  
- **Risky Trajectory**: 45% risk deduction

Trajectory risk + LV risk are subtracted from your Success Rate Counter before launch.

### 5. **Launch Observation** (Scene 4.2)
Watch your rocket lift off in real-time 3D. Backend RNG rolls 1–100. If roll ≤ Success Rate → **Mission Passed**. Otherwise → **Mission Failed**.

### 6. **Transit Photo Feed** (Scene 5)
30-minute real-time satellite telemetry feed with 6 photo milestones. Verify daily trajectory drift. Fast-forward available.

### 7. **Mission Outcome** (Scene 6)
- **Success**: +50 Credibility, advance to next mission tier
- **Failure**: -25 Credibility, newer team assembled (Team Generation +1), 30-minute cooldown before retry

---

## 🛠 Technical Architecture

### Autoloads (Global Singletons)
| File | Purpose |
|------|---------|
| `GameState.gd` | Budget math, success rate calculation, equipment tier state, mission progression |
| `MissionState.gd` | Full physics engine: mass, power (inverse-square law), data downlink, science yield, composite NASA 4-pillar score, alerts/blockers |

### Scenes
| Scene | Description |
|-------|-------------|
| `MainGame.tscn` / `MainGame.gd` | Master controller: 7-stage state machine, FPS player, dual-input UI (FPS + mouse failsafe), 3D world builder |
| `Boardroom.tscn` / `Boardroom.gd` | Boardroom meeting room |
| `MainDashboard.tscn` / `MainDashboard.gd` | Mission selection dashboard |

### Data-Driven Design
All gameplay values in `data/catalog.json`:
- 3 Mission tiers (Heliophysics, Stellar Circumnavigation, Deep Space Pioneer)
- 4 Orbits (LEO, MEO, HEO, Heliocentric Pluto)
- 3 Launch Vehicles (Falcon 9, SLS Block 1, Ariane 6)
- 3 Bus classes (CubeSat 3U → LargeSat 100U)
- 3 Power systems (Solar, RTG, Hybrid)
- 3 Batteries, 3 Comms, 3 ADCS, 3 Propulsion, 6 Instruments

### 3D Asset Pipeline
- **Procedural fallback**: All 3D content generated at runtime (boxes, materials, lights)
- **GLB slot system**: Drop `.glb` files in `assets/models/` — auto-scaled & centered via bounding-box scaler
- **Supported slots**: earth, door, director, engineer, scientist, desk, file_folder, satellite, rocket

---

## 🎯 Key Systems

### Success Rate Formula
```
Success Rate = Part1_Tier + Part2_Tier - LV_Risk - Trajectory_Risk
```
Clamped to 0–99%. Higher = better launch odds.

### NASA 4-Pillar Composite Score (MissionState)
1. **Objectives** — Science yield vs. mission target
2. **Yield** — Data capture ratio (downlink capacity)
3. **Efficiency** — Science per dollar (budget discipline)
4. **Risk** — Power margin, eclipse margin, blockers, warnings

### Inverse-Square Power Law
Solar panel output scales with `1 / distance²`. RTG/Hybrid unaffected by distance.

### Eclipse Battery Drain
```
Drain (Ah) = Peak_Power_Load (W) × Eclipse_Hours / 28V
Usable Capacity = Battery_Rating × 0.8
```

---

## 🎮 Controls

| Input | Action |
|-------|--------|
| **WASD / Arrow Keys** | First-person movement |
| **Mouse** | Look around (captured) |
| **Shift** | Sprint |
| **E / Enter** | Interact with doors, NPCs, files |
| **ESC** | Toggle mouse capture / Close modals |
| **Mouse Click** | UI fallback (never soft-locked) |

**Dual-Input Failsafe**: Every interaction works via both keyboard (E/Enter) AND mouse-clickable quick-action buttons in the top HUD. You can never get stuck.

---

## 📦 Project Structure

```
RIFT/
├── .git/
├── .gitattributes
├── LICENSE
├── README.md
├── project.godot          # Godot project config
├── autoload/
│   ├── GameState.gd       # Budget & progression state
│   └── MissionState.gd    # Physics engine & mission metrics
├── data/
│   ├── catalog.json       # All equipment, missions, orbits
│   └── narrative.json     # Story text, briefings
├── scenes/
│   ├── MainGame.tscn/gd   # Master 7-stage controller
│   ├── Boardroom.tscn/gd
│   └── MainDashboard.tscn/gd
├── assets/
│   ├── models/            # Drop .glb files here (auto-detected)
│   └── videos/            # Background video (space_bg.ogv)
└── run_log.txt            # Startup verification output
```

---

## 🏃 Running the Game

### Requirements
- **Godot 4.4+** (Forward+ renderer)

### Launch
```bash
# From project root
godot --path .
```

Or open `project.godot` in Godot Editor and press **F5**.

### Startup Verification
On launch, the game runs an automated self-test suite (see `run_log.txt`):
- GameState math verification
- 3D scene hierarchy validation
- UI node verification
- GLB asset slot verification (10 models)
- All 9 stage transitions verified

---

## 🧪 Development

### Adding Custom 3D Models
1. Export as `.glb` (glTF Binary)
2. Place in `assets/models/` with matching name:
   - `earth.glb` (3.2m diameter)
   - `door.glb` (2.8m)
   - `director.glb` / `engineer.glb` / `scientist.glb` (1.8m)
   - `desk.glb` (2.2m)
   - `file_folder.glb` (0.5m)
   - `satellite.glb` (2.5m)
   - `rocket.glb` (4.0m)
3. Restart — auto-scaled, centered, collision-ready

### Modifying Gameplay Data
Edit `data/catalog.json` — all stats, costs, masses, science yields are data-driven.

### Extending Missions
Add new mission objects to `catalog.json` → automatically appears in progression.

---

## 📝 License

MIT License — see [LICENSE](LICENSE)

---

## 🙏 Credits

Built with **Godot 4** • Designed in **Claude Code Game Studios** agent architecture  
Procedural 3D, data-driven simulation, dual-input failsafe UI — zero external dependencies

---

*"The stars are not the limit. They're the destination."*  
— RIFT Space Command Motto