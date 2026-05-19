# CPS_preparation
# 📘 CYBER-PHYSICAL SYSTEMS — COMPLETE EXAM PREPARATION GUIDE
### Based on: Rajeev Alur (Principles of CPS) + Lee & Seshia (Introduction to Embedded Systems, 2nd Ed.)
### Prepared by: Senior Academic Mentor | Exam-Oriented Deep Study Resource

---

> **HOW TO USE THIS DOCUMENT**
> This guide mirrors the exact blueprint structure. Every major topic includes:
> Theory → Derivations → Formulas → Worked Problems → Exam Tips → Revision Notes
> Start from your weakest topic. Return to revision notes before the exam.

---

# 📋 SYLLABUS MAP

| Unit | Topic | Source | Exam Weight |
|---|---|---|---|
| **1** | Continuous Dynamics & ODEs | Lee & Seshia Ch.2 (pp.19–30) | HIGH |
| **2** | Stability & BIBO | Lee & Seshia Ch.2 (pp.30–31) | HIGH |
| **3** | Feedback Control (Proportional) | Lee & Seshia Ch.2 (pp.31–36) | HIGH |
| **4** | Async Composition & Interleaving | Lee & Seshia Ch.5 (pp.107–120) | HIGH |
| **5** | Hard vs. Soft Real-Time | Lee & Seshia Ch.12 (pp.319–334) | MEDIUM |
| **6** | ESMs & Asynchronous Processes | Alur Ch.4 (pp.125–136) | HIGH |
| **7** | Real-Time Scheduling (RMS & EDF) | Alur Ch.8 (pp.339–378) | HIGH |
| **8** | Task Dependency Graphs (DAGs) | Alur Ch.2 | MEDIUM |
| **9** | Cyber Security in CPS | Stallings Ch.15 | HIGH |

---

---

# ═══════════════════════════════════════════════
# UNIT 1 — CONTINUOUS DYNAMICS & STATE-SPACE MODELS
## Source: Lee & Seshia, Ch.2 (pp.19–34) + Alur
# ═══════════════════════════════════════════════

---

## 1.1 Introduction & Intuition

**What is Continuous Dynamics?**

Imagine a helicopter hovering in the air. Its body constantly wants to spin due to Newton's third law (the main rotor spins one way, the body wants to spin the other way). How do we mathematically describe how fast the helicopter is rotating at any moment?

The answer is: we write a **differential equation** — a mathematical description of how quantities *change over time*. This is the foundation of continuous dynamics.

**Key Insight:** A CPS has two worlds — the *physical world* (continuous, governed by physics) and the *cyber world* (discrete, governed by code). Continuous Dynamics is how we model the **physical plant** — the thing we want to control.

---

## 1.2 Newtonian Mechanics — The Foundation

### Newton's Second Law (Translational Motion)

$$\mathbf{F}(t) = M\ddot{\mathbf{x}}(t)$$

| Symbol | Meaning |
|---|---|
| **F(t)** | Force vector (3D), function of time |
| **M** | Mass of the object (scalar) |
| **ẍ(t)** | Second derivative of position = Acceleration |
| **ẋ(t)** | First derivative of position = Velocity |
| **x(t)** | Position vector |

**Deriving velocity from force:**
$$\dot{x}(t) = \dot{x}(0) + \frac{1}{M}\int_0^t F(\tau)\,d\tau$$

**Deriving position from force:**
$$x(t) = x(0) + t\dot{x}(0) + \frac{1}{M}\int_0^t\int_0^\tau F(\alpha)\,d\alpha\,d\tau$$

> 🔑 **Key:** If you know initial position x(0), initial velocity ẋ(0), and the force F(t) for all time, you can determine the complete motion of the object.

---

### Newton's Law for Rotational Motion (Torque)

$$\mathbf{T}(t) = \frac{d}{dt}\left[\mathbf{I}(t)\dot{\boldsymbol{\theta}}(t)\right]$$

For a **spherical object** (simplified — moment of inertia is scalar I):

$$\mathbf{T}(t) = I\ddot{\boldsymbol{\theta}}(t)$$

| Symbol | Meaning |
|---|---|
| **T(t)** | Torque vector |
| **I** | Moment of inertia (scalar for spherical body) |
| **θ̈(t)** | Angular acceleration |
| **θ̇(t)** | Angular velocity |
| **θ(t)** | Orientation/angle |

**Three rotation axes (important for 3D systems):**
- **Roll θₓ** — rotation around x-axis
- **Yaw θᵧ** — rotation around y-axis (helicopter example uses this)
- **Pitch θ_z** — rotation around z-axis

---

## 1.3 The Helicopter Example (Canonical Exam Problem)

> **This example appears in almost every exam. Master it completely.**

**Physical setup:**
- Helicopter has a main rotor (which produces torque causing the body to spin)
- A tail rotor counteracts this spin
- **We model only the yaw (rotation around y-axis)**

**Simplification (Model Order Reduction):**
- Ignore position and roll/pitch
- Only consider yaw angle θᵧ
- The moment of inertia reduces to scalar Iᵧᵧ

**Resulting ODE (First-order):**
$$\ddot{\theta}_y(t) = \frac{T_y(t)}{I_{yy}}$$

**Integrating once:**
$$\dot{\theta}_y(t) = \dot{\theta}_y(0) + \frac{1}{I_{yy}}\int_0^t T_y(\tau)\,d\tau \tag{2.4}$$

**Actor model representation:**

```
Input: Ty(t)  ──[×(1/Iyy)]──[∫ + θ̇y(0)]──→  Output: θ̇y(t)
```

This is a **cascade** of two actors: a *Scale actor* (multiplies by 1/Iyy) and an *Integrator* (with initial condition θ̇y(0)).

---

## 1.4 Actor Models — Formal Definition

**Actor:** A box with input port(s) and output port(s) that represents a system function.

$$S: X \to Y$$

where X and Y are sets of continuous-time signals (functions R → R).

**Types of actor composition:**

| Composition Type | Description |
|---|---|
| **Cascade (Serial)** | Output of S1 feeds input of S2: y₁ = x₂ |
| **Parallel** | Both actors process the same input independently |
| **Feedback** | Output feeds back to input — creates directed cycle |
| **Adder** | y(t) = x₁(t) + x₂(t) or y(t) = x₁(t) - x₂(t) |

**Adder (subtractor) notation:**
```
x₁ ──→(+)──→ y = x₁ - x₂
x₂ ──→(-)
```

---

## 1.5 Properties of Systems

### 1.5.1 Causality

**Causal system:** Output at time τ depends only on inputs at time t ≤ τ (current and past).

$$x_1|_{t\le\tau} = x_2|_{t\le\tau} \implies S(x_1)|_{t\le\tau} = S(x_2)|_{t\le\tau}$$

**Strictly Causal:** Output at time τ depends only on inputs at time t < τ (strictly past).

> 🔑 The integrator is **strictly causal**. The adder is **causal but not strictly causal**.
> Strictly causal actors are required in every directed cycle of a feedback system to avoid instantaneous loops.

---

### 1.5.2 Memoryless Systems

**Memoryless:** Output depends only on the **current** input value.

Formally: ∃ function f : A → B such that (S(x))(t) = f(x(t)) for all t.

| Actor | Memory? |
|---|---|
| Adder: y = x₁ + x₂ | **Memoryless** |
| Integrator: y(t) = ∫x(τ)dτ | **Has memory** |
| Scale: y(t) = ax(t) | **Memoryless** |

> ⚠️ **Exam Trick:** A system that is both strictly causal AND memoryless always produces constant output (see Exercise 2 in the textbook).

---

### 1.5.3 Linearity

**Linear System:** Satisfies the superposition principle:

$$S(ax_1 + bx_2) = aS(x_1) + bS(x_2) \quad \forall\, x_1, x_2, a, b$$

| System | Linear? | Condition |
|---|---|---|
| Integrator (initial value = 0) | **Yes** | Must set i = 0 |
| Integrator (initial value ≠ 0) | **No** | Initial condition breaks linearity |
| Scale actor | **Always Yes** | No condition needed |
| Adder | **Always Yes** | No condition needed |
| Helicopter (θ̇y(0) = 0) | **Yes** | Initial velocity must be zero |

> 🔑 **Rule:** Any actor with a non-zero initial condition is generally **not linear**.

---

### 1.5.4 Time Invariance

**Time Invariant:** A delay in input produces the same delay in output.

$$S(D_\tau(x)) = D_\tau(S(x)) \quad \forall\, x, \tau$$

where D_τ(x)(t) = x(t - τ) is the delay actor.

**Example — Helicopter:**
The version with explicit initial condition θ̇y(0) is **NOT time invariant**.
The version integrated from -∞ (no explicit initial state) **IS time invariant**.

**LTI System:** Linear AND Time Invariant → Most amenable to analysis (Laplace transforms, Transfer Functions, etc.)

> 🎯 **Exam Focus:** "Is this system LTI?" always check: (1) Is initial condition zero? (2) Does the system change behavior if you shift the input in time?

---

## 1.6 BIBO Stability

### Definition (Lee & Seshia)

**BIBO Stable (Bounded-Input Bounded-Output):** A system is stable if every bounded input produces a bounded output.

- Input bounded: ∃ A < ∞ such that |w(t)| ≤ A for all t
- Output bounded: ∃ B < ∞ such that |v(t)| ≤ B for all t
- **Stable:** ∀ bounded inputs, ∃ bounded output

### Proving Instability — The Helicopter Example

**Input:** Unit step u(t) = {0, t < 0; 1, t ≥ 0} — this is **bounded** (never exceeds 1)

**Output:** The integral of a unit step grows without bound → θ̇y(t) → ∞

**Therefore:** The open-loop helicopter is **UNSTABLE**.

> 🔑 **Intuition:** An integrator fed a constant input will produce an ever-growing ramp output → UNSTABLE.

---

## 1.7 First-Order Linear ODEs — Solving Technique

**Standard form:**
$$\dot{y}(t) + ay(t) = bu(t)$$

**Solution (for step input u(t) = 1, t ≥ 0):**
$$y(t) = y(0)e^{-at} + \frac{b}{a}(1 - e^{-at})$$

**Two parts:**
1. **Homogeneous part:** y(0)e^{-at} (decays to zero if a > 0)
2. **Particular part:** (b/a)(1 - e^{-at}) (steady state = b/a)

### Steady-State Analysis

As t → ∞:
- If a > 0 → y(∞) = b/a (stable, converges to steady state)
- If a < 0 → y(t) → ∞ (unstable)
- If a = 0 → integrator behavior (marginally stable/unstable)

---

## 1.8 Laplace Transform — Converting to s-Domain

> **Why Laplace?** Differential equations in time domain are hard to solve. Laplace transforms convert them to *algebraic equations* in the s-domain.

### Key Laplace Transform Pairs

| Time Domain | s-Domain | Condition |
|---|---|---|
| f(t) | F(s) | — |
| df/dt | sF(s) - f(0⁻) | — |
| d²f/dt² | s²F(s) - sf(0⁻) - f'(0⁻) | — |
| ∫₀ᵗ f(τ)dτ | F(s)/s | — |
| e^(-at) | 1/(s+a) | a > 0 |
| 1 (unit step) | 1/s | t ≥ 0 |
| δ(t) (impulse) | 1 | — |

### Transfer Function (s-Domain Model)

**Definition:** The transfer function H(s) is the ratio of the Laplace transform of the output to the Laplace transform of the input, assuming **zero initial conditions**.

$$H(s) = \frac{Y(s)}{U(s)} \bigg|_{\text{zero IC}}$$

**Steps to find Transfer Function:**
1. Write the ODE
2. Take Laplace transform of both sides (set ICs = 0)
3. Rearrange to get Y(s)/U(s)

### Worked Example — Transfer Function of Helicopter

**ODE:** $\ddot{\theta}_y = T_y/I_{yy}$

**With initial conditions zero:**
$$s\dot{\Theta}_y(s) = T_y(s)/I_{yy}$$
$$H(s) = \frac{\dot{\Theta}_y(s)}{T_y(s)} = \frac{1}{I_{yy} \cdot s}$$

This is an **integrator** in the s-domain (1/s = integrator).

---

## 1.9 State-Space Representation

**State-Space Form** expresses a system as a set of first-order ODEs:

$$\dot{\mathbf{x}}(t) = A\mathbf{x}(t) + B\mathbf{u}(t)$$
$$\mathbf{y}(t) = C\mathbf{x}(t) + D\mathbf{u}(t)$$

| Matrix | Size | Meaning |
|---|---|---|
| **A** | n×n | System/state matrix |
| **B** | n×m | Input matrix |
| **C** | p×n | Output matrix |
| **D** | p×m | Feedthrough matrix |
| **x** | n×1 | State vector |
| **u** | m×1 | Input vector |
| **y** | p×1 | Output vector |

### Converting Higher-Order ODE to State-Space

**For a 2nd-order ODE:** $\ddot{y} + a_1\dot{y} + a_0 y = b_0 u$

**Step 1 — Assign states:**
- x₁ = y (output position)
- x₂ = ẏ (output velocity)

**Step 2 — Write as first-order system:**
- ẋ₁ = x₂
- ẋ₂ = -a₀x₁ - a₁x₂ + b₀u

**Step 3 — Matrix form:**
$$\dot{\mathbf{x}} = \begin{bmatrix} 0 & 1 \\ -a_0 & -a_1 \end{bmatrix}\mathbf{x} + \begin{bmatrix} 0 \\ b_0 \end{bmatrix}u$$
$$y = \begin{bmatrix} 1 & 0 \end{bmatrix}\mathbf{x}$$

### Transfer Function from State-Space

$$H(s) = C(sI - A)^{-1}B + D$$

---

## 1.10 System Stability (State-Space View)

### Eigenvalue / Pole Criterion

The stability of a system is determined by the **eigenvalues of the A matrix** (also called the **poles** of the transfer function):

| Pole Location | System Behavior |
|---|---|
| All poles have **negative real parts** (Re(λ) < 0) | **Asymptotically Stable** ✅ |
| Any pole has **positive real part** (Re(λ) > 0) | **Unstable** ❌ |
| Poles on **imaginary axis** (Re(λ) = 0) | **Marginally stable** ⚠️ |

**Steps to determine stability:**
1. Find eigenvalues of A (or poles of H(s))
2. Check real parts
3. If all negative → stable

### Worked Example — Checking Stability

Given: $A = \begin{bmatrix} 0 & 1 \\ -2 & -3 \end{bmatrix}$

**Characteristic equation:** det(λI - A) = 0

$$\det\begin{bmatrix} \lambda & -1 \\ 2 & \lambda+3 \end{bmatrix} = \lambda(\lambda+3) + 2 = \lambda^2 + 3\lambda + 2 = 0$$

**Roots:** λ = -1, λ = -2

Both negative → **System is asymptotically stable** ✅

---

## 1.11 ✏️ Solved Numerical Problems

### Problem 1 — Find Transfer Function

**Given ODE:** $\ddot{y}(t) + 5\dot{y}(t) + 6y(t) = 3u(t)$, zero initial conditions.

**Solution:**
1. Laplace transform: $s^2Y(s) + 5sY(s) + 6Y(s) = 3U(s)$
2. Factor: $Y(s)(s^2 + 5s + 6) = 3U(s)$
3. Transfer Function: $H(s) = \frac{3}{s^2 + 5s + 6} = \frac{3}{(s+2)(s+3)}$

**Stability check:** Poles at s = -2 and s = -3 → Both negative real parts → **STABLE** ✅

---

### Problem 2 — State-Space Conversion

**Convert to state-space:** $\ddot{y} + 4\dot{y} + 3y = u$

**Step 1:** Let x₁ = y, x₂ = ẏ

**Step 2:** ẋ₁ = x₂; ẋ₂ = -3x₁ - 4x₂ + u

**Step 3:**
$$A = \begin{bmatrix}0 & 1\\-3 & -4\end{bmatrix}, \quad B = \begin{bmatrix}0\\1\end{bmatrix}, \quad C = \begin{bmatrix}1 & 0\end{bmatrix}, \quad D = 0$$

**Eigenvalues:** λ² + 4λ + 3 = 0 → λ = -1, -3 → **STABLE** ✅

---

### Problem 3 — Steady State Value

**ODE:** $\dot{y}(t) + 2y(t) = 4$, y(0) = 1

**Solution:** y(t) = y(0)e^(-2t) + (4/2)(1 - e^(-2t)) = e^(-2t) + 2(1 - e^(-2t))

$$y(t) = 2 - e^{-2t}$$

**Steady state:** y(∞) = **2** (as t → ∞, exponential decays to 0)

---

## 1.12 📝 Common Exam Questions

1. "Write the differential equation for the helicopter yaw model."
2. "Convert the given ODE to its Laplace domain transfer function."
3. "Is the system stable? Justify using pole locations."
4. "Convert the second-order ODE to state-space form."
5. "Find the steady-state output of the system."
6. "Is the integrator actor memoryless? Is it causal?"
7. "Show all derivation steps for the solution of the ODE."

---

## 1.13 ⚠️ Common Student Mistakes

- **Forgetting initial conditions** in Laplace transform: always include `sF(s) - f(0⁻)` for derivatives
- **Not showing steps** in ODE solutions — partial marks require visible methodology
- **Confusing causality and stability** — they are independent properties
- **Forgetting to set ICs to zero** when computing Transfer Functions
- **Not checking eigenvalue signs** — negative real part = stable, not negative value

---

## 1.14 🔁 Unit 1 — Rapid Revision Notes

```
CONTINUOUS DYNAMICS — CHEAT SHEET
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Newton:       F = Mẍ     (translational)
              T = Iθ̈     (rotational, spherical)

Helicopter ODE: θ̈y = Ty/Iyy
Integrated:    θ̇y(t) = θ̇y(0) + (1/Iyy)∫₀ᵗ Ty(τ)dτ

Laplace:      d/dt ↔ s    ∫ ↔ 1/s    (with zero ICs)
Transfer Fn:  H(s) = Y(s)/U(s) | zero ICs

State-Space:  ẋ = Ax + Bu
              y = Cx + Du

Stability:    All eigenvalues of A have Re(λ) < 0 → STABLE
              BIBO: Bounded input → Bounded output

LTI:          Linear (superposition) + Time Invariant (delay commutes)
```

---

---

# ═══════════════════════════════════════════════
# UNIT 2 — FEEDBACK CONTROL & STABILITY
## Source: Lee & Seshia, Ch.2 (pp.31–37)
# ═══════════════════════════════════════════════

---

## 2.1 Introduction — Why Feedback?

**Recall:** The open-loop helicopter is **unstable**. Without control, the helicopter body spins forever.

**Feedback control** uses the *error* between desired behavior and actual behavior to *automatically correct* that behavior. It's the fundamental concept behind autopilots, thermostats, cruise control, and industrial robots.

**The Big Picture:**

```
                     ┌─────────────────────────────┐
Desired output (ψ) → │ Error Signal (e = ψ - θ̇y)  │
                     │ ↓                            │
                     │ Controller (K)               │
                     │ ↓                            │
                     │ Control Input (Ty = Ke)      │
                     │ ↓                            │
                     │ Plant (Helicopter: 1/Iyy·s)  │
                     │ ↓                            │
                     └──────────────────────────────┘
Actual output (θ̇y) ←──── feedback ─────────────────┘
```

---

## 2.2 Proportional Control — Formal Setup

**System equations (from Figure 2.3 in Lee & Seshia):**

$$e(t) = \psi(t) - \dot{\theta}_y(t) \quad \text{(error = desired - actual)}$$
$$T_y(t) = K \cdot e(t) \quad \text{(control input proportional to error)}$$

Substituting into helicopter model:

$$\dot{\theta}_y(t) = \dot{\theta}_y(0) + \frac{1}{I_{yy}}\int_0^t T_y(\tau)\,d\tau = \dot{\theta}_y(0) + \frac{K}{I_{yy}}\int_0^t (\psi(\tau) - \dot{\theta}_y(\tau))\,d\tau$$

---

## 2.3 Case 1 — Stabilization (ψ = 0, Stop Helicopter Spinning)

**Goal:** Keep the helicopter from rotating at all → ψ(t) = 0

**Equation becomes:**
$$\dot{\theta}_y(t) = \dot{\theta}_y(0) - \frac{K}{I_{yy}}\int_0^t \dot{\theta}_y(\tau)\,d\tau \tag{2.11}$$

### Solving the Integral Equation

Using calculus identity: $\int_0^t ae^{a\tau}d\tau = e^{at}u(t) - 1$

**Solution:**
$$\boxed{\dot{\theta}_y(t) = \dot{\theta}_y(0)\,e^{-Kt/I_{yy}}\,u(t)} \tag{2.12}$$

### Stability Analysis of the Closed Loop

| Condition | Behavior |
|---|---|
| **K > 0** | Exponential decay → θ̇y → 0 ✅ **STABLE** |
| **K > 0, large** | Faster decay (approaches desired value more quickly) |
| **K < 0** | Exponential growth → θ̇y → ∞ ❌ **UNSTABLE** |
| **K = 0** | No control applied → helicopter still spins ❌ |

> 🔑 **Key Insight:** Feedback with positive gain K **stabilizes** the unstable helicopter plant.

---

## 2.4 Case 2 — Tracking (ψ(t) = au(t), Non-Zero Desired Speed)

**Goal:** Make the helicopter reach a desired angular velocity a.

**Initial conditions:** θ̇y(0) = 0

**Derivation:**
$$\dot{\theta}_y(t) = \frac{K}{I_{yy}}\left[\int_0^t a\,d\tau - \int_0^t \dot{\theta}_y(\tau)\,d\tau\right]$$
$$= \frac{K \cdot at}{I_{yy}} - \frac{K}{I_{yy}}\int_0^t \dot{\theta}_y(\tau)\,d\tau$$

**Solution:**
$$\boxed{\dot{\theta}_y(t) = a\,u(t)\left(1 - e^{-Kt/I_{yy}}\right)} \tag{2.13}$$

### Analysis of the Tracking Solution

| Term | Meaning |
|---|---|
| a · u(t) | Desired output |
| -a·e^(-Kt/Iyy) | **Tracking error** — decays to zero |
| Full expression → a as t → ∞ | Asymptotically tracks the desired velocity |

> 🔑 **Tracking error** approaches zero as t → ∞ for K > 0 → The system **tracks** the desired output.

---

## 2.5 Case 3 — Disturbance Rejection (Two-Input Helicopter)

**Physical reality:** Ty = Tt + Tr (torque from top rotor + tail rotor)

**Setup:** External disturbance Tt = bu(t), controller only commands Tr.

Using algebraic transformation (K·a₁ + a₂ = K·(a₁ + a₂/K)):

**Equivalent input to the closed-loop system:**
$$x(t) = \psi(t) + T_t(t)/K = (b/K)u(t)$$

**Solution (from tracking formula):**
$$\dot{\theta}_y(t) = (b/K)u(t)\left(1 - e^{-Kt/I_{yy}}\right) \tag{2.14}$$

**Key observation:** As K → ∞, (b/K) → 0, so the disturbance effect vanishes!
Higher controller gain K → Better disturbance rejection.

---

## 2.6 Transfer Function of Closed-Loop System

For the proportional controller feedback loop:

**Open-loop transfer function (plant):** $G(s) = \frac{1}{I_{yy}\cdot s}$

**Controller:** C(s) = K

**Closed-loop transfer function:**
$$T_{cl}(s) = \frac{KG(s)}{1 + KG(s)} = \frac{K/I_{yy}\cdot s}{1 + K/I_{yy}\cdot s} = \frac{K/I_{yy}}{s + K/I_{yy}}$$

**Pole of closed loop:** $s = -K/I_{yy}$

- If K > 0 → pole at negative real → **STABLE** ✅
- Larger K → pole further left → faster response

---

## 2.7 ✏️ Solved Numerical Problems

### Problem 4 — Verify Closed-Loop Solution

**Verify that θ̇y(t) = θ̇y(0)·e^(-Kt/Iyy) satisfies equation (2.11).**

**LHS:** θ̇y(t) = θ̇y(0)e^(-Kt/Iyy)

**RHS:** θ̇y(0) - (K/Iyy)∫₀ᵗ θ̇y(0)e^(-Kτ/Iyy) dτ

$$= \dot{\theta}_y(0) - \frac{K}{I_{yy}}\cdot\dot{\theta}_y(0)\cdot\left[\frac{-I_{yy}}{K}e^{-K\tau/I_{yy}}\right]_0^t$$

$$= \dot{\theta}_y(0) - \dot{\theta}_y(0)\left[-e^{-Kt/I_{yy}} + 1\right]$$

$$= \dot{\theta}_y(0) - \dot{\theta}_y(0) + \dot{\theta}_y(0)e^{-Kt/I_{yy}} = \dot{\theta}_y(0)e^{-Kt/I_{yy}} = \text{LHS} \checkmark$$

---

### Problem 5 — Design a Controller

**Given:** Plant G(s) = 1/(s+1). Design proportional controller K such that the closed-loop system settles (poles have real parts ≤ -3).

**Closed-loop pole:** $s = -1 - K$

**Requirement:** $-1 - K \le -3 \implies K \ge 2$

**Answer:** K ≥ 2 ensures settling.

---

## 2.8 📝 Common Exam Questions — Feedback Control

1. "Draw the feedback control block diagram for the helicopter."
2. "Show that proportional control stabilizes the helicopter. Derive the expression for θ̇y(t)."
3. "For K > 0, prove that the tracking error decays to zero."
4. "Find the closed-loop transfer function given plant G(s) and controller K."
5. "What happens to system behavior as K increases?"
6. "What is the steady-state value of θ̇y(t) for ψ(t) = au(t)?"

---

## 2.9 🔁 Unit 2 — Rapid Revision Notes

```
FEEDBACK CONTROL — CHEAT SHEET
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Block diagram:   ψ → (+/-) → e → [K] → Ty → [Helicopter] → θ̇y ─┐
                               └─────────────────────────────────┘

Proportional:    Ty(t) = K·e(t) = K·(ψ(t) - θ̇y(t))

Stabilization:   θ̇y(t) = θ̇y(0)·exp(-Kt/Iyy)  → 0 if K > 0
Tracking:        θ̇y(t) = a(1 - exp(-Kt/Iyy))  → a if K > 0

Closed-loop TF:  T_cl(s) = KG(s)/(1 + KG(s))
Closed-loop pole: s = -K/Iyy → stable if K > 0

Larger K → Faster response, better disturbance rejection
Negative K → Instability
```

---

---

# ═══════════════════════════════════════════════
# UNIT 3 — ASYNCHRONOUS COMPOSITION & INTERLEAVING SEMANTICS
## Source: Lee & Seshia Ch.5 (pp.107–120) + Alur Ch.4 (pp.125–141)
# ═══════════════════════════════════════════════

---

## 3.1 Introduction — Why Asynchronous Composition?

**Synchronous composition:** Both machines react simultaneously (like two musicians playing in perfect sync). Easy to reason about.

**Asynchronous composition:** Machines react independently at their own pace (like two workers in an office who don't always coordinate). More realistic for distributed, networked, or multi-processor CPS.

**Real-world examples:**
- A sensor sending measurements at irregular intervals
- Two microcontrollers communicating over a network with variable delay
- A PLC controller and a field sensor operating independently

---

## 3.2 Synchronous Composition (Baseline for Comparison)

**Formal definition (Lee & Seshia):**

Given FSMs A and B:
$$\text{States}_C = \text{States}_A \times \text{States}_B \quad \text{(Cartesian product)}$$
$$\text{Inputs}_C = \text{Inputs}_A \times \text{Inputs}_B$$
$$\text{Outputs}_C = \text{Outputs}_A \times \text{Outputs}_B$$
$$\text{initialState}_C = (\text{initialState}_A, \text{initialState}_B)$$

**Update function:**
$$\text{update}_C((s_A, s_B), (i_A, i_B)) = ((s'_A, s'_B), (o_A, o_B))$$
where $(s'_A, o_A) = \text{update}_A(s_A, i_A)$ and $(s'_B, o_B) = \text{update}_B(s_B, i_B)$.

> **Both A and B react simultaneously in every reaction of C.**

**Key property:** If A and B are both deterministic, the synchronous composition C is also **deterministic** (determinism is compositional).

---

## 3.3 Asynchronous Composition — The Core Concept

In asynchronous composition, component machines react **independently**. The key design decision: *what is a reaction of the composite machine C?*

### Four Possible Semantics (Lee & Seshia)

| Semantics | Who Reacts? | How Chosen? |
|---|---|---|
| **Semantics 1** | A **or** B | **Nondeterministically** |
| **Semantics 2** | A, B, **or both** | **Nondeterministically** |
| **Semantics 3** | A **or** B | **Environment chooses** |
| **Semantics 4** | A, B, **or both** | **Environment chooses** |

---

### 3.3.1 Interleaving Semantics (Semantics 1)

> **"Semantics 1 is referred to as an interleaving semantics, meaning that A or B never react simultaneously." — Lee & Seshia, p.112**

**Key properties of Interleaving:**
- At each reaction, exactly ONE machine reacts
- The choice is **nondeterministic** — either A or B
- The other machine stays in its current state, producing **absent** outputs

**Formal update function (Interleaving):**

$$\text{update}_C((s_A, s_B), (i_A, i_B)) = ((s'_A, s'_B), (o'_A, o'_B))$$

**Either** (A reacts, B stays):
$$s'_A = \text{update}_A(s_A, i_A), \quad s'_B = s_B, \quad o'_B = \text{absent}$$

**Or** (B reacts, A stays):
$$s'_B = \text{update}_B(s_B, i_B), \quad s'_A = s_A, \quad o'_A = \text{absent}$$

---

### 3.3.2 Critical Subtlety — Input Events Can Be Missed!

> ⚠️ **Important Exam Point:**

If machine A receives an input event, but B is nondeterministically chosen to react first, then **A misses the input event**!

**Consequence:** In a CPS with irregular sensor inputs, the wrong scheduling choice can cause the controller to miss a critical sensor reading.

**Solution options:**
- Use synchronous composition (guaranteed no events missed)
- Use Semantics 3/4 (environment controls scheduling)
- Employ a priority-based scheduler

---

### 3.3.3 Interleaving vs. Synchronous — Head-to-Head

| Property | Synchronous Composition | Interleaving Semantics |
|---|---|---|
| **Both react per step** | Yes (always) | No (one at a time) |
| **Determinism preserved** | Yes (compositional) | No |
| **Events missed** | Never | Possible |
| **State space size** | |StatesA| × |StatesB| | Same, but more transitions |
| **Use case** | Tight coordination | Distributed, independent |

---

## 3.4 Asynchronous Processes (Alur's Model — Chapter 4)

### 3.4.1 Formal Model

An **asynchronous process P** consists of:
- **I** — finite set of input channels (inputs: x?v = receive value v on channel x)
- **O** — finite set of output channels (outputs: y!v = send value v on channel y)
- **S** — finite set of typed state variables
- **Init** — initialization
- **Input tasks** — for each channel x: Guard → Update
- **Output tasks** — for each channel y: Guard → Update
- **Internal tasks** — no I/O: Guard → Update

### 3.4.2 Three Task Types

| Task Type | Notation | Trigger | I/O Effect |
|---|---|---|---|
| **Input task** | s →(x?v) t | Receives value v on channel x | Updates state; no output |
| **Output task** | s →(y!v) t | Guard satisfied; produces output | Updates state; sends output |
| **Internal task** | s →(ε) t | Guard satisfied | Updates state; no I/O |

> 🔑 **ε (epsilon)** means "no observable communication" — the action is invisible to the outside world.

### 3.4.3 The Buffer Process Example

```
{0, 1, null} x := null
bool in ──→  [Ai: x := in]          ──→  bool out
             [Ao: x≠null → {out:=x; x:=null}]
```

**Ai (Input task):** Always enabled; copies received value into internal buffer x.
**Ao (Output task):** Guard = (x ≠ null); sends buffered value out; resets buffer.

**Key behaviors:**
- The process can accept multiple inputs before producing output (buffer accumulates)
- At most one value is buffered (size-1 buffer)
- If a new input arrives while buffer is non-empty → old value is **overwritten and lost**

---

## 3.5 Executions and Interleaving Semantics (Alur)

### Formal Definition of Execution

A **finite execution** is:
$$s_0 \xrightarrow{l_1} s_1 \xrightarrow{l_2} s_2 \xrightarrow{l_3} \cdots \xrightarrow{l_k} s_k$$

where each lᵢ is an input action, output action, or internal action.

**Interleaving Semantics:** The order of task execution is **totally unconstrained**. At each step, any enabled task can fire.

### AsyncInc Example

```
nat x := 0; y := 0
Ax : x := x + 1    (internal, always enabled)
Ay : y := y + 1    (internal, always enabled)
```

From initial state (0,0), reachable states include: (1,0), (0,1), (2,0), (1,1), (0,2)...

**All states (i, j) for i,j ≥ 0 are reachable** — the nondeterministic scheduler can choose any ordering of Ax and Ay.

> 🎯 **Exam Focus:** "List all possible execution sequences" → systematically enumerate all orderings using the interleaving semantics.

---

## 3.6 Parallel Composition of Asynchronous Processes (Alur)

### Composition Rules

When composing P1 | P2:

| Case | Rule |
|---|---|
| **Channel x = output of P1, input of P2** | P1 and P2 **synchronize** — output of P1 triggers matching input of P2 as one joint action |
| **Channel x = input of both P1 and P2** | Both process it simultaneously; combined guard = Guard1 ∧ Guard2 |
| **Channel only in P1 (not P2)** | P1 acts alone; P2 state unchanged |
| **Internal tasks** | Each component's internal tasks remain independent; other component unchanged |

**No cyclic dependencies possible** (unlike synchronous composition), because producing output and receiving input are separate steps.

---

## 3.7 Sequence Detection FSMs

> **Exam-critical: you may need to design and trace an FSM that detects a specific input pattern.**

### Design Method

1. **Identify the target sequence** (e.g., "detect 1-0-1" in a bit stream)
2. **Handle overlapping:** Can the start of the next match overlap with the end of the current match?
3. **Define states:** Usually one state per "prefix matched so far"
4. **Define transitions:** For each state and input, determine next state and output
5. **Label all transitions with `Input/Output` syntax** — EXAM REQUIREMENT

### Example — Detect "01" Sequence

**States:** 
- S0: No progress (initial)
- S1: Saw '0' (first character matched)
- S2: Saw '01' (full match — output = 1)

**Transitions:**
```
S0 → S0: input=1, output=0  (reset, wrong start)
S0 → S1: input=0, output=0  (first character matched)
S1 → S1: input=0, output=0  (stay, got another 0)
S1 → S2: input=1, output=1  (MATCH! output 1)
S2 → S1: input=0, output=0  (overlapping - new match could start)
S2 → S0: input=1, output=0  (no overlap possible)
```

---

## 3.8 Task Dependency Graphs (DAGs)

### What is a DAG?

A **Directed Acyclic Graph (DAG)** where:
- **Nodes** = individual tasks
- **Directed edges** = precedence constraints (A → B means "A must complete before B starts")
- **Acyclic** = no circular dependencies

### Converting Text Constraints to DAG

**Given:** "Task C can only start after both Task A and Task B have completed. Task D requires Task C."

**DAG:**
```
A ──→ C ──→ D
B ──→ C
```

### Topological Sort

A **topological ordering** is any sequence where every task appears after all its prerequisites.

**Algorithm:**
1. Find all nodes with in-degree 0 (no prerequisites) → these can go first
2. Remove those nodes and their outgoing edges
3. Repeat until all nodes are ordered

**Example:** For the DAG A→C, B→C, C→D:

Valid topological orderings:
- A, B, C, D ✅
- B, A, C, D ✅

Invalid:
- C, A, B, D ❌ (C requires A and B first)
- A, C, B, D ❌ (B hasn't finished before C starts)

---

## 3.9 ✏️ Solved Problems — Interleaving

### Problem 6 — Enumerate Execution Sequences

**Two tasks T1 (period 2) and T2 (period 3). List all valid execution interleavings for the first 5 steps assuming both are always enabled.**

T1 and T2 can execute in any order at each step.

Sample valid interleavings (5 steps):
1. T1, T1, T2, T1, T1
2. T2, T1, T2, T1, T2
3. T1, T2, T1, T2, T1
4. T2, T2, T1, T1, T2
... (many more valid orderings)

**Total orderings:** C(5,k) for each k = number of T2 executions — but with interleaving, all orderings are valid.

---

## 3.10 🔁 Unit 3 — Rapid Revision Notes

```
ASYNC COMPOSITION & INTERLEAVING — CHEAT SHEET
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Synchronous:  Both A and B react simultaneously every step
Interleaving: A or B reacts (nondeterministic choice) each step

Semantics 1 = Interleaving (nondeterministic)
Semantics 2 = A, B, or both react (nondeterministic)
Semantics 3 = A or B (environment chooses) ← also interleaving
Semantics 4 = A, B, or both (environment chooses)

KEY: Interleaving → events can be MISSED (dangerous in CPS!)
KEY: Sync composition preserves determinism; async does NOT

Async Process:
  Input task:    x?v → update state (no output)
  Output task:   y!v → update state + send value
  Internal task: ε   → update state (no I/O)

Execution: s₀ →(l₁) s₁ →(l₂) s₂ ... (any enabled task fires)

FSM rule: ALWAYS write Input/Output on EVERY transition arrow
DAG: Tasks as nodes, precedence as directed edges, topological sort = valid execution order
```

---

---

# ═══════════════════════════════════════════════
# UNIT 4 — HARD vs. SOFT REAL-TIME SYSTEMS
## Source: Lee & Seshia Ch.12 (pp.319–334) + Alur Ch.8
# ═══════════════════════════════════════════════

---

## 4.1 Introduction — What is a Real-Time System?

**Real-time system:** A system where **timing is a correctness requirement**, not just a performance metric.

> "Tasks have deadlines, which are values of physical time by which the task must be completed." — Lee & Seshia

**Key difference from normal computing:** A slow answer in a real-time system isn't just inconvenient — it can be **wrong** or even **dangerous**.

---

## 4.2 Definitions (Lee & Seshia)

### 4.2.1 Deadline

**Deadline (dᵢ):** The time by which task execution i must be completed.

$$f_i \le d_i \quad \text{(feasibility condition)}$$

### 4.2.2 Hard Deadline (Hard Real-Time)

> "Sometimes, a deadline is a real physical constraint imposed by the application, where **missing the deadline is considered an error**. Such a deadline is called a **hard deadline**. Scheduling with hard deadlines is called **hard real-time scheduling**." — Lee & Seshia, p.322

**Examples of hard real-time systems:**
- **Automotive airbag controller** — must detect crash and deploy within 30ms; any delay = fatality
- **Aircraft fly-by-wire** — control surfaces must respond within strict time; any miss = crash
- **Cardiac pacemaker** — must deliver impulse at precise timing; late = arrhythmia
- **Anti-lock braking system (ABS)** — must respond within milliseconds; miss = skid
- **Industrial robot arm** — must stop within hard deadline to avoid collision

### 4.2.3 Soft Deadline (Soft Real-Time)

> "Often, a deadline reflects a design decision that need not be enforced strictly. It is **better to meet the deadline, but missing the deadline is not an error**. Generally it is better to not miss the deadline by much. This case is called **soft real-time scheduling**." — Lee & Seshia, p.322

**Examples of soft real-time systems:**
- **Video streaming** — dropping a frame is annoying but not catastrophic
- **Audio playback** — slight buffer underrun causes a click but not a disaster
- **Smartphone UI** — occasional slow response is frustrating but not dangerous
- **Online game** — higher latency degrades experience but isn't a safety issue

---

## 4.3 Task Timing Parameters (Lee & Seshia Figure 12.1)

```
Timeline:
─────────────────────────────────────────────────────→ time
     rᵢ       sᵢ      fᵢ       dᵢ
     │         │───────│         │
     │         execution         deadline
     │←──────oᵢ────────────────→│
             (response time)
```

| Parameter | Symbol | Definition |
|---|---|---|
| **Release time / Arrival time** | rᵢ | Earliest time the task is enabled |
| **Start time** | sᵢ | Time execution actually begins; sᵢ ≥ rᵢ |
| **Finish time** | fᵢ | Time execution completes; fᵢ ≥ sᵢ |
| **Response time** | oᵢ | oᵢ = fᵢ - rᵢ (total time from release to finish) |
| **Execution time** | eᵢ | Actual processing time (excludes blocking/preemption) |
| **Worst-Case Execution Time** | WCET | Upper bound on eᵢ |
| **Deadline** | dᵢ | Must have fᵢ ≤ dᵢ for hard real-time |

---

## 4.4 Scheduling Concepts

### Types of Schedulers

| Scheduler Type | Assignment | Ordering | Timing | Notes |
|---|---|---|---|---|
| **Fully Static** | Design time | Design time | Design time | Most predictable; difficult with variable execution times |
| **Static Order** | Design time | Design time | Run time | Off-line scheduler |
| **Static Assignment** | Design time | Run time | Run time | Each processor has fixed task set |
| **Fully Dynamic** | Run time | Run time | Run time | On-line scheduler; most flexible |

### Preemptive vs. Non-preemptive

| | Preemptive | Non-preemptive |
|---|---|---|
| **Definition** | Higher-priority task can interrupt currently running task | Running task completes before next task starts |
| **Pro** | Better deadline compliance, more flexible | Simpler, lower overhead |
| **Con** | Context-switch overhead, priority inversion risks | May miss deadlines if current task runs too long |

### Comparing Schedulers

**Maximum lateness:**
$$L_{max} = \max_{i \in T}(f_i - d_i)$$

- Lₘₐₓ ≤ 0 → All deadlines met (feasible schedule)
- Lₘₐₓ > 0 → Some deadlines missed; for soft real-time, minimize this value

**Processor Utilization:**
$$U = \sum_{J \in \mathcal{J}} \frac{\eta(J)}{\pi(J)}$$

- 100% means no idle time
- A scheduler is **optimal** if it delivers a feasible schedule whenever U ≤ 1

---

## 4.5 Hard Real-Time vs. Soft Real-Time — Complete Comparison

| Feature | Hard Real-Time | Soft Real-Time |
|---|---|---|
| **Deadline violation** | UNACCEPTABLE (system error) | Acceptable (degraded quality) |
| **Design approach** | Worst-case analysis (WCET) | Average-case optimization |
| **Scheduling** | Deterministic, provably correct | Best-effort |
| **Typical metric** | Zero missed deadlines | Minimize lateness or missed % |
| **Examples** | Airbag, pacemaker, ABS, avionics | Video player, online game, UI |
| **Cost of failure** | Potentially catastrophic | User inconvenience |
| **Verification** | Formal proofs, schedulability tests | Statistical measurement |

> 🔑 **Exam must-know:** "A pacemaker is hard real-time because missing the electrical impulse deadline can cause fatal arrhythmia." Always give a specific, technical justification.

---

## 4.6 Periodic Task Model (from Alur)

A **periodic job J** is specified by:

$$(\eta(J),\; \delta(J),\; \pi(J))$$

| Parameter | Symbol | Meaning |
|---|---|---|
| **Period** | π(J) | Time between successive arrivals |
| **Deadline** | δ(J) | Max time from arrival to completion |
| **WCET** | η(J) | Worst-case execution time |
| **Constraint** | | η(J) ≤ δ(J) ≤ π(J) |

**Arrival time of ath instance:** α(J,a) = (a-1)·π(J)
**Absolute deadline of ath instance:** δ(J,a) = (a-1)·π(J) + δ(J)

---

## 4.7 🔁 Unit 4 — Rapid Revision Notes

```
HARD vs. SOFT REAL-TIME — CHEAT SHEET
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Hard Real-Time:  Missing deadline = ERROR (safety-critical)
Soft Real-Time:  Missing deadline = degraded quality (not error)

Task parameters:
  rᵢ = release time     sᵢ = start time
  fᵢ = finish time      dᵢ = deadline
  eᵢ = execution time   oᵢ = response time = fᵢ - rᵢ
  WCET = worst-case execution time

Feasible schedule: fᵢ ≤ dᵢ  for all tasks
Max lateness:      Lmax = max(fᵢ - dᵢ) ≤ 0 for feasible schedule

Periodic task: (η, δ, π) where η ≤ δ ≤ π
  Implicit deadline: δ = π
  Utilization: U = Σ η(J)/π(J)
```

---

---

# ═══════════════════════════════════════════════
# UNIT 5 — REAL-TIME SCHEDULING: RMS & EDF
## Source: Alur Ch.8 (pp.339–378) + Lee & Seshia Ch.12
# ═══════════════════════════════════════════════

---

## 5.1 Schedulability

**A job model is schedulable** if there exists a deadline-compliant schedule σ.

**Utilization check (first test):**
$$U = \sum_{J \in \mathcal{J}} \frac{\eta(J)}{\pi(J)}$$

If U > 1 → **IMMEDIATELY INFEASIBLE** (no policy can work).

---

## 5.2 Earliest Deadline First (EDF)

### 5.2.1 Rule

> At every time slot t, assign the processor to the **ready job with the SMALLEST current absolute deadline**.

**"Ready" at time t:** Active instance has not yet received all η(J) time slots.

### 5.2.2 Step-by-Step Construction

1. Calculate arrival time and absolute deadline for each active instance at time t
2. List all ready (unfinished) jobs with their current deadlines
3. Pick the job with smallest deadline (tie-break: any consistent rule, e.g., lower job ID)
4. Advance time by 1 slot
5. Check for new arrivals → if a new job with earlier deadline arrives → preempt current job
6. Repeat

### 5.2.3 Optimality of EDF (Theorem 8.2)

> **If a periodic job model is schedulable, and σ is an EDF schedule, then σ is deadline-compliant.**

EDF is **optimal** — if any scheduling policy can meet all deadlines, EDF can too.

### 5.2.4 EDF Schedulability Condition (Implicit Deadlines)

$$\boxed{U \le 1 \iff \text{schedulable by EDF}}$$

For implicit deadlines (δ = π): necessary and sufficient condition.

---

## 5.3 Rate Monotonic Scheduling (RMS)

### 5.3.1 Rule

> **Assign fixed priorities: shorter period = higher priority.**
> At each step, run the highest-priority ready job.
> Preempt lower-priority job if higher-priority job becomes ready.

Formally: if π(J) < π(K), then ρ(J) > ρ(K).

### 5.3.2 Liu & Layland Utilization Bound (Theorem 8.6)

$$\boxed{B_n = n(2^{1/n} - 1)}$$

> If $U \le B_n$, then RMS is **guaranteed** to produce a deadline-compliant schedule.

| n (tasks) | Bound Bₙ |
|---|---|
| 1 | 1.000 |
| 2 | 0.828 |
| 3 | 0.780 |
| 4 | 0.757 |
| 5 | 0.743 |
| 6 | 0.735 |
| 7 | 0.729 |
| 8 | 0.724 |
| 9 | 0.721 |
| 10 | 0.718 |
| **∞** | **ln 2 ≈ 0.693** |

**Practical rule:** If U ≤ 0.69, RMS always works regardless of number of tasks.

### 5.3.3 Two Schedulability Zones for RMS

```
U-axis: 0────────────────Bₙ──────────1.0────────→
        │                │            │
        │  Deterministic │  Fallback  │  Infeasible
        │  (guaranteed)  │  (check!)  │  (any policy)
```

- **U ≤ Bₙ (Deterministic zone):** RMS definitely works
- **Bₙ < U ≤ 1 (Pessimistic fallback zone):** RMS *might* work — must verify by constructing schedule
- **U > 1 (Infeasible):** No policy works

---

## 5.4 ✏️ Full Gantt Chart Construction — Worked Examples

### Example 1 — EDF Scheduling

**Given jobs:**
- J1: π=5, δ=4, η=3 (period 5, deadline 4, WCET 3)
- J2: π=3, δ=3, η=1 (period 3, deadline 3, WCET 1)

**Step 1: Compute Utilization**
$$U = \frac{3}{5} + \frac{1}{3} = 0.6 + 0.333 = 0.933$$

U < 1 → Potentially schedulable by EDF.

**Step 2: List absolute deadlines for each instance**

| Time | J1 instance | J1 deadline | J2 instance | J2 deadline |
|---|---|---|---|---|
| 0 | 1 | 4 | 1 | 3 |
| 3 | 1 (cont.) | 4 | 2 | 6 |
| 5 | 2 | 9 | 2 (cont.) | 6 |
| 6 | 2 | 9 | 3 | 9 |

**Step 3: Construct EDF schedule (t=0 to 14)**

```
t=0: J1(d=4) vs J2(d=3) → J2 wins (d=3 < d=4) → Run J2
t=1: J2 done, J1 ready → Run J1
t=2: J1 ready → Run J1
t=3: J2 instance 2 arrives (d=6), J1 ready (d=4) → J1 wins → Run J1
t=4: J1 done (1st), J2 ready (d=6) → Run J2
t=5: J1 instance 2 arrives (d=9), J2 done → Run J1
t=6: J2 instance 3 arrives (d=9), J1(d=9) tie → Run J1 (lower ID)
t=7: J1 runs → Run J1
t=8: J1 instance 2 done, J2 ready (d=9) → Run J2
t=9: J2 done → idle (J1 instance 3 not arrived until t=10)
t=10: J1 instance 3 arrives (d=14) → Run J1
... continues
```

**Gantt chart (t=0 to 14):**
```
t:  0  1  2  3  4  5  6  7  8  9  10 11 12 13 14
    J2 J1 J1 J1 J2 J1 J1 J1 J2 __ J1 J1 J1 J2 __
```
Deadlines met: J1 instances finish by t=4, t=8, t=13 ≤ deadlines 4,9,14 ✅
J2 instances finish by t=1, t=5, t=9 ≤ deadlines 3,6,9 ✅

---

### Example 2 — RMS Scheduling

**Given jobs (implicit deadlines):**
- J1: π=5, η=3 (higher priority — shorter period)
- J2: π=3, η=1 (highest priority — shortest period)

Wait — J2 has shorter period → J2 gets **highest priority**.

**Priority assignment:** J2 > J1 (J2 period 3 < J1 period 5)

**Step 1: Utilization**
$$U = \frac{3}{5} + \frac{1}{3} = 0.933$$

B₂ = 0.828. Since U = 0.933 > 0.828, we are in the fallback zone — must verify by construction.

**Step 2: Construct RMS schedule**
```
t=0: J1 and J2 both arrive. J2 has higher priority → Run J2
t=1: J2 done. Only J1 ready → Run J1
t=2: J1 running → Run J1
t=3: J2 instance 2 arrives (higher priority) → PREEMPT J1, Run J2
t=4: J2 done. J1 resumes → Run J1 (completes remaining 1 slot)
t=4: J1 finishes. Deadline = 5 ✅ (finished at t=5)
```

Actually let's be precise:

```
t:  0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15
    J2 J1 J1 J2 J1 J1 J2 J1 J1 J2 __ J1 J1 J1 J2 __
```

J1 instance 1: runs at t=1,2,4 → finishes at t=5 ≤ deadline 5 ✅
J2 instance 1: runs at t=0 → finishes at t=1 ≤ deadline 3 ✅
Pattern repeats with period 15.

---

### Example 3 — Full 3-Task RMS

**Tasks:**
- τ1: π=4, η=1
- τ2: π=6, η=2
- τ3: π=12, η=4

**Priority order:** τ1 > τ2 > τ3 (shortest period first)

**Utilization:**
$$U = \frac{1}{4} + \frac{2}{6} + \frac{4}{12} = 0.25 + 0.333 + 0.333 = 0.917$$

B₃ = 0.780. U > B₃ → fallback zone. Verify by construction.

**Gantt Chart (t=0 to 12):**
```
t:   0  1  2  3  4  5  6  7  8  9  10 11 12
     τ1 τ2 τ2 τ3 τ1 τ3 τ2 τ2 τ3 τ1 τ3 τ3 (cycle)
```
Checking deadlines at t=12:
- τ1 instances at t=0,4,8 → finish at t=1,5,9 ≤ deadlines 4,8,12 ✅
- τ2 instances at t=0,6 → finish at t=3,8 ≤ deadlines 6,12 ✅
- τ3 at t=0 → finishes at t=12 ≤ deadline 12 ✅ (barely!)

---

## 5.5 EDF vs. RMS — Master Comparison Table

| Feature | EDF | RMS |
|---|---|---|
| **Type** | Dynamic priority | Fixed priority (static) |
| **Priority rule** | Smallest current absolute deadline | Shortest period |
| **Optimal?** | **Yes** — optimal among all policies | Optimal among fixed-priority only |
| **Schedulability condition** | U ≤ 1 (necessary & sufficient, implicit deadlines) | U ≤ Bₙ (sufficient only); verify if U > Bₙ |
| **Preemption** | Yes (deadline changes trigger) | Yes (higher priority preempts) |
| **Runtime overhead** | Higher (must compare deadlines each slot) | Minimal (fixed priority — compare once) |
| **Handles dynamic task arrival?** | Yes | Requires recalculation |
| **Implementation** | Priority queue sorted by deadline | Static priority table |
| **Best for** | Maximum utilization, dynamic systems | Safety-critical embedded, minimal overhead |

---

## 5.6 📝 Common Exam Questions — Scheduling

1. "For the given task set, compute utilization. Is it schedulable under RMS? Under EDF? Justify."
2. "Draw the Gantt chart for EDF/RMS for the given job model up to t=15ms."
3. "Three tasks with given parameters — apply Liu & Layland bound to determine schedulability."
4. "Explain why EDF is optimal. What does optimality mean here?"
5. "A task set has U=0.85 and 3 tasks. RMS bound is 0.78. What can you conclude? What must you do?"
6. "What is the difference between hard and soft real-time? Give examples."
7. "What is preemption? When does it occur in RMS scheduling?"

---

## 5.7 🔁 Unit 5 — Rapid Revision Notes

```
REAL-TIME SCHEDULING — MASTER CHEAT SHEET
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Task: (η, δ, π) where η ≤ δ ≤ π
Utilization: U = Σ η/π
If U > 1 → INFEASIBLE (no policy works)

EDF: 
  Rule: Always run job with smallest absolute deadline
  Condition: Schedulable IFF U ≤ 1 (implicit deadlines)
  Optimal: Yes (among all policies)
  Priority: Dynamic (changes over time)

RMS:
  Rule: Shorter period = higher priority (fixed forever)
  Bound: Bₙ = n(2^(1/n) - 1) → B₁=1, B₂=0.828, B∞=0.693
  Zone 1: U ≤ Bₙ → RMS guaranteed ✓
  Zone 2: Bₙ < U ≤ 1 → check by construction
  Zone 3: U > 1 → infeasible ✗
  Optimal: Only among fixed-priority policies

Gantt chart tips:
  1. Compute U first (exam partial marks)
  2. Mark arrival and deadline times
  3. Apply priority rule at each step
  4. Mark preemptions clearly
  5. Verify each deadline at the end
```

---

---

# ═══════════════════════════════════════════════
# UNIT 6 — CYBER SECURITY IN CPS
## Source: William Stallings, Computer Security Ch.15 + Academic Sources
# ═══════════════════════════════════════════════

---

## 6.1 CIA Triad — Foundation of CPS Security

Every security requirement in a CPS maps to one (or more) of these three properties:

```
        Confidentiality
           ▲
          / \
         /   \
        /     \
       ▼       ▼
   Integrity ←─→ Availability
```

| Property | Definition | CPS Example |
|---|---|---|
| **Confidentiality** | Only authorized parties can read the data | Sensor readings not visible to unauthorized users |
| **Integrity** | Data has not been modified by unauthorized parties | Control commands are not altered in transit |
| **Availability** | System remains operational and accessible | Industrial control system stays up during attack |

> 🔑 **For exams:** When asked to classify a threat, always identify which CIA property is being violated and give a specific mitigation.

---

## 6.2 Replay Attacks in CPS

### 6.2.1 What is a Replay Attack?

**Scenario:** An attacker records a legitimate message (e.g., a valid sensor reading or control command) and re-transmits it later — possibly after the physical conditions have changed.

```
Legitimate:  Sensor ──[reading: temp=25°C]──→ Controller → Safe action
Attack:      Attacker records this message
             Later: Attacker ──[replayed: temp=25°C]──→ Controller
             Reality: Actual temp = 90°C (fire!) → Controller takes wrong action
```

### 6.2.2 CIA Impact of Replay Attacks

| CIA Property | Impact |
|---|---|
| **Integrity** | ✅ Violated — controller receives stale/incorrect data |
| **Availability** | ✅ Potentially violated — wrong actions may crash the system |
| **Confidentiality** | ❌ Usually not violated (attacker doesn't need to understand data) |

### 6.2.3 Countermeasures

| Countermeasure | How It Works |
|---|---|
| **Timestamps** | Include timestamp in message; receiver rejects old messages |
| **Sequence numbers** | Each message gets unique number; duplicates rejected |
| **Nonces (challenge-response)** | Fresh random number included in each message; attacker can't predict |
| **HMAC with nonce** | Cryptographic authentication of each message including nonce |

---

## 6.3 Cryptographic Hash Functions

### 6.3.1 Definition

A **cryptographic hash function** H maps input of arbitrary length to a fixed-length output (hash/digest):

$$H: \{0,1\}^* \to \{0,1\}^n$$

Common outputs: MD5 (128-bit), SHA-1 (160-bit), SHA-256 (256-bit), SHA-3 (variable).

### 6.3.2 Three Essential Security Properties

| Property | Definition | Attack It Prevents |
|---|---|---|
| **Pre-image resistance** | Given h, computationally infeasible to find x such that H(x) = h | Recovering original data from hash |
| **Second pre-image resistance** | Given x, computationally infeasible to find x' ≠ x such that H(x') = H(x) | Substituting a message with the same hash |
| **Collision resistance** | Computationally infeasible to find ANY x ≠ x' such that H(x) = H(x') | Birthday attacks, forging signatures |

### 6.3.3 Digital Signatures with PKI Pipeline

```
SIGNING (Sender):
Message M ──→ [Hash function H] ──→ digest d
                                        ↓
                              d ──→ [Encrypt with sender's PRIVATE KEY]
                                        ↓
                              Signature S
Message M + Signature S ──→ Transmitted

VERIFICATION (Receiver):
Received M' + S ──→ [Hash function H] ──→ d'
                   S ──→ [Decrypt with sender's PUBLIC KEY] ──→ d
                   
Compare d' == d? → YES: Integrity and authenticity verified ✅
                  NO:  Message tampered or wrong sender ❌
```

**Why hash first then encrypt?**
- Hashing is fast; encrypting a hash (small) is much faster than encrypting a large message
- This is called an "Encrypt the hash" or "sign the digest" approach

---

## 6.4 Message Authentication Codes (MAC)

### 6.4.1 What is MAC?

MAC provides **integrity + authenticity** using a **pre-shared symmetric key** (both parties know the same secret key K).

$$\text{MAC} = H(K \| M)$$  
(HMAC: H(K ⊕ opad || H(K ⊕ ipad || M)))

### 6.4.2 MAC Block Diagram

```
GENERATING MAC:
Message M ──→ ┌──────────────────────────┐ ──→ MAC tag T
Shared Key K ─→ │  MAC Algorithm (e.g. HMAC) │
               └──────────────────────────┘

SENDING:    Transmit (M, T) to receiver

VERIFYING:
Received (M', T') 
M' + Shared Key K ──→ [Recompute MAC] ──→ T_verify
Compare T_verify == T'?
  → Match: M' is authentic and unmodified ✅
  → Mismatch: Message tampered or wrong key ❌
```

### 6.4.3 Worked Example — MAC on "SHIVA"

**Given:** Message = "SHIVA", Shared Key K = "SecretKey123"

1. Compute HMAC: H(K ⊕ ipad || "SHIVA") → inner hash
2. Compute outer: H(K ⊕ opad || inner hash) → final MAC tag T
3. Send ("SHIVA", T)
4. Receiver recomputes T_verify using same K
5. If T_verify = T → verified ✅

**MAC vs. Digital Signatures:**
| Feature | MAC | Digital Signature |
|---|---|---|
| Key type | Symmetric (shared) | Asymmetric (public/private) |
| Non-repudiation | ❌ No | ✅ Yes |
| Computation speed | Fast | Slower |
| Key management | Both parties need same key | Public key can be distributed freely |

---

## 6.5 MuTESLA Protocol

### 6.5.1 The Problem

In **Wireless Sensor Networks (WSNs)**:
- Nodes are resource-constrained (limited CPU, memory, battery)
- Standard asymmetric cryptography (RSA, ECC) requires heavy computation → Too expensive for tiny sensors
- But we still need authenticated broadcast from the base station

**Challenge:** How do we provide asymmetric-style authentication using only symmetric primitives?

### 6.5.2 MuTESLA Core Principles

**TESLA** = Timed Efficient Stream Loss-tolerant Authentication

MuTESLA adapts TESLA for multi-level WSNs.

**Key innovation:** **Delayed key disclosure** — the sender uses a key Kᵢ to compute a MAC on message Mᵢ, but only reveals Kᵢ in a *later* message. By the time the key is revealed, it's too late for an attacker to forge a message that would be accepted.

```
Time interval structure:
t=1: Send M₁ + MAC(K₁, M₁)    [K₁ not yet revealed]
t=2: Send M₂ + MAC(K₂, M₂) + REVEAL K₁
     Receiver now verifies M₁ using K₁
t=3: Send M₃ + MAC(K₃, M₃) + REVEAL K₂
     Receiver now verifies M₂ using K₂
...
```

**Key Chain:** Keys form a one-way hash chain:
$$K_n \to K_{n-1} \to \cdots \to K_2 \to K_1 \to K_0$$
(Each key is the hash of the previous: Kᵢ₋₁ = H(Kᵢ))

This means knowing K₁ lets you verify K₂, K₃, etc. via the hash chain.

### 6.5.3 Three Core Requirements of MuTESLA

| Requirement | Why Needed |
|---|---|
| **Loose time synchronization** | Receiver must know approximate time to reject messages with revealed keys (prevents attacker from using an already-disclosed key) |
| **Delayed key disclosure** | Provides asymmetric trust — attacker can't forge MAC before key is revealed, and after revelation it's too late |
| **One-way key chain** | Ensures keys can be verified forward from a trusted anchor without revealing future keys |

### 6.5.4 Why Not Standard Asymmetric Crypto?

| Property | Asymmetric (RSA) | MuTESLA (MAC-based) |
|---|---|---|
| Computation | **Heavy** (e.g., RSA 1024-bit: millions of ops) | **Light** (just hash operations) |
| Memory | **Large** key sizes | **Small** symmetric keys |
| Battery impact | **High** | **Low** |
| Suitable for sensor nodes? | ❌ No | ✅ Yes |

---

## 6.6 Buffer Overflow Attacks

### 6.6.1 The Mechanism

**Languages like C do NOT perform bounds checking on arrays.** A programmer writes:
```c
char buffer[10];
gets(buffer);  // Gets user input — NO SIZE LIMIT!
```

If the user inputs more than 10 characters, the excess bytes overflow beyond the buffer and overwrite adjacent memory.

### 6.6.2 Stack Frame Layout (Draw This in Exam!)

```
HIGH ADDRESS ───────────────────────────────────
│   PREVIOUS STACK FRAME                       │
├───────────────────────────────────────────────┤
│   Return Address  ← ATTACKER OVERWRITES THIS │ ← This is the critical target
├───────────────────────────────────────────────┤
│   Saved Frame Pointer (SFP)                  │ ← Also overwritten
├───────────────────────────────────────────────┤
│   Local Variable 2                           │
├───────────────────────────────────────────────┤
│   Local Variable 1                           │
├───────────────────────────────────────────────┤
│   buffer[0..9]     ← Input starts HERE       │ ← Buffer boundary
│   (10 bytes)                                  │
LOW ADDRESS  ────────────────────────────────────
             ↑ Stack grows downward
             ↑ New data is written upward toward higher addresses
```

**Attack flow:**
1. Attacker provides input longer than buffer size (e.g., 100 bytes into 10-byte buffer)
2. Bytes overflow upward, overwriting SFP, then Return Address
3. Attacker replaces Return Address with address of their **shellcode** (malicious code)
4. When the function returns, CPU jumps to attacker's code instead of the legitimate return point
5. Attacker gains control of the program's execution

### 6.6.3 Defense Mechanisms

| Defense | How It Works | Bypassed by |
|---|---|---|
| **Stack Canary** | Place a random "canary" value before return address; check it on function return | Overwriting canary too; brute force |
| **ASLR (Address Space Layout Randomization)** | Randomize memory addresses at runtime; attacker can't predict target addresses | Memory leaks, brute force |
| **Non-executable Stack (NX/DEP)** | Mark stack memory as non-executable; shellcode on stack can't run | Return-to-libc attacks |
| **Safe code boundaries** | Use safe string functions (strncpy, snprintf); perform bounds checking | Developer mistakes |
| **CFI (Control Flow Integrity)** | Restrict valid jump targets at runtime | — |

### 6.6.4 Exam Diagram — Stack Before and After Attack

```
BEFORE ATTACK:              AFTER ATTACK (overflow):
┌────────────────┐          ┌────────────────┐
│  Return Addr   │          │  0xDEADBEEF    │ ← Attacker's address
├────────────────┤          ├────────────────┤
│  Saved FP      │          │  AAAAAAAAAA    │ ← Overwritten
├────────────────┤          ├────────────────┤
│  buf[9]        │          │  SHELLCODEEEE  │ ← Shellcode injected
│  ...           │          │  SHELLCODEEEE  │
│  buf[0] = 'A'  │          │  AAAAAAAAAAAA  │
└────────────────┘          └────────────────┘
```

---

## 6.7 System Threat Classification (CIA Triad Mapping)

### Common CPS Threats and Their CIA Classification

| Threat | C | I | A | Description | Mitigation |
|---|---|---|---|---|---|
| **Man-in-the-Middle** | ✅ | ✅ | ❌ | Attacker intercepts and possibly modifies communications | Encryption + MAC |
| **Replay Attack** | ❌ | ✅ | ✅ | Replayed old message causes wrong actions | Timestamps, nonces |
| **DoS/DDoS** | ❌ | ❌ | ✅ | Flood network to prevent legitimate communication | Rate limiting, firewalls |
| **Buffer Overflow** | ✅ | ✅ | ✅ | Code injection via memory corruption | ASLR, canaries, NX |
| **Eavesdropping** | ✅ | ❌ | ❌ | Passive listening to sensor/control data | Encryption |
| **Sensor spoofing** | ❌ | ✅ | ❌ | Injecting false sensor data | Authentication, anomaly detection |
| **Ransomware** | ✅ | ✅ | ✅ | Encrypts plant data/control software | Backups, air-gap |

---

## 6.8 Merkle Trees (Hash Trees)

### What is a Merkle Tree?

A **binary tree** where:
- **Leaf nodes** contain hashes of individual data blocks
- **Internal nodes** contain hashes of their two children
- **Root** = single hash summarizing all data

```
                    Root = H(H12 || H34)
                   /                     \
          H12 = H(H1||H2)         H34 = H(H3||H4)
          /            \           /            \
   H1=H(D1)      H2=H(D2)   H3=H(D3)      H4=H(D4)
      |               |          |               |
     D1              D2         D3              D4
```

### Why Merkle Trees in CPS?

- **Efficient integrity verification:** To verify D1, you only need H2, H34, and Root (log₂N hashes instead of all N)
- **Used in:** Blockchain, TESLA key chains, firmware update verification, certificate transparency
- **Security:** Any change to any data block Dᵢ changes the root → tampering detected

---

## 6.9 🔁 Unit 6 — Rapid Revision Notes

```
CPS SECURITY — CHEAT SHEET
━━━━━━━━━━━━━━━━━━━━━━━━━━
CIA Triad:
  Confidentiality = no unauthorized reads
  Integrity = no unauthorized modifications
  Availability = system stays operational

Hash function properties:
  Pre-image resistance: can't find x from H(x)
  2nd pre-image: can't find x'≠x with H(x')=H(x)
  Collision: can't find ANY x≠x' with same hash

Digital Signature Pipeline:
  Sender: M → Hash → d → Encrypt(Private Key) → Signature S
  Receiver: M' → Hash → d' ; S → Decrypt(Public Key) → d
  Verify: d' == d?

MAC: H(K || M) using shared symmetric key K
  Fast, no non-repudiation, both parties need same key

MuTESLA:
  ✓ Delayed key disclosure (asymmetric auth from symmetric crypto)
  ✓ One-way key chain
  ✓ Loose time synchronization required
  ✗ No standard asymmetric crypto (too heavy for WSNs)

Buffer Overflow:
  Vulnerability: No bounds checking in C
  Target: Return address on stack
  Attack: Overflow buffer → overwrite return address → run shellcode
  Defenses: Stack canary, ASLR, NX stack, safe string functions

Replay Attack:
  Record + retransmit old message
  Violates: Integrity + Availability
  Countermeasures: Timestamps, nonces, sequence numbers
```

---

---

# 📊 CROSS-TOPIC COMPARISON TABLES

---

## Comparison 1: Synchronous vs. Asynchronous Models

| Feature | Synchronous Model (Lee & Seshia Ch.5) | Asynchronous Model (Alur Ch.4) |
|---|---|---|
| Execution | All tasks execute every round | One task executes per step |
| Time model | Discrete rounds / global clock | No global clock |
| Outputs | All tasks produce outputs every round | Single task produces one output/input |
| Precedence | Required (to order tasks in a round) | Not needed (each step = one task) |
| Communication | Shared variables, instant | Channels with possible buffering |
| Nondeterminism | Explicit (nondeterministic FSMs) | Inherent (which task fires next) |
| Complexity | State space = product of individual states | Same; but richer execution tree |
| Example | Digital circuits, embedded controllers | Networked sensors, distributed CPS |

---

## Comparison 2: EDF vs. RMS (Complete)

| Feature | EDF | RMS |
|---|---|---|
| Type | Dynamic priority | Fixed priority |
| Priority based on | Current absolute deadline | Fixed period |
| Priority changes | Yes (with each new instance) | Never |
| Optimal? | YES (overall optimal) | Optimal among fixed-priority |
| Schedulable condition | U ≤ 1 (necessary & sufficient) | U ≤ Bₙ (sufficient); check if Bₙ < U ≤ 1 |
| Bound | None (just U ≤ 1) | Bₙ = n(2^(1/n)-1); for large n → 0.693 |
| Preemptive | Yes | Yes |
| Overhead | Higher (sort by deadline) | Lower (static table lookup) |
| Handles non-equal deadlines | Yes | Assumes δ = π for optimality |

---

## Comparison 3: Hash Functions vs. MAC vs. Digital Signatures

| Feature | Hash Function | MAC | Digital Signature |
|---|---|---|---|
| Key required | No | Symmetric (shared) | Asymmetric (public/private) |
| Provides integrity | ✅ | ✅ | ✅ |
| Provides authentication | ❌ | ✅ (both parties) | ✅ (any verifier) |
| Non-repudiation | ❌ | ❌ | ✅ |
| Speed | Fast | Medium | Slow |
| Examples | SHA-256, MD5 | HMAC-SHA256 | RSA-SHA256, ECDSA |
| Used in CPS | Message integrity | Sensor data authentication | Firmware signing |

---

# 📌 FINAL EXAM CHECKLIST

Before the exam, make sure you can:

**Continuous Dynamics:**
- [ ] Write Newton's second law for translational and rotational motion
- [ ] Derive the helicopter ODE and its integral form
- [ ] Convert a 2nd-order ODE to transfer function via Laplace
- [ ] Convert an ODE to state-space (A, B, C, D matrices)
- [ ] Check stability via eigenvalues/poles
- [ ] Solve a first-order linear ODE analytically (show all steps)

**Feedback Control:**
- [ ] Draw the proportional control block diagram
- [ ] Derive θ̇y(t) = θ̇y(0)·e^(-Kt/Iyy) for the stabilization case
- [ ] Derive θ̇y(t) = a(1 - e^(-Kt/Iyy)) for the tracking case
- [ ] Find closed-loop transfer function for a given G(s) and K
- [ ] Explain why K > 0 stabilizes and K < 0 destabilizes

**Async Composition:**
- [ ] Define interleaving semantics precisely
- [ ] Explain how input events can be missed under interleaving
- [ ] Draw the composition FSM for asynchronous side-by-side
- [ ] Differentiate Semantics 1, 2, 3, 4
- [ ] Draw an FSM with proper `Input/Output` labels on every transition

**Real-Time Scheduling:**
- [ ] Compute utilization U = Σ η/π
- [ ] Apply Liu & Layland bound Bₙ = n(2^(1/n)-1)
- [ ] Construct EDF and RMS Gantt charts
- [ ] Identify preemption points in a schedule
- [ ] Classify a system as hard or soft real-time with justification

**CPS Security:**
- [ ] Name and define all three hash function security properties
- [ ] Draw the digital signature pipeline (both sign and verify)
- [ ] Draw the MAC verification block diagram
- [ ] Explain MuTESLA's three core principles
- [ ] Sketch the stack diagram for a buffer overflow
- [ ] Classify given CPS threats into CIA triad

---

# 📚 REFERENCES

| Source | Coverage |
|---|---|
| Rajeev Alur — *Principles of Cyber-Physical Systems* (MIT Press, 2015), Ch.4 pp.125–136 | Asynchronous Processes, ESMs |
| Rajeev Alur — *Principles of Cyber-Physical Systems*, Ch.8 pp.339–378 | RMS, EDF, Schedulability |
| Edward A. Lee & Sanjit A. Seshia — *Introduction to Embedded Systems*, 2nd Ed. (MIT Press, 2017), Ch.2 pp.19–37 | Continuous Dynamics, Feedback Control |
| Lee & Seshia Ch.5 pp.107–121 | Asynchronous Composition & Interleaving |
| Lee & Seshia Ch.12 pp.319–334 | Hard/Soft Real-Time, Scheduling Basics |
| William Stallings — *Computer Security: Principles and Practice*, Ch.15 | CPS Security, Buffer Overflow, Crypto |

**Trusted Online References:**
- https://ptolemy.berkeley.edu/books/leeseshia/ — Lee & Seshia textbook (free)
- https://www.cs.columbia.edu/~hgs/rts/ — Real-Time Systems resources
- https://cseweb.ucsd.edu/~mihir/papers/hmac.html — HMAC original paper
- https://www.sans.org/reading-room/whitepapers/vpns/overview-muTESLA-protocol-1515 — MuTESLA overview

---

*Document generated from: Rajeev Alur (Principles of CPS) + Lee & Seshia (Introduction to Embedded Systems, 2nd Ed.) + Stallings (Computer Security)*
*Coverage: All blueprint topics including Continuous Dynamics, Feedback Control, Stability, Async Composition, Interleaving, Hard/Soft Real-Time, RMS, EDF, ESMs, Task Graphs, Buffer Overflow, MAC, MuTESLA, Merkle Trees, Replay Attacks*
