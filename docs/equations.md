# Equations of motion


## 1. Setup and coordinate convention
- Diagram or description of the rocket: CG, gimbal pivot, L, θ, δ
- Which direction is positive for θ and for δ


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
- τ_aero = -k·θ
- What k represents, and what its sign means


## 5. Full model
- I·θ'' = T·L·sin(δ) − k·θ
- State-space form (θ, ω)


## 6. Linearization and closed-form solution
- sin(δ) ≈ δ, and how big the error is
- Solution for a step input


## 7. Assumptions and limitations
- Constant mass, constant k, no drag, no translation, small angles, ...

