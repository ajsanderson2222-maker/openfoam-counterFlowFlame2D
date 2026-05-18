# counterFlowFlame2D — OpenFOAM 13 Laminar Methane Counter-Flow Diffusion Flame

Simulation of a laminar opposed-jet diffusion flame: methane (CH₄) and air streams
flow toward each other, meet at a stagnation plane, and react in a thin diffusion
flame. This is a canonical configuration for studying laminar flame structure,
extinction, and chemistry–diffusion coupling.

---

## Physical Setup

| Parameter | Value |
|---|---|
| Gap width | 20 mm |
| Fuel stream | Pure CH₄ at 0.1 m/s (→) |
| Oxidiser stream | Air (O₂/N₂) at 0.1 m/s (←) |
| Inlet temperature | 300 K (both streams) |
| Pressure | 1 atm |
| Strain rate (a = 2U/L) | 10 s⁻¹ |
| Flame type | Non-premixed diffusion flame |

The two streams collide at a stagnation plane near the mid-gap. Fuel diffuses toward
the oxidiser side and oxygen diffuses toward the fuel side; reaction occurs where the
mixture fraction crosses stoichiometry.

---

## Mesh

| Parameter | Value |
|---|---|
| Cells | 100 × 40 = 4,000 |
| Cell type | Hexahedral (2D, 1 cell deep) |
| Domain | 20 mm × 20 mm |
| Resolution | 0.2 mm × 0.5 mm per cell |
| Fuel inlet | Left face (x = 0) |
| Air inlet | Right face (x = 20 mm) |
| Outlets | Top and bottom faces |

**Mesh layout and flow configuration:**
![Mesh](mesh.png)

---

## Chemistry

| Setting | Value |
|---|---|
| Mechanism | Single-step irreversible Arrhenius |
| Reaction | CH₄ + 2 O₂ → CO₂ + 2 H₂O |
| Pre-exponential A | 5.2×10¹⁶ |
| Temperature exponent β | 0 |
| Activation temperature Ta | 14,906 K (Ea ≈ 124 kJ/mol) |
| Species | CH₄, O₂, CO₂, H₂O, N₂ |

The mechanism was developed by Bui-Pham (1992) specifically for this counter-flow
configuration. The Euler-implicit chemistry solver is used with a Rosenbrock stiff ODE
integrator to handle the stiff reaction source terms.

---

## Solver Setup

| Setting | Value |
|---|---|
| Solver | `multicomponentFluid` |
| Combustion model | PaSR (Partially Stirred Reactor) |
| Turbulence | Laminar |
| Thermodynamics | JANAF polynomials |
| Time step | Adaptive, max Co = 0.4 |
| End time | 0.5 s |
| Write interval | 0.05 s |

The flame reaches steady state within ~0.1 s. The remaining 0.4 s confirms
the solution is stationary.

---

## Results

### Flame Structure

The flame establishes rapidly and remains stationary — this is a steady laminar
diffusion flame. The stagnation point sits at approximately x = 7.5 mm (fuel-side
of mid-gap), with the flame zone displaced toward the air side at x ≈ 11 mm due to
the stoichiometry of methane combustion requiring more oxygen by mass.

**Temperature evolution (flame establishment):**
![Temperature animation](temperature_animation.gif)

**Temperature snapshots:**
![Temperature contours](T_contours.png)

**CH₄ mass fraction snapshots:**
![CH4 contours](CH4_contours.png)

**O₂ mass fraction snapshots:**
![O2 contours](O2_contours.png)

**CO₂ mass fraction snapshots:**
![CO2 contours](CO2_contours.png)

### Centreline Profiles

**Temperature, species, and velocity along the centreline at t = 0.5 s:**
![Centreline profiles](centreline_profiles.png)

Key features visible in the centreline plots:

- **Temperature**: Peak ~2,100 K at x ≈ 11 mm. The asymmetric profile reflects the
  displaced flame position — the fuel-side gradient is steeper because CH₄ diffuses
  into a region already preheated by conduction from the flame.
- **Species**: CH₄ and O₂ are mutually exclusive — they are fully consumed before
  reaching each other's inlet. CO₂ and H₂O peak sharply at the flame zone.
- **Velocity**: The two opposing streams decelerate to zero at the stagnation point
  (~x = 7.5 mm), with a velocity overshoot at the flame caused by thermal expansion
  of the hot gas.

### Convergence

**Solver residuals:**
![Convergence](convergence.png)

Residuals drop sharply as the flame establishes within the first 0.1 s and remain
at machine-precision levels for the remainder of the run, confirming a fully
converged steady flame.

---

## Reference

Bui-Pham, M. N. (1992). *Studies in structures of laminar hydrocarbon flames.*
PhD Thesis, University of California, San Diego.

---

## Running the Case

```bash
source /opt/openfoam13/etc/bashrc
cd openfoam-counterFlowFlame2D
blockMesh
foamRun
```
