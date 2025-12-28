# Space: Mathematics

## Overview
Mathematical foundations for space simulation, including orbital mechanics, coordinate systems, gravitational physics, and numerical methods.

## Learning Objectives
- Master orbital mechanics mathematics
- Understand coordinate system transformations
- Learn gravitational force calculations
- Handle numerical precision issues

---

## Orbital Mechanics

### Kepler's Laws

#### First Law: Elliptical Orbits
Planets orbit in ellipses with the central body at one focus.

Ellipse equation:
$$\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$$

Where:
- $a$ = semi-major axis
- $b$ = semi-minor axis
- $e = \sqrt{1 - \frac{b^2}{a^2}}$ = eccentricity

#### Second Law: Equal Areas
Line from central body to orbiting body sweeps equal areas in equal times.

Area swept per time:
$$\frac{dA}{dt} = \frac{1}{2} r^2 \dot{\theta} = \text{constant}$$

Where $r$ is distance, $\theta$ is angle.

#### Third Law: Period and Distance
$$T^2 \propto a^3$$

For circular orbits:
$$T^2 = \frac{4\pi^2}{GM} a^3$$

Where:
- $T$ = orbital period
- $G$ = gravitational constant
- $M$ = mass of central body

---

## Gravitational Force

### Newton's Law of Universal Gravitation
$$F = G \frac{m_1 m_2}{r^2}$$

Where:
- $G = 6.674 \times 10^{-11} \text{ m}^3 \text{ kg}^{-1} \text{ s}^{-2}$
- $m_1, m_2$ = masses
- $r$ = distance between centers

### Force Vector
$$\mathbf{F} = -G \frac{m_1 m_2}{|\mathbf{r}|^2} \frac{\mathbf{r}}{|\mathbf{r}|}$$

Where $\mathbf{r} = \mathbf{r}_2 - \mathbf{r}_1$ is position vector.

### Acceleration
From $F = ma$:
$$\mathbf{a} = -G \frac{M}{|\mathbf{r}|^2} \frac{\mathbf{r}}{|\mathbf{r}|}$$

Where $M$ is mass of central body.

---

## Orbital Elements

### Classical Orbital Elements
Six parameters define an orbit:

1. **Semi-major axis** $a$: Size of orbit
2. **Eccentricity** $e$: Shape of orbit (0 = circle, <1 = ellipse)
3. **Inclination** $i$: Tilt of orbit plane
4. **Longitude of ascending node** $\Omega$: Orientation of orbit
5. **Argument of periapsis** $\omega$: Orientation of ellipse
6. **True anomaly** $\nu$: Position along orbit

### Conversion to Cartesian
From orbital elements to position/velocity:

**Perifocal coordinates** (orbit plane):
$$r = \frac{a(1-e^2)}{1 + e\cos\nu}$$
$$x_p = r\cos\nu$$
$$y_p = r\sin\nu$$
$$v_x = \sqrt{\frac{GM}{a(1-e^2)}}(-\sin\nu)$$
$$v_y = \sqrt{\frac{GM}{a(1-e^2)}}(e + \cos\nu)$$

**Rotate to inertial frame** using rotation matrices for $i$, $\Omega$, $\omega$.

---

## Coordinate Systems

### World Space
Global coordinate system:
$$\mathbf{p}_{\text{world}} = (x, y, z)$$

### Orbital Frame
Relative to central body:
$$\mathbf{p}_{\text{orbital}} = \mathbf{p}_{\text{world}} - \mathbf{p}_{\text{central}}$$

### Local Frame
Relative to spacecraft:
$$\mathbf{p}_{\text{local}} = \mathbf{R}^{-1}(\mathbf{p}_{\text{world}} - \mathbf{p}_{\text{spacecraft}})$$

Where $\mathbf{R}$ is rotation matrix.

### Celestial Frame
Fixed to stars (inertial frame):
$$\mathbf{p}_{\text{celestial}} = \mathbf{R}_{\text{precession}} \mathbf{R}_{\text{nutation}} \mathbf{p}_{\text{world}}$$

---

## Orbital Velocity

### Circular Orbit Velocity
$$v = \sqrt{\frac{GM}{r}}$$

Where $r$ is orbital radius.

### Elliptical Orbit Velocity
At distance $r$ from central body:
$$v = \sqrt{GM\left(\frac{2}{r} - \frac{1}{a}\right)}$$

This is the **vis-viva equation**.

### Escape Velocity
Speed needed to escape gravitational influence:
$$v_{\text{escape}} = \sqrt{\frac{2GM}{r}}$$

---

## N-Body Problem

### Force on Body $i$
$$\mathbf{F}_i = \sum_{j \neq i} G \frac{m_i m_j}{|\mathbf{r}_{ij}|^2} \frac{\mathbf{r}_{ij}}{|\mathbf{r}_{ij}|}$$

Where $\mathbf{r}_{ij} = \mathbf{r}_j - \mathbf{r}_i$.

### Acceleration
$$\mathbf{a}_i = \sum_{j \neq i} G \frac{m_j}{|\mathbf{r}_{ij}|^2} \frac{\mathbf{r}_{ij}}{|\mathbf{r}_{ij}|}$$

### Equations of Motion
$$\frac{d\mathbf{r}_i}{dt} = \mathbf{v}_i$$
$$\frac{d\mathbf{v}_i}{dt} = \mathbf{a}_i$$

---

## Numerical Integration

### Euler Method
$$\mathbf{r}(t+\Delta t) = \mathbf{r}(t) + \mathbf{v}(t) \Delta t$$
$$\mathbf{v}(t+\Delta t) = \mathbf{v}(t) + \mathbf{a}(t) \Delta t$$

**Error**: $O(\Delta t^2)$ per step, $O(\Delta t)$ global.

### Verlet Integration
Position update:
$$\mathbf{r}(t+\Delta t) = 2\mathbf{r}(t) - \mathbf{r}(t-\Delta t) + \mathbf{a}(t) \Delta t^2$$

**Error**: $O(\Delta t^4)$ per step, $O(\Delta t^2)$ global.

### Runge-Kutta 4 (RK4)
$$\mathbf{k}_1 = \mathbf{f}(t, \mathbf{y})$$
$$\mathbf{k}_2 = \mathbf{f}(t + \frac{\Delta t}{2}, \mathbf{y} + \frac{\Delta t}{2}\mathbf{k}_1)$$
$$\mathbf{k}_3 = \mathbf{f}(t + \frac{\Delta t}{2}, \mathbf{y} + \frac{\Delta t}{2}\mathbf{k}_2)$$
$$\mathbf{k}_4 = \mathbf{f}(t + \Delta t, \mathbf{y} + \Delta t \mathbf{k}_3)$$
$$\mathbf{y}(t+\Delta t) = \mathbf{y}(t) + \frac{\Delta t}{6}(\mathbf{k}_1 + 2\mathbf{k}_2 + 2\mathbf{k}_3 + \mathbf{k}_4)$$

**Error**: $O(\Delta t^5)$ per step, $O(\Delta t^4)$ global.

---

## Precision Issues

### Floating-Point Precision
For large distances, floating-point precision becomes issue.

**Relative Error**:
$$\epsilon_{\text{relative}} = \frac{\epsilon_{\text{machine}}}{|\mathbf{r}|}$$

For 32-bit float: $\epsilon_{\text{machine}} \approx 10^{-7}$

### Solution: Relative Coordinates
Use relative coordinates to central body:
$$\mathbf{r}_{\text{relative}} = \mathbf{r}_{\text{absolute}} - \mathbf{r}_{\text{central}}$$

### Hierarchical Coordinates
Use local coordinate systems:
- **Solar system**: Relative to sun
- **Planet system**: Relative to planet
- **Surface**: Relative to surface point

---

## Time Scales

### Orbital Period
$$T = 2\pi \sqrt{\frac{a^3}{GM}}$$

### Time Dilation (Relativity)
For high speeds:
$$t' = \frac{t}{\sqrt{1 - \frac{v^2}{c^2}}}$$

Where $c$ is speed of light.

Usually negligible for game speeds.

---

## Energy

### Total Energy
$$E = \frac{1}{2}mv^2 - \frac{GMm}{r}$$

### Kinetic Energy
$$E_k = \frac{1}{2}mv^2$$

### Potential Energy
$$E_p = -\frac{GMm}{r}$$

### Conservation
Total energy is conserved (no external forces):
$$\frac{dE}{dt} = 0$$

---

## Hohmann Transfer

### Transfer Orbit
Elliptical orbit connecting two circular orbits.

**Semi-major axis**:
$$a_{\text{transfer}} = \frac{r_1 + r_2}{2}$$

**Velocity at periapsis**:
$$v_1 = \sqrt{GM\left(\frac{2}{r_1} - \frac{1}{a_{\text{transfer}}}\right)}$$

**Velocity at apoapsis**:
$$v_2 = \sqrt{GM\left(\frac{2}{r_2} - \frac{1}{a_{\text{transfer}}}\right)}$$

**Delta-V required**:
$$\Delta v_1 = v_1 - v_{\text{circular},1}$$
$$\Delta v_2 = v_{\text{circular},2} - v_2$$
$$\Delta v_{\text{total}} = \Delta v_1 + \Delta v_2$$

---

## Exercises for Self-Learning

1. **Orbital Elements**: Convert between orbital elements and Cartesian
2. **Gravitational Force**: Calculate forces in N-body system
3. **Numerical Integration**: Implement RK4 for orbital motion
4. **Hohmann Transfer**: Calculate transfer orbit between two planets
5. **Precision**: Test floating-point precision with large distances

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)
- **Physics**: Learn space physics details (4-space-physics.md)

---

## References
- Orbital mechanics textbooks
- Numerical methods for physics
- Coordinate system transformations
- N-body problem solutions
