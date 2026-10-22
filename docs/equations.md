# Equations of motion


## 1. Setup and coordinate convention
The rocket flies vertically (nose up). The model is 2D: only rotation in the pitch plane is simulated.
(diagram will come soon)


| Symbol | Meaning | Positive direction |
|---|---|---|
| θ | Pitch angle, measured from vertical | Counterclockwise (nose tilts left of vertical) |
| ω = θ' | Pitch rate | Counterclockwise |
| δ | Gimbal angle, thrust vector relative to the rocket axis | Thrust tilts to the right of the axis (nose up), which gives a counterclockwise torque |
| L | Distance from CG to gimbal pivot, along the axis | Positive: the pivot is aft of the CG |
| d | x_CP − x_CG, with x measured from the nose tip | Negative: CP forward of the CG |

All positions are measured from the nose tip. Angles are in radians inside the equations.

---

## Parameters (from the OpenRocket design)

Source: OpenRocket component analysis, motor loaded, Mach 0.3, angle of attack 0°. Finless rocket, 75 cm long, 5 cm diameter, AeroTech G79W motor.

| Quantity | Symbol | Value | Where it came from |
|---|---|---|---|
| Loaded mass | m | 1.253 kg | Component analysis, total row |
| CG position | x_CG | 44.245 cm | Component analysis |
| CP position | x_CP | 6.937 cm | Component analysis |
| Normal force slope | C_Nα | 2 per rad | Component analysis (displayed rounded) |
| Reference area | S | 19.6 cm² = 0.00196 m² | Component analysis |
| Pitch moment of inertia | I | 0.02675 kg·m² | OpenRocket simulation output, longitudinal moment of inertia at t = 0 (read from the plot) |
| Gimbal pivot | x_pivot | about 60.0 cm | Forward end of the motor: motor CG at 67.493 cm minus half its 150 mm length |
| Lever arm | L | 0.1576 m | x_pivot − x_CG |
| Aero distance | d | −0.3731 m | x_CP − x_CG |
| Average thrust | T | 79.2 N | OpenRocket motor dialog (107 Ns ÷ 1.35 s) |
| Burn time | t_b | 1.35 s | OpenRocket motor dialog |
| Propellant mass | | 60.9 g | Launch mass 154 g − empty mass 93.1 g |
| Representative airspeed | V | 40 m/s | Choice, see Section 4 |
| Air density | ρ | 1.225 kg/m³ | Sea-level standard atmosphere |

Estimates: burnout I ≈ 0.023 kg·m² and burnout mass ≈ 1.19 kg.

OpenRocket is used only as a source of parameters. Its flight simulation is not used, because it has no gimbal and the rocket tumbles under thrust.



## 2. Control torque
- Derivation of τ = T·L·sin(δ)


1. The motor produces thrust of magnitude T, directed along the nozzle axis.


2. When the nozzle is deflected by angle δ from the rocket’s centerline, split the thrust into two components in the body frame:
        Axial: T·cos(δ), along the rocket axis
        Lateral: T·sin(δ), perpendicular to the rocket axis


3. The pivot sits on the rocket axis, which passes through the CG, so the axial component’s line of action goes through the CG. Its lever arm is zero, so it produces no torque.


4. The lateral component acts at the pivot, a distance L from the CG, and is perpendicular to the lever arm. For a force perpendicular to the arm, torque = force × arm: τ = T·L·sin(δ)


- Units check
The pivot lies exactly on the axis through the CG (no offset).
The thrust acts at the pivot point.
Thrust direction follows the nozzle instantly. Real servos lag, which is why your plan includes an actuator rate limit.
T is treated as the instantaneous thrust. It changes in time (from the ThrustCurve.org data), so in the simulation this becomes T(t).
The lateral force T·sin(δ) also pushes the rocket sideways. The 2D pitch-only model ignores that translation. Say so in Section 7.




## 3. Rotational dynamicss
- τ = I·θ''


1. Treat the rocket as many small mass elements dm, each at distance r from the axis of rotation.
2. If the rocket rotates with angular acceleration α = θ'’, an element’s tangential acceleration is a = r·α.
3. The tangential force on that element is dF = dm · r·α.
4. The torque on that element is dτ = r · dF = r² · α · dm.
5. Add up all elements. α is the same for every element because the body is rigid: τ_net = α · ∫ r² dm = I·α


- What I is, and why we take it about the CG
I = ∫ r² dm is the moment of inertia, the rotational analog of mass. it measures how hard it is to change the rotation rate, and it depends on how the mass is distributed, not just on how much there is. Mass far from the axis course counts much more.
Units: kg·m² or N·m


the reason why we talk about the CG is because a free rocket has no fixed pivot point. Its motion is spilt into 2 independents part, the CG translates as of all the mass were concentrated there (F=ma) and the body roatess about the CG. The simple equation (τ = I·θ'’) os exactly true only when both tau and I are measured about the CG. Using any other point adds extra coupling term between translation and rotation. This is also why the gimbal lever arm L is measured from the CG.




## 4. Aerodynamic torque (simplified)

**Idea.** When the rocket axis is tilted relative to the oncoming air, the air pushes sideways on the body. That force acts at the center of pressure (CP), not the CG, so it produces a torque about the CG.

**Derivation.**
1. Angle of attack α is the angle between the rocket axis and the airflow. In this model the rocket flies on a fixed vertical path at speed V, so **α = θ**.
2. For small α the normal force is N = q·S·C_Nα·α, with dynamic pressure q = ½ρV².
3. N acts at the CP, a distance d = x_CP − x_CG from the CG (negative when the CP is forward). With the CP aft of the CG, the force pushes the tail away from the airflow and rotates the nose back toward it, so the torque is τ_aero = −N·d·(sign convention fixed in Section 1).
4. Result:

   **τ_aero = −k·θ,   k = ½ρV²·S·C_Nα·d**

**Units.** ρ [kg/m³] · V² [m²/s²] · S [m²] · C_Nα [1/rad] · d [m] gives kg·m²/s² per rad = **N·m/rad**.

**Sign of k.**
- k > 0: CP aft of the CG. Restoring torque, statically stable.
- k < 0: CP forward of the CG. The torque grows with the angle, statically unstable.
- k = 0: neutral.

**This rocket.** d = −0.3731 m, so k < 0 (unstable by design). The stability margin is (6.937 − 44.245) / 5 ≈ −7.5 calibers.

**Value at V = 40 m/s.**
k = ½ · 1.225 · 40² · 0.00196 · 2 · (−0.3731) ≈ **−1.43 N·m/rad**

V = 40 m/s is a representative speed: the burnout speed is about (107 Ns ÷ 1.22 kg) − 9.81 · 1.35 ≈ 74 m/s, the speed rises roughly linearly from zero, and the average over the burn is about 37 m/s, rounded to 40.

**Instability time scale.** With I = 0.02675 kg·m², the uncontrolled growth rate is a = √(|k|/I) ≈ 7.3 s⁻¹, so the angle grows by a factor of e roughly every 0.14 s. Over the 1.35 s burn this is a very large amplification, which is why the controller has to act within about a tenth of a second.

**Limits.**
- k ∝ V². At 74 m/s the same formula gives about −4.9 N·m/rad, and near launch k is close to zero. The model uses one constant value.
- α = θ only for vertical flight with no wind. A crosswind w adds roughly w/V to the angle of attack, which matters for the wind-gust disturbance in a later stage.
- C_Nα and CP depend on Mach number and angle of attack. The values here are at Mach 0.3, α = 0°.
- No aerodynamic damping term is included.


## 5. Full model

**I·θ'' = T·L·sin(δ) − k·θ**

State-space form, state x = [θ, ω]ᵀ:

- θ' = ω
- ω' = (T·L·sin(δ) − k·θ) / I

Linearized (sin δ ≈ δ), input u = δ:

x' = A·x + B·u, with A = [[0, 1], [−k/I, 0]] and B = [0, T·L/I]ᵀ

Control authority: T·L = 79.2 × 0.1576 ≈ 12.5 N·m/rad, so θ'' ≈ 467 rad/s² per radian of δ, or about 41 rad/s² at δ = 5°.


## 6. Linearization and closed-form solution

**Linearization.** sin δ = δ − δ³/6 + …, so the relative error of sin δ ≈ δ is roughly δ²/6.

| δ | Relative error |
|---|---|
| 1° | 0.005% |
| 5° | 0.13% |
| 10° | 0.51% |
| 15° | 1.15% |
| 20° | 2.06% |
| 30° | 4.7% |

**Closed-form step response.** Constant T and k, step input δ(t) = δ₀, with θ(0) = 0 and θ'(0) = 0.

For k < 0 (this rocket), let a = √(|k|/I). The equation is θ'' − a²·θ = (T·L/I)·δ₀.

1. Particular solution: θ_p = −T·L·δ₀/|k|.
2. Homogeneous solution: A·cosh(a·t) + B·sinh(a·t).
3. θ(0) = 0 gives A = T·L·δ₀/|k|. θ'(0) = 0 gives B = 0.

**θ(t) = (T·L·δ₀ / |k|) · (cosh(a·t) − 1),   a = √(|k|/I)**

For k > 0, with ω_n = √(k/I): θ(t) = (T·L·δ₀ / k)·(1 − cos(ω_n·t)). For k = 0: θ(t) = ½·(T·L/I)·δ₀·t².

Verify each by substituting back into the ODE.

**Choosing the validation test.**
- Keep θ small so the linear model holds. With δ₀ = 0.002 rad (about 0.11°) and t_end = 0.3 s: θ(0.3) = (12.5 · 0.002 / 1.43) · (cosh(2.19) − 1) ≈ 0.06 rad, about 3.5°.
- At this δ₀ the sin δ ≈ δ error is negligible (about 10⁻⁶ relative), so this test mainly checks the integrator, not the small-angle approximation.
- It does not show that the model matches a real rocket.


## 7. Assumptions and limitations
| Assumption | What it limits |
|---|---|
| Rigid body | No structural flex or propellant slosh |
| Constant mass, I, and CG | Propellant is about 61 g of 1253 g (about 5%). By 0.73 s of burn, OpenRocket showed mass down about 3% but I down about 8%, because the propellant sits far from the CG. Estimated burnout I is about 13% below the ignition value |
| Constant thrust T in the closed form | The simulation later uses T(t) from the 19-point thrust curve |
| Constant k | k ∝ V². About −1.4 N·m/rad at 40 m/s, about −4.9 at 74 m/s, near zero at launch |
| α = θ (vertical flight, no translation, no wind) | Breaks down with wind or a curved path |
| No aerodynamic damping | Overstates oscillation in the stable case |
| No drag | Drag changes V and therefore k |
| Pivot on the axis through the CG, thrust acts at the pivot | No thrust offset, except as a later disturbance |
| Pivot at the motor's forward end | A different choice changes L (59 cm tube front gives L ≈ 0.148 m, about 6% lower) |
| Moving motor mass ignored | The gimbaled motor is about 154 g of 1253 g. Its mass shift is ignored |
| Gimbal responds instantly (closed form only) | The simulation adds a rate limit |
| Small-angle linearization of sin δ | Error ≈ δ²/6, see Section 6 |
| 2D pitch only | No coupling with yaw or roll |
| OpenRocket used only for parameters | No validation against OpenRocket flight results is claimed |

