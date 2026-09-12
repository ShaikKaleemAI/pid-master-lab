# PID Master Lab — Live Animation Engine

![status](https://img.shields.io/badge/status-active-brightgreen)
![stack](https://img.shields.io/badge/stack-vanilla%20JS%20%2B%20Canvas-yellow)
![deps](https://img.shields.io/badge/dependencies-zero-informational)
![topics](https://img.shields.io/badge/topics-controls%20%7C%20simulation-blueviolet)
![license](https://img.shields.io/badge/license-MIT-blue)

An interactive PID controller simulator built to make control theory
*visible*: the same three gains (Kp, Ki, Kd) drive a live numerical
simulation, a physics-based vehicle animation, and a full set of analysis
views — all in one self-contained HTML file, no libraries.

## Why this project is on my resume

- **The simulation and the animation share one source of truth.** Every
  visual — the vehicle arena, the step/error/effort charts, the pole map,
  the Bode plot — is drawn from the same `simSystem()` output. Nothing is
  faked or independently tuned to "look right"; changing a gain changes
  every view consistently.
- **Four physical scenarios, one control loop.** Car cruise control,
  motor RPM, drone altitude, and rocket attitude are modeled as different
  plant transfer functions feeding the *same* PID loop — a concrete
  demonstration that PID is a general-purpose control pattern, not a
  car-specific trick.
- **Built from first principles, not a charting library.** Every chart,
  gauge, pole-zero map, and Bode plot is hand-drawn on `<canvas>|
  (`drawStepChart`, `drawPoleMap`, `drawBode`, …), including a real cubic
  solver (`solveCubic`) for locating closed-loop poles.
- **Ziegler–Nichols auto-tuning is actually implemented**, not just
  described — `autoTuneZN()` computes gains from the classic method
  rather than the app just letting you drag sliders until it looks stable.

## Feature tour

| Page | What it shows |
|---|---|
| **Arena** | A live physics animation of the selected scenario (car, motor, drone, or rocket) responding to the current PID gains in real time, with a signal-flow diagram and a plain-language explanation of what's happening at each moment. |
| **Compare** | Side-by-side response comparison across gain sets or scenarios. |
| **Charts** | Step response, error signal, and control-effort plots for the active configuration. |
| **Cause & Effect** | Isolates what changing Kp, Ki, or Kd alone does to the response — parameter sweeps rendered as overlaid curves. |
| **Real World** | Explains the physical meaning of the transfer function for each scenario — why a car, a motor, a drone, and a rocket all reduce to the same PID math with different plant dynamics. |
| **Math** | The underlying control theory: transfer functions, pole-zero maps, Bode plots, and the cubic root-finding behind them. |
| **Code** | The simulation logic itself, presented for readers who want the implementation, not just the behavior. |

## Core mechanics

- **`simSystem(b, k, Kp, Ki, Kd, tEnd, dt)`** — the numerical plant + PID
  simulation every other view reads from.
- **`computeMetrics(data)`** — overshoot, settling time, rise time, and
  steady-state error extracted from a simulated response.
- **`drawPoleMap` / `getPolesFor`** — closed-loop pole locations from the
  characteristic equation, solved with a hand-written cubic solver
  (`solveCubic`), plotted on the s-plane.
- **`drawBode`** — open-loop gain/phase Bode plot computed directly from
  the transfer function (`olGain`).
- **`autoTuneZN()`** — Ziegler–Nichols auto-tuning: finds the ultimate gain
  and period, then derives Kp/Ki/Kd from the classic tuning rules.
- **Per-scenario renderers** (`drawCar`, `drawMotor`, `drawRPMGauge`,
  `drawDrone`, `drawRocket`) — each a hand-built canvas animation driven
  by the same simulated `(y, err, u)` signal.

## Architecture

```
4_PID__working_Ultimate_Lab_.html
├── <style>          layout, sidebar, page tabs, canvases
└── <script>
    ├── Simulation core    simSystem, computeMetrics, getParams,
    │                     loadScenario, solveCubic
    ├── Live loop           updateAll → restartArena → frame() —
    │                     the per-tick animation driver
    ├── Analysis renderers  drawStepChart, drawErrorChart,
    │                     drawEffortChart, drawSweepCharts, drawPoleMap,
    │                     drawBode, drawCompare
    ├── Scenario renderers  drawCar/drawMotor/drawRPMGauge/drawDrone/
    │                     drawRocket + their HUD overlays
    ├── Auto-tune           autoTuneZN
    └── Navigation           showPage(id, tabEl) — tab switching across
                            Arena / Compare / Charts / Cause & Effect /
                            Real World / Math / Code
```

## Run it

Open the file directly in a browser — no build step, no server, no
external libraries. Pure HTML/CSS/JS and `<canvas>`.

## Ideas for extending

- Export a scenario's response data (CSV) for offline analysis.
- Add a disturbance-rejection scenario (step disturbance mid-simulation)
  to demonstrate integral windup and anti-windup handling explicitly.
- A discrete-time (sampled) simulation mode alongside the current
  continuous-time integration, to show quantization/sampling effects.
