---
layout: page
title: Geometric Tricopter Controller
description: Developed a non-linear tracking control strategy for a Y-configuration tricopter on the SE(3) manifold.
img: assets/img/tricopter_main.jpg
importance: 2
category: project
related_publications: False
---

This project was motivated by an exploration of geometric numerical methods.

Geometric numerical methods focus on developing control systems for dynamics that evolve on nonlinear manifolds which cannot be globally identified with Euclidean spaces. By characterizing the intrinsic geometric properties of these manifolds, this approach provides unique insights into control theory that are unobtainable from models using local coordinates.

Eessentially, they are used to maintain long-term consistency in mathematical representations such as energy, trajectory, orientation, etc. For example, when modeling satelite orbits, most traditional methods will round their calculations to an arbitrary precision. This will ultimetly lead to an unstable orbit, as the error built up from rounding will compound over time. However, utililizing geometric methods allows us to create long-term stable orbital dynamics that respect the total energy in the system.

In the context of drones, geometric methods come into play because of the difficulty of representing 3D position and attitude (orientation).

### Limitations of Traditional Representations
When modeling 3D rotation and translation, traditional methods often fall short:

* **Euclidean/Euler Angles:** Object translation is represented by x,y,z vector and attitude is represented with yaw, pitch, roll angles. These local parametrizations suffer from mathematical singularities, such as gimbal lock, which can cause control laws to fail during complex maneuvers.
* **Quaternions:** Object translation and orientation is uniquely represented by one 4-dimensional vector q = [w,x,y,z]. While they avoid singularities, quaternions have their own failure point, in that q = -q, meaning that there is always ambiguity when dealing with a quaternion measurment.

By formulating our controller geometrically, directly on the **Special Euclidean group $SE(3)$** using rotation matrices, we eliminate these difficulties and create a globally consistant controller thta works no matter the drone's position/attitude.


The rest of this page is an outline/summary of the geometric controller I developed, proved, and simulated. For the full controller structure and proofs of the stability analysis, you should read the paper. It's a little technical, but its available along with the simulation code at https://github.com/tomaszfrelek2/geometric-controller.
---

## Project Overview

Tricopter UAVs are less popular than quadcopter UAVs, but they offer significant agility advantages over standard quadrotors, particularly in **yaw authority**. This is because yawing is done by using a tilting rear rotor rather than differential drag torque as in a quadcopter. However, this configuration introduces a unique mechanical challenge: the tilting required for yaw control generates **parasitic lateral forces** that couple the heading moment with translational dynamics.

This project presents a **geometric tracking control strategy** defined globally on the **Special Euclidean group SE(3)**. By treating the vehicle dynamics intrinsically on the manifold, the controller avoids the singularities of Euler angles (gimbal lock) and the ambiguities of quaternions (unwinding phenomena).

---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tricopter_model.png" title="Tricopter Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The standard Y-configuration tricopter model features two fixed front rotors and one rear rotor mounted on a tilting servo mechanism.
</div>


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/servo_angle.png" title="Servo Angle" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Tilting the rear rotor to yaw the drone will also create unwanted translational movement. This complicates the controller dynamics.
</div>


---

## Methodology

### 1. Nonlinear Control Allocation
To resolve the coupling between yaw and translation, we developed a **nonlinear allocation scheme**. This allows the attitude dynamics to be treated as a fully actuated rigid body while isolating parasitic lateral forces as **bounded disturbances**.

### 2. Geometric Control on SE(3)
The controller separates the system into two subsystems:
* **Translational Subsystem**: Calculates the required thrust magnitude and vertical axis direction to track a 3D position trajectory $x_d(t)$.
* **Rotational Subsystem**: Computes the virtual moment vector $M$ to align the vehicle with a desired rotation matrix $R_d$, constrained by a reference heading.



---

## Stability Analysis

We conducted a rigorous **Lyapunov stability analysis** to prove the reliability of the controller under aggressive maneuvers:
* **Exponential Stability**: The attitude tracking errors are shown to converge exponentially to zero.
* **UUB Stability**: Translational tracking errors are **Uniformly Ultimately Bounded (UUB)**, meaning the vehicle stays within a calculated "ball" of error despite parasitic forces.
* **Almost Global Attractiveness**: The system can recover from nearly any initial orientation, ensuring recovery even from large initial attitude errors.



## Results & Simulations

The controller was validated in a numerical environment modeling full nonlinear dynamics and actuator saturation.
* **Trajectory Tracking**: Demonstrated smooth convergence to a diagonal path with zero steady-state error.
* **Aggressive Recovery**: Successfully stabilized the vehicle from a nearly inverted initial state (178° roll), proving the robustness of the geometric approach over linear methods.


---

<div class="row">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/diag_man.png" title="Normal Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/agg_man.png" title="Recovery Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <b>Left:</b> Simulation of a simple diagonal flight. <b>Right:</b> Simulation of an upside-down recovery from a 178° roll angle.
</div>

---