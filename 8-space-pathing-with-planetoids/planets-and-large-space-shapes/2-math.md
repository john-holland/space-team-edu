# Planets and Large Space Shapes: Mathematics

## Overview
Mathematical foundations for planetary physics, large space structures, atmospheric systems, and pathfinding for massive objects.

## Learning Objectives
- Master planetary gravity mathematics
- Understand atmospheric physics calculations
- Learn geological feature mathematics
- Calculate pathfinding for large-scale objects

---

## Planetary Gravity

### Gravitational Field
Gravitational acceleration at distance $r$:
$$g(r) = \frac{GM}{r^2}$$

Where:
- $G = 6.674 \times 10^{-11} \text{ m}^3 \text{ kg}^{-1} \text{ s}^{-2}$
- $M$ = planet mass
- $r$ = distance from center

### Surface Gravity
At planet surface ($r = R$):
$$g_{\text{surface}} = \frac{GM}{R^2}$$

### Non-Spherical Gravity
For non-spherical planets (oblate spheroid):
$$g(\theta) = g_{\text{equator}} \left(1 + \frac{5}{2}J_2 \frac{3\cos^2\theta - 1}{2}\right)$$

Where:
- $\theta$ = latitude
- $J_2$ = oblateness coefficient

### Gravity Gradient
Rate of change of gravity:
$$\frac{dg}{dr} = -\frac{2GM}{r^3}$$

---

## Atmospheric Physics

### Atmospheric Pressure
Pressure at altitude $h$:
$$P(h) = P_0 e^{-\frac{h}{H}}$$

Where:
- $P_0$ = surface pressure
- $H$ = scale height = $\frac{kT}{mg}$

**Scale Height**:
$$H = \frac{RT}{Mg}$$

Where:
- $R$ = gas constant
- $T$ = temperature
- $M$ = molar mass
- $g$ = gravity

### Air Density
Density at altitude:
$$\rho(h) = \rho_0 e^{-\frac{h}{H}}$$

Where $\rho_0$ is surface density.

### Drag Force
Atmospheric drag:
$$F_{\text{drag}} = \frac{1}{2} \rho v^2 C_d A$$

Where:
- $\rho$ = air density
- $v$ = velocity
- $C_d$ = drag coefficient
- $A$ = cross-sectional area

---

## Crater Physics

### Impact Energy
Energy of impact:
$$E = \frac{1}{2} m v^2$$

Where:
- $m$ = impactor mass
- $v$ = impact velocity

### Crater Diameter
Empirical relationship:
$$D = k E^{1/3.4}$$

Where $k$ is material-dependent constant.

### Crater Depth
$$d = 0.1 D$$

Approximate relationship.

### Ejecta
Material thrown out:
$$M_{\text{ejecta}} \approx 100 \times m$$

Where $m$ is impactor mass.

---

## Tectonic Plates

### Plate Velocity
Typical plate velocities: 1-10 cm/year

**Conversion**:
$$v = 1 \text{ cm/year} = 3.17 \times 10^{-10} \text{ m/s}$$

### Stress Accumulation
Stress builds up at plate boundaries:
$$\sigma(t) = \sigma_0 + G \cdot v \cdot t$$

Where:
- $\sigma_0$ = initial stress
- $G$ = shear modulus
- $v$ = relative velocity
- $t$ = time

### Earthquake Magnitude
Richter scale:
$$M = \log_{10}(A) - \log_{10}(A_0)$$

Where $A$ is amplitude, $A_0$ is reference.

---

## Ocean Physics

### Wave Height
Wind-generated waves:
$$H = 0.0248 \frac{v^2}{g}$$

Where:
- $v$ = wind speed
- $g$ = gravity

### Tidal Forces
Tidal acceleration:
$$a_{\text{tidal}} = \frac{2GM_{\text{moon}} R}{d^3}$$

Where:
- $M_{\text{moon}}$ = moon mass
- $R$ = planet radius
- $d$ = distance to moon

### Ocean Depth Pressure
Pressure at depth $d$:
$$P(d) = P_{\text{surface}} + \rho g d$$

Where $\rho$ is water density.

---

## Large Space Structures

### Dyson Sphere
**Radius**: Typically 1 AU (Earth-Sun distance)

**Surface Area**:
$$A = 4\pi r^2$$

For $r = 1.5 \times 10^{11}$ m:
$$A \approx 2.8 \times 10^{23} \text{ m}^2$$

**Orbital Velocity**:
$$v = \sqrt{\frac{GM_{\text{star}}}{r}}$$

### Ring World
**Radius**: Large (millions of km)

**Circumference**:
$$C = 2\pi r$$

**Rotational Speed**:
For artificial gravity $g$:
$$v = \sqrt{gr}$$

**Angular Velocity**:
$$\omega = \frac{v}{r} = \sqrt{\frac{g}{r}}$$

---

## Quantum Structures

### Quantum Tunneling
Probability of tunneling:
$$P = e^{-2\kappa d}$$

Where:
- $\kappa = \sqrt{\frac{2m(V - E)}{\hbar^2}}$
- $d$ = barrier width
- $V$ = barrier height
- $E$ = particle energy

### Quantum Entanglement
For entangled systems, correlations exist even at distance.

**Bell Inequality**:
For local hidden variables:
$$|E(a,b) - E(a,b')| + |E(a',b) + E(a',b')| \leq 2$$

Quantum mechanics violates this.

---

## Plasma Physics

### Plasma Frequency
$$\omega_p = \sqrt{\frac{ne^2}{m_e \epsilon_0}}$$

Where:
- $n$ = electron density
- $e$ = electron charge
- $m_e$ = electron mass
- $\epsilon_0$ = permittivity

### Lightning
**Energy**:
$$E = \frac{1}{2}CV^2$$

Where:
- $C$ = capacitance
- $V$ = voltage

**Gigantic Lightning**:
For planet-scale:
$$E \approx 10^{12} \text{ J}$$

---

## Pathfinding Mathematics

### Gravity Well Navigation
Avoid deep gravity wells:
$$E_{\text{escape}} = \frac{GMm}{r}$$

**Delta-V to Escape**:
$$\Delta v = \sqrt{\frac{2GM}{r}}$$

### Optimal Trajectory
Minimize delta-V:
$$\min \sum_i |\Delta \mathbf{v}_i|$$

Subject to:
- Avoid obstacles
- Reach destination
- Time constraints

### Multi-Scale Coordinates
**Solar System**: Relative to star
**Planetary**: Relative to planet
**Surface**: Relative to surface point

**Transformation**:
$$\mathbf{p}_{\text{surface}} = \mathbf{R}^{-1}(\mathbf{p}_{\text{planetary}} - \mathbf{p}_{\text{origin}})$$

---

## Octree Balancing

### Fast Balancing
For large-scale pathfinding, use octree:

**Subdivision Criteria**:
- **Density**: Subdivide dense regions
- **Complexity**: Subdivide complex areas
- **Scale**: Different LOD at different scales

### G/g Approximation
Approximate gravity at different scales:
- **Global**: Full gravity calculation
- **Regional**: Approximate with point mass
- **Local**: Constant gravity approximation

### Event Playback
Record physics events (like Uncharted):
$$E(t) = \{\text{event}_1(t_1), \text{event}_2(t_2), \ldots\}$$

Playback for consistent results.

---

## Stress and Factors

### Structural Stress
For large structures:
$$\sigma = \frac{F}{A}$$

Where:
- $F$ = force
- $A$ = cross-sectional area

### Critical Stress
Structure fails when:
$$\sigma \geq \sigma_{\text{yield}}$$

### Safety Factor
$$SF = \frac{\sigma_{\text{yield}}}{\sigma_{\text{applied}}}$$

Typically $SF \geq 2$ for safety.

---

## Exercises for Self-Learning

1. **Gravity Calculation**: Calculate gravity at different altitudes
2. **Atmospheric Model**: Implement atmospheric pressure model
3. **Crater Generation**: Generate craters based on impact energy
4. **Tidal Forces**: Calculate tidal effects
5. **Large Structure**: Design physics for Dyson sphere or ring world

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Planetary physics
- Atmospheric science
- Structural mechanics
- Quantum mechanics
- Plasma physics
