# TVC Pitch-Control Simulation

A Python simulation of a 2D rocket with a gimbaled motor (thrust vector control), with a PID controller that holds a commanded pitch angle against disturbances.

> **This is a simulation-only, simplified pitch-only model.** There is no physical rocket and no flight data. Results show how the controller behaves in this model, not how a real rocket would fly.

## Status

| Stage | Description | Status |
|---|---|---|
| 1 | Equations of motion (`docs/equations.md`) | Done |
| 2 | Uncontrolled simulation, tested against the closed-form solution | Not started |
| 3 | PID controller with gimbal angle limit, rate limit, anti-windup | Not started |
| 4 | Disturbances: wind gust, thrust misalignment | Not started |
| 5 | Saturation analysis: largest disturbance rejected before the gimbal saturates | Not started |
| 6 | Final checks, plots, write-up | Not started |

## Model

Rigid body, 2D pitch dynamics about the center of gravity (CG):

```
I · θ'' = T · L · sin(δ) − k · θ
```

| Symbol | Meaning | Units |
|---|---|---|
| θ | Pitch angle (positive counterclockwise) | rad |
| δ | Gimbal angle | rad |
| T | Thrust | N |
| L | Distance from CG to gimbal pivot | m |
| I | Pitch moment of inertia about the CG | kg·m² |
| k | Aerodynamic stiffness, `k = ½ρV²·S·C_Nα·d` | N·m/rad |

`k > 0` means the center of pressure is aft of the CG (statically stable). `k < 0` means it is forward (unstable).

Full derivation, sign conventions, and limits: [`docs/equations.md`](docs/equations.md).

## Assumptions and limitations

- Constant mass, moment of inertia, CG location, and lever arm L (a real motor loses propellant as it burns)
- Constant k (in reality k scales with V², and is near zero at launch)
- Vertical flight with no translation, so angle of attack equals pitch angle. This breaks down with wind or a curved flight path
- No drag and no aerodynamic damping
- Small-angle linearization (`sin δ ≈ δ`) is used only for the closed-form validation; the simulation keeps `sin δ`
- Gimbal responds instantly in the closed-form check; the simulation adds an actuator rate limit
- 2D pitch only: no yaw, roll, or 3D coupling

## Validation

Planned: the numerical result for a small-angle step input is compared against the closed-form solution

```
θ(t) = (T·L·δ₀ / k) · (1 − cos(ω_n t)),   ω_n = √(k / I)
```

using a pytest test. This checks the integrator and the small-angle approximation. It does **not** show that the model matches a real rocket.

OpenRocket is used only as a source of parameters (mass, CG, CP, C_Nα, moment of inertia, thrust curve). The simulation is not validated against OpenRocket flight results.

## Planned structure

```
docs/       equations and notes
src/        model.py, sim.py, pid.py, params.py, plots.py
tests/      pytest checks
data/       thrust curve and parameters
results/    plots
```

Folders are created as they gain content.

## Setup

```
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
```

## Tools

Python (NumPy, SciPy, Matplotlib), OpenRocket for parameters, Autodesk Fusion for geometry.

## Possible extensions (out of scope for now)

3D dynamics, quaternions, Kalman filtering, CFD, planar 3-DOF with translation and angle-of-attack from velocity.
