# Space Physics: Theory and Implementation

## Overview
Detailed theory and implementation of space physics, including orbital mechanics, N-body problems, relativistic effects, and numerical methods.

## Learning Objectives
- Master orbital mechanics theory
- Understand N-body physics
- Learn numerical integration methods
- Handle relativistic effects

---

## Orbital Mechanics Fundamentals

### Two-Body Problem

#### Equations of Motion
For two bodies with masses $m_1$ and $m_2$:

**Position vectors**: $\mathbf{r}_1$, $\mathbf{r}_2$

**Relative position**: $\mathbf{r} = \mathbf{r}_2 - \mathbf{r}_1$

**Equation of motion**:
$$\ddot{\mathbf{r}} = -\frac{G(m_1 + m_2)}{r^3}\mathbf{r}$$

Where $r = |\mathbf{r}|$ and $G$ is gravitational constant.

#### Solution: Conic Sections
The solution is a conic section:
- **Ellipse**: $e < 1$ (bound orbit)
- **Parabola**: $e = 1$ (escape orbit)
- **Hyperbola**: $e > 1$ (unbound orbit)

Where $e$ is eccentricity.

### Kepler's Laws

#### First Law: Elliptical Orbits
Planets orbit in ellipses with the central body at one focus.

**Ellipse equation**:
$$r = \frac{a(1-e^2)}{1 + e\cos\nu}$$

Where:
- $a$ = semi-major axis
- $e$ = eccentricity
- $\nu$ = true anomaly

#### Second Law: Equal Areas
Line from central body to orbiting body sweeps equal areas in equal times.

**Area swept per time**:
$$\frac{dA}{dt} = \frac{1}{2}r^2\dot{\nu} = \text{constant}$$

#### Third Law: Period and Distance
$$T^2 = \frac{4\pi^2}{G(m_1 + m_2)}a^3$$

Where $T$ is orbital period.

---

## N-Body Problem

### Equations of Motion
For $N$ bodies:

**Force on body $i$**:
$$\mathbf{F}_i = \sum_{j \neq i} G\frac{m_i m_j}{|\mathbf{r}_{ij}|^2}\frac{\mathbf{r}_{ij}}{|\mathbf{r}_{ij}|}$$

Where $\mathbf{r}_{ij} = \mathbf{r}_j - \mathbf{r}_i$.

**Acceleration**:
$$\ddot{\mathbf{r}}_i = \sum_{j \neq i} G\frac{m_j}{|\mathbf{r}_{ij}|^2}\frac{\mathbf{r}_{ij}}{|\mathbf{r}_{ij}|}$$

### Energy Conservation
**Total energy**:
$$E = \sum_i \frac{1}{2}m_i|\dot{\mathbf{r}}_i|^2 - \sum_{i<j} G\frac{m_i m_j}{|\mathbf{r}_{ij}|}$$

**Conservation**:
$$\frac{dE}{dt} = 0$$

### Angular Momentum Conservation
**Total angular momentum**:
$$\mathbf{L} = \sum_i m_i \mathbf{r}_i \times \dot{\mathbf{r}}_i$$

**Conservation**:
$$\frac{d\mathbf{L}}{dt} = 0$$

---

## Numerical Integration Methods

### Euler Method
**Update equations**:
$$\mathbf{r}(t+\Delta t) = \mathbf{r}(t) + \dot{\mathbf{r}}(t)\Delta t$$
$$\dot{\mathbf{r}}(t+\Delta t) = \dot{\mathbf{r}}(t) + \ddot{\mathbf{r}}(t)\Delta t$$

**Error**: $O(\Delta t^2)$ per step, $O(\Delta t)$ global.

**Pros**: Simple, fast
**Cons**: Low accuracy, energy not conserved

### Verlet Integration
**Position update**:
$$\mathbf{r}(t+\Delta t) = 2\mathbf{r}(t) - \mathbf{r}(t-\Delta t) + \ddot{\mathbf{r}}(t)\Delta t^2$$

**Velocity** (for output):
$$\dot{\mathbf{r}}(t) = \frac{\mathbf{r}(t+\Delta t) - \mathbf{r}(t-\Delta t)}{2\Delta t}$$

**Error**: $O(\Delta t^4)$ per step, $O(\Delta t^2)$ global.

**Pros**: Better energy conservation, time-reversible
**Cons**: Requires previous position

### Runge-Kutta 4 (RK4)
**Stage calculations**:
$$\mathbf{k}_1 = \mathbf{f}(t, \mathbf{y})$$
$$\mathbf{k}_2 = \mathbf{f}(t + \frac{\Delta t}{2}, \mathbf{y} + \frac{\Delta t}{2}\mathbf{k}_1)$$
$$\mathbf{k}_3 = \mathbf{f}(t + \frac{\Delta t}{2}, \mathbf{y} + \frac{\Delta t}{2}\mathbf{k}_2)$$
$$\mathbf{k}_4 = \mathbf{f}(t + \Delta t, \mathbf{y} + \Delta t\mathbf{k}_3)$$

**Update**:
$$\mathbf{y}(t+\Delta t) = \mathbf{y}(t) + \frac{\Delta t}{6}(\mathbf{k}_1 + 2\mathbf{k}_2 + 2\mathbf{k}_3 + \mathbf{k}_4)$$

**Error**: $O(\Delta t^5)$ per step, $O(\Delta t^4)$ global.

**Pros**: High accuracy, good energy conservation
**Cons**: More expensive (4 function evaluations per step)

### Symplectic Methods
Methods that preserve symplectic structure (energy conservation):

**Leapfrog (Verlet)**:
$$\mathbf{r}(t+\Delta t) = \mathbf{r}(t) + \dot{\mathbf{r}}(t+\frac{\Delta t}{2})\Delta t$$
$$\dot{\mathbf{r}}(t+\frac{\Delta t}{2}) = \dot{\mathbf{r}}(t-\frac{\Delta t}{2}) + \ddot{\mathbf{r}}(t)\Delta t$$

**Pros**: Excellent long-term energy conservation
**Cons**: Slightly more complex

---

## Relativistic Effects

### Special Relativity
For high speeds (approaching speed of light):

**Time dilation**:
$$t' = \frac{t}{\sqrt{1 - \frac{v^2}{c^2}}}$$

**Length contraction**:
$$L' = L\sqrt{1 - \frac{v^2}{c^2}}$$

**Mass increase**:
$$m' = \frac{m}{\sqrt{1 - \frac{v^2}{c^2}}}$$

Where $c$ is speed of light.

### General Relativity
For strong gravitational fields:

**Schwarzschild metric** (non-rotating black hole):
$$ds^2 = -\left(1 - \frac{2GM}{c^2r}\right)dt^2 + \frac{dr^2}{1 - \frac{2GM}{c^2r}} + r^2d\Omega^2$$

**Event horizon**: $r = \frac{2GM}{c^2}$

### When to Use Relativity
- **Special Relativity**: $v > 0.1c$ (10% speed of light)
- **General Relativity**: Near black holes, strong fields
- **Games**: Usually negligible, can ignore

---

## Perturbation Theory

### Small Perturbations
When additional forces are small compared to main gravity:

**Perturbed acceleration**:
$$\ddot{\mathbf{r}} = \mathbf{a}_{\text{gravity}} + \mathbf{a}_{\text{perturbation}}$$

**Examples**:
- Atmospheric drag
- Solar radiation pressure
- Third-body perturbations
- Non-spherical gravity

### Averaging
Average out short-period effects:
$$\bar{\mathbf{a}} = \frac{1}{T}\int_0^T \mathbf{a}(t)dt$$

---

## Stability and Chaos

### Orbital Stability
**Stable orbits**: Small perturbations decay
**Unstable orbits**: Small perturbations grow

**Stability criteria**: Depends on orbital parameters

### Chaos in N-Body Systems
- **3-body problem**: Can be chaotic
- **Sensitive to initial conditions**: Small changes → large differences
- **Long-term prediction**: Difficult or impossible

### Lyapunov Exponent
Measure of chaos:
$$\lambda = \lim_{t \to \infty} \frac{1}{t}\ln\frac{|\delta\mathbf{r}(t)|}{|\delta\mathbf{r}(0)|}$$

- $\lambda > 0$: Chaotic
- $\lambda < 0$: Stable
- $\lambda = 0$: Marginally stable

---

## Practical Considerations

### Time Steps
**Choice of $\Delta t$**:
- Too large: Instability, energy drift
- Too small: Slow computation
- **Rule of thumb**: $\Delta t < \frac{T}{100}$ where $T$ is shortest orbital period

### Precision Issues
**Floating-point errors**:
- Accumulate over time
- Can cause energy drift
- **Solution**: Use higher precision or symplectic methods

### Performance
**N-body complexity**: $O(N^2)$ for direct calculation
**Optimizations**:
- Barnes-Hut algorithm: $O(N\log N)$
- Fast Multipole Method: $O(N)$
- Approximations for distant bodies

---

## Exercises for Self-Learning

1. **Two-Body Problem**: Implement and verify Kepler's laws
2. **N-Body Simulation**: Compare different integration methods
3. **Energy Conservation**: Monitor energy in simulation
4. **Perturbations**: Add atmospheric drag to orbit
5. **Chaos**: Study chaotic behavior in 3-body system

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)
- **Implementation**: See Unity code (3-unity-implementation.md)
- **Unity Physics**: See detailed implementation (5-space-physics-unity-implementation.md)

---

## References
- Orbital mechanics textbooks
- Numerical methods for physics
- N-body problem solutions
- Relativistic mechanics

