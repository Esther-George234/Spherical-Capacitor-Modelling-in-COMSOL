# Finite-Element Analysis of a Spherical Capacitor in COMSOL Multiphysics

**Capacitance Computation and Electric Field / Potential Characterization for Homogeneous and Partially Filled Dielectric Configurations**

---

## 1. Abstract

This project models a spherical capacitor in COMSOL Multiphysics using the Electrostatics (`es`) interface. The objective is to compute the capacitance and to characterize the electric field and electric potential distributions under a 1 V applied potential. Two configurations are studied: (i) a capacitor completely filled with a dielectric of relative permittivity ε_r = 8, and (ii) a capacitor in which three quarters of the volume is filled with the dielectric and one quarter with air. Capacitance is obtained through two independent numerical routes, (a) the surface integral of the normal electric displacement, `es.Dn`, and (b) COMSOL's built-in Maxwell capacitance matrix, and both are compared against the analytical (textbook) solution. Results show that the `es.Dn` surface-integration method agrees more closely with the analytical value than the built-in Maxwell capacitance option, even on an extremely fine mesh.

## 2. Objectives

1. Build and solve a 3D (or axisymmetric) electrostatic model of a spherical capacitor.
2. Determine capacitance using two independent numerical methods.
3. Validate the numerical results against the analytical reference solution using absolute error.
4. Visualize and interpret the electric field and potential distributions.
5. Extend the model to a partially filled (dielectric + air) configuration.

## 3. Theoretical Background

### 3.1 Governing equations

The electrostatic problem is governed by Gauss's law and the potential formulation:

$$\nabla \cdot \mathbf{D} = \rho_v, \qquad \mathbf{D} = \varepsilon_0 \varepsilon_r \mathbf{E}, \qquad \mathbf{E} = -\nabla V$$

which, in a charge-free dielectric, reduces to Laplace's equation:

$$\nabla \cdot \left( \varepsilon_0 \varepsilon_r \nabla V \right) = 0$$

### 3.2 Analytical capacitance

For concentric spheres of inner radius *a* and outer radius *b* filled with a dielectric of relative permittivity ε_r:

$$C = \frac{4\pi \varepsilon_0 \varepsilon_r \, a b}{b - a}$$

The radial field and potential between the electrodes are:

$$E(r) = \frac{Q}{4\pi \varepsilon_0 \varepsilon_r r^2}, \qquad V(r) = \frac{Q}{4\pi \varepsilon_0 \varepsilon_r}\left(\frac{1}{r} - \frac{1}{b}\right)$$

### 3.3 Partially filled configuration

In Part 2 the volume is divided into four equal sectors by two orthogonal planes passing through the centre. Three sectors contain the dielectric (ε_r = 8) and one contains air (ε_r = 1). Because the field is purely radial, the sector interfaces are parallel to **E** and each sector sees the same potential difference. Each sector therefore contributes a capacitance proportional to its solid-angle fraction, and the total is the weighted combination:

$$C_{total} = \tfrac{3}{4} C_{diel} + \tfrac{1}{4} C_{air}$$

where C_diel and C_air are the capacitances of the fully filled configurations with ε_r = 8 and ε_r = 1 respectively.

### 3.4 Numerical capacitance methods

**Method 1: Charge from `es.Dn`.** The enclosed charge is obtained by integrating the normal displacement over the inner electrode surface:

$$Q = \oint_S \mathbf{D}\cdot\mathbf{n}\, dS$$

With an applied potential of V = 1 V, the capacitance is numerically equal to the charge:

$$C = \frac{Q}{V} = Q \quad (V = 1\ \text{V})$$

**Method 2: Built-in Maxwell capacitance.** The Terminal node of the Electrostatics interface is used to compute the Maxwell capacitance matrix, and the relevant entry is read from the global evaluation (`es.C11`).

## 4. Model Description

| Item | Description |
|---|---|
| Software | COMSOL Multiphysics (Electrostatics, `es`) |
| Study | Stationary |
| Geometry | Two concentric spheres, inner radius *a* = `3 cm`, outer radius *b* = `6 cm` |
| Boundary conditions | Inner sphere: Terminal / Electric Potential, V = 1 V. Outer sphere: Ground (V = 0). |
| Materials | Part 1: ε_r = 8 (entire domain). Part 2: ε_r = 8 (3 sectors), air ε_r = 1 (1 sector). |
| Mesh | Extremely fine, physics-controlled |

### 4.1 Part 1: Fully dielectric-filled capacitor

The full domain between the electrodes is assigned the dielectric material with ε_r = 8. Capacitance is calculated with both methods and compared against the analytical value. **The distributed model file implements this part.**

### 4.2 Part 2: Dielectric and air configuration (exercise)

Part 2 is intentionally left as an extension exercise. To build it from the provided Part 1 file:

1. Add **two Work Planes** to the geometry, both passing through the sphere centre and orthogonal to each other.
2. Apply a **Partition** operation (Partition Objects/Domains) using the work planes to split the spherical shell into four equal sectors.
3. Assign the **dielectric (ε_r = 8)** to three sectors and **air (ε_r = 1)** to the remaining sector.
4. Re-run the study and compute the capacitance with both `es.Dn` integration and the Maxwell capacitance, then compare with the analytical value from Section 3.3.

## 5. Results and Discussion

### 5.1 Capacitance comparison

| Configuration | Analytical | Reference book | COMSOL `es.Dn` | COMSOL Maxwell (`es.C11`) | Abs. error (`es.Dn`) | Abs. error (Maxwell) |
|---|---|---|---|---|---|---|
| Part 1: ε_r = 8 | `53.38` pF | `53.3` pF | `53.382` pF | `53.407` pF | `0.002` pF | `0.027` pF |
| Part 2: 3/4 dielectric + 1/4 air | `41.7` pF | `41.7` pF | `41.704` pF | `41.724` pF | `0.004` pF | `0.024` pF |

Absolute error is defined as:

$$\varepsilon_{abs} = \left| C_{numerical} - C_{analytical} \right|$$

### 5.2 Observations

- The `es.Dn` surface-integral method produced a smaller absolute error than the built-in Maxwell capacitance option, and this held even with an extremely fine mesh.
- A plausible explanation is that the Maxwell capacitance is derived from the energy / terminal-charge post-processing, which is more sensitive to discretization of the field near the electrodes, whereas the direct flux integral of **D** over the electrode surface converges more directly to the enclosed charge. This should be confirmed with a mesh-convergence study.
- The electric field magnitude is largest at the inner electrode surface and decays as 1/r², consistent with the analytical solution. The potential decreases monotonically from 1 V at the inner sphere to 0 V at the outer sphere.
- In the dielectric region the displacement field is continuous across the radial direction, while the E-field is reduced by the factor 1/ε_r relative to air at the same radius.


## 6. Repository Contents

```
.
├── README.md
├── spherical_capacitor_part1.mph    # COMSOL model, Part 1 (downloadable, see below)
├── report/                          # Detailed report with absolute error analysis
└── figures/                         # Plots exported from COMSOL
```

**Model download:** `https://mega.nz/file/jgoCADha#R8YZsRV1rC1Q29OKn0MB6_HMpmiZu0gOrKPUuWKW9Wg`

## 7. How to Reproduce

1. Download the `.mph` file from the MEGA link above.
2. Open it in COMSOL Multiphysics (Electrostatics module required).
3. Run the Stationary study.
4. For `es.Dn`: go to Results, Derived Values, Surface Integration, select the inner sphere boundary, and evaluate `es.Dn`.
5. For the Maxwell capacitance: go to Results, Derived Values, Global Evaluation, and evaluate `es.C11`.
6. Compare both values with the analytical expression of Section 3.2.

## 8. Conclusions

- Capacitance was computed by two independent COMSOL methods and validated against the analytical solution.
- The `es.Dn` flux-integral approach is more accurate than the built-in Maxwell capacitance option for this geometry.
- The project served as an introduction to COMSOL workflow (geometry, materials, physics, meshing, post-processing) and to the electrostatic analysis of a simple, analytically tractable system.

## 9. Possible Extensions

- Mesh-convergence study quantifying error versus element count for both methods.
- Layered (radial) dielectrics to obtain a series-capacitor configuration.
- Use of axisymmetric 2D modelling to reduce computational cost.

## 10. Author

`Esther George Fayez`, `Electromagnetic Fields`, `2025/2026`

## 11. References

1. `William Hart Hayt_ John A. Buck - Engineering electromagnetics-McGraw-Hill (2012), CH6 Capacitors, Problem 6.11`
2. COMSOL Multiphysics, *AC/DC Module User's Guide*, COMSOL AB.
