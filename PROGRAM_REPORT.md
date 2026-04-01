# Bottle Rocket Simulator – Program Report

## 1) What this program is

`simulator.py` is a single-file Tkinter desktop app that simulates and visualizes a compressed-gas + liquid rocket engine (water rocket style). It computes thrust over time and related performance metrics, then plots selected outputs with Matplotlib embedded in the GUI.

Primary outputs shown to the user:
- Total impulse (`Ic`)
- Burn/coast total time (`tc`)
- Specific impulse estimate (`Ist`)
- Delta-v estimate (`delta_v`)

## 2) Core program structure

The application is organized in one monolithic script with:
- Global constants and output arrays
- Simulation/optimization functions
- Plotting functions
- Save/load/export helpers
- Tkinter widget creation + event loop (`root.mainloop()`) at module bottom

Key function groups:
- Simulation engine: `simulate`, `opt_sim`
- Parameter sweep optimization: `optimalize`
- Plot updates: `plot_Ft`, `plot_mass`, `plot_temperature`, `plot_pressure`, `plot_volume`, and optimization plots
- Persistence: `save_to_file`, `load_from_file`
- Export: `export_to_file`, `save_to_export`
- Efficiency configuration dialogs for simulation and optimization

## 3) Physics model currently implemented

The thrust model is split into staged loops:
1. Launch rod phase (optional rod geometry handling)
2. Water-expulsion phase (liquid thrust)
3. Gas-expulsion phase, including a choked-flow branch and a subsonic branch

The code uses a very small fixed timestep (`delta_t = 0.0001`) and integrates force/mass changes iteratively. Atmospheric pressure is fixed at 1 atm.

## 4) Inputs and outputs

### Inputs (GUI)
- Gas choice (preset or custom `k`, `Rs`)
- Pressure, throat diameter, total chamber volume, water fraction, dry mass, gas temperature
- Liquid density
- Launch rod length and rod inside diameter
- Efficiency coefficients per stage

### Outputs (GUI)
- Text panel with `Ic`, `tc`, `Ist`, `delta_v`
- Plots versus time (thrust, mass, temperature, pressure, volumes)
- Optimization plots for selected variable sweep (e.g., pressure, throat, volume, etc.)

### File operations
- Save/load of session fields to text file (`save_to_file`, `load_from_file`)
- Export to `.eng` profile using time-force pairs (`save_to_export`)

## 5) Strengths

- Broad, configurable user input surface for educational/what-if exploration.
- Includes both direct simulation and optimization sweeps.
- Includes stage-specific efficiency tuning.
- Includes export path for engine profile usage.

## 6) Main technical debt and risks

1. **Single-file architecture:** all concerns are tightly coupled in one file (GUI, model, plotting, I/O), which hurts testability and maintainability.
2. **Global mutable state:** simulation arrays and settings are global, making behavior harder to reason about and unit-test.
3. **Potential data mutation bug in pressure plotting:** `plot_pressure` divides `array_pressure` values in-place by 100000 each time the plot is requested, so repeated plotting can corrupt units.
4. **Load path robustness:** `load_from_file` assumes all expected lines are present and indexed, with no guard for short/corrupt files.
5. **No automated test suite:** there are helper scripts under `test_files/`, but no formal repeatable tests for simulation correctness or regression detection.
6. **GUI boot on import:** because widget construction and `root.mainloop()` run at module level, importing functions for unit tests is difficult.
7. **Inconsistent naming/typos:** e.g., `optimalize`, `itteration`, `lenght` reduce readability and increase maintenance risk.

## 7) Practical way to test the program now

## A. Quick sanity checks (non-GUI)
1. Syntax/bytecode check:
   - `python3 -m py_compile simulator.py`
2. Optional strict linting (if tools installed):
   - `python3 -m pip install ruff`
   - `ruff check simulator.py`

## B. Manual GUI smoke test
Run:
- `python3 simulator.py`

Then verify this checklist:
1. Start app and click **Simulate** with valid values.
2. Confirm text output appears and thrust plot renders.
3. Click each left plot button (thrust/mass/temp/pressure/volume).
4. Open **Efficiency** dialog, set valid coefficients, rerun simulate.
5. Save a project via File → Save; reload via File → Load; verify fields repopulate.
6. Export `.eng`; inspect file has one header row + many `time thrust` rows.
7. On right panel, choose optimization variable, range, iterations; click **Optimalize**; verify each optimization plot button works.
8. Enter intentionally invalid values (negative pressure, invalid water %, invalid rod diameter) and verify “Invalid input.” / “Wrong input.” handling.

## C. High-value regression scenario
Use one canonical input set and record baseline values (`Ic`, `tc`, `Ist`, `delta_v`). Re-run after every change to detect unintended model drift.

## 8) What is left to fix/add (prioritized backlog)

## P0 (fix first)
1. **Fix pressure plot mutation bug** by plotting converted copies instead of mutating `array_pressure`.
2. **Add input file validation** in `load_from_file` (line count + numeric parse safety + user feedback).
3. **Separate model from GUI** so simulation functions can be imported and tested independently.

## P1 (next)
4. Add automated tests:
   - Unit tests for core simulation math with deterministic fixtures
   - Regression tests asserting metric ranges for known scenarios
   - File save/load round-trip tests
5. Replace global state with structured objects/dataclasses.
6. Add logging and structured error reporting.

## P2 (quality improvements)
7. Normalize naming (`length`, `iteration`, `optimize`) and refactor duplicated gas-selection blocks.
8. Add a command-line mode (headless) for batch optimization/testing.
9. Add CI checks (lint + tests) and versioned changelog.
10. Improve docs with validated example input presets and expected outputs.
