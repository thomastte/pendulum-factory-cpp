# pendulum-factory-cpp

The C++ part of the MAS417 pendulum kit factory: `pendulum_sim`, a small command-line program that simulates a rigid (physical) pendulum with the 4th-order Runge–Kutta method (RK4) and measures its period.

It is normally used as the `cpp/` submodule of the main repository **pendulum-factory**. The Python program calls it with `subprocess`.

## Build

Requires CMake ≥ 3.16 and a C++20 compiler. No external libraries.

| OS | Compiler |
|---|---|
| Windows | Visual Studio Build Tools, workload "Desktop development with C++" |
| macOS | `xcode-select --install` |
| Linux | `sudo apt install build-essential cmake` |

From this folder:
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

From the main repository, use `-S cpp -B cpp/build` instead. The executable ends up in:
- `build/pendulum_sim` (macOS, Linux)
- `build/Release/pendulum_sim.exe` (Windows with Visual Studio)

## Usage

```
pendulum_sim I M d g theta0 damping t_end dt out.csv
```

| Argument | Unit | Meaning |
|---|---|---|
| `I` | kg·m² | moment of inertia of the swinging body about the axle |
| `M` | kg | mass of the swinging body |
| `d` | m | distance from the axle to the centre of mass |
| `g` | m/s² | gravity, e.g. 9.81 |
| `theta0` | rad | start angle (released from rest) |
| `damping` | 1/s | viscous damping, 0 for none |
| `t_end` | s | simulated time; must cover at least two swings |
| `dt` | s | time step, e.g. 1e-4 |
| `out.csv` | | output file |

**Output:**
- `out.csv` with columns `t,theta,omega`, one row per time step.
- One line on standard output: `period=<seconds>`.

**Exit codes:**
- 0: OK.
- 1: wrong arguments, or the output file can't be written.
- 2: fewer than two swings were simulated (increase `t_end`).

Example (point mass 50 g at 0.2 m, 5° start angle):
```bash
./build/pendulum_sim 0.002 0.05 0.2 9.81 0.0872664626 0 10 0.0001 sim.csv
# period=0.897567486617
```

## Model

θ'' = −(M·g·d / I)·sin θ − c·θ′, integrated with fixed-step RK4.

The period is the mean time between upward zero crossings of θ. Each crossing time is found by linear interpolation between the two samples around it.

## Verification

- For a point mass, the simulated period equals the exact elliptic-integral period 4·√(L/g)·K(sin(θ0/2)) to about 1e-13 relative error at 5° and at 45°.
- This is checked automatically by `tests/test_simulate.py` in the Python repository.
- This verifies the *code*, not the physics. The physical model is checked against the slicer's mass estimate and, if printed, a real kit.

## AI

The first version of `src/main.cpp` was generated with Claude and is marked in the file. See `docs/ai-log.md` in the main repository.
