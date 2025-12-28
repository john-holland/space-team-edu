# Space Pathing with Planetoids: Mathematics

## Overview
Mathematical foundations for pathfinding in 3D space with gravitational bodies, including trajectory planning, obstacle avoidance, and multi-scale pathfinding.

## Learning Objectives
- Master 3D pathfinding mathematics
- Understand trajectory optimization
- Learn gravitational path planning
- Calculate intercept trajectories

---

## 3D Pathfinding

### 3D A* Pathfinding
Extend A* to 3D space:

**Node**: $(x, y, z)$ position in 3D

**Neighbors**: 6-connected (face) or 26-connected (includes edges/corners)

**Heuristic**: 3D Euclidean distance
$$h(n) = \sqrt{(x_g - x_n)^2 + (y_g - y_n)^2 + (z_g - z_n)^2}$$

**Cost**: 3D distance
$$c(n_1, n_2) = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2 + (z_2 - z_1)^2}$$

---

## Trajectory Planning

### Hohmann Transfer
Elliptical transfer orbit between two circular orbits.

**Transfer time**:
$$t_{\text{transfer}} = \pi \sqrt{\frac{a^3}{GM}}$$

Where $a$ is semi-major axis of transfer orbit.

**Delta-V**:
$$\Delta v_1 = \sqrt{\frac{GM}{r_1}}\left(\sqrt{\frac{2r_2}{r_1 + r_2}} - 1\right)$$
$$\Delta v_2 = \sqrt{\frac{GM}{r_2}}\left(1 - \sqrt{\frac{2r_1}{r_1 + r_2}}\right)$$

### Bi-Elliptic Transfer
Three-impulse transfer (sometimes more efficient):
- First burn: Raise apoapsis
- Second burn: Raise periapsis
- Third burn: Circularize

More complex but can save fuel for large transfers.

---

## Gravitational Path Planning

### Patched Conic Approximation
Approximate trajectory using conic sections:
- **Hyperbolic**: Approach/departure
- **Elliptical**: Transfer orbit
- **Parabolic**: Escape trajectory

### Sphere of Influence
Radius where planet's gravity dominates:
$$r_{\text{SOI}} = a\left(\frac{m}{M}\right)^{2/5}$$

Where:
- $a$ = semi-major axis of planet's orbit
- $m$ = planet mass
- $M$ = central body mass

### Patched Conics
1. **Heliocentric**: Trajectory around sun
2. **Planetocentric**: Trajectory around planet (within SOI)
3. **Patch**: Connect at SOI boundary

---

## Obstacle Avoidance

### Sphere Obstacle
Planet/moon as sphere obstacle.

**Distance to obstacle**:
$$d = |\mathbf{p} - \mathbf{p}_{\text{obstacle}}| - r_{\text{obstacle}}$$

**Avoidance constraint**:
$$d \geq d_{\text{safe}}$$

Where $d_{\text{safe}}$ is safe distance.

### Multiple Obstacles
Avoid multiple spheres:
$$d_i \geq d_{\text{safe},i} \quad \forall i$$

### Collision Check
Check if trajectory intersects obstacle:
$$\min_{t \in [0, T]} |\mathbf{r}(t) - \mathbf{p}_{\text{obstacle}}| < r_{\text{obstacle}} + r_{\text{safe}}$$

---

## RRT (Rapidly-Exploring Random Tree)

### 3D RRT
Sample 3D space, build tree toward goal.

**Sampling**:
$$\mathbf{x}_{\text{rand}} = \text{RandomSample}()$$

**Nearest node**:
$$n_{\text{near}} = \arg\min_{n \in \text{tree}} |\mathbf{x}_{\text{rand}} - n|$$

**New node**:
$$\mathbf{x}_{\text{new}} = n_{\text{near}} + \frac{\mathbf{x}_{\text{rand}} - n_{\text{near}}}{|\mathbf{x}_{\text{rand}} - n_{\text{near}}|} \cdot \Delta d$$

**Collision check**: Verify $\mathbf{x}_{\text{new}}$ doesn't intersect obstacles.

### RRT with Gravity
Account for orbital mechanics in edge cost:
$$c(n_1, n_2) = \Delta v_{\text{required}}$$

Calculate $\Delta v$ using orbital mechanics.

---

## Trajectory Optimization

### Objective Function
Minimize fuel (delta-V):
$$\min \sum_{i} |\Delta \mathbf{v}_i|$$

Subject to constraints:
- Avoid obstacles
- Reach target
- Time constraints

### Constraints
**Obstacle avoidance**:
$$|\mathbf{r}(t) - \mathbf{p}_{\text{obstacle}}| \geq r_{\text{obstacle}} + r_{\text{safe}}$$

**Orbital mechanics**:
$$\mathbf{r}(t), \mathbf{v}(t) \text{ satisfy orbital equations}$$

**Time**:
$$t_{\text{arrival}} \leq t_{\text{max}}$$

### Optimization Methods
- **Gradient descent**: Local optimization
- **Genetic algorithms**: Global optimization
- **Particle swarm**: Population-based
- **Sequential quadratic programming**: Constrained optimization

---

## Intercept Calculations

### Intercept Problem
Shooter at $\mathbf{p}_s$ with velocity $\mathbf{v}_s$
Target at $\mathbf{p}_t$ with velocity $\mathbf{v}_t$
Projectile speed $v_p$

Find: Intercept point and time

### Solution
Relative position:
$$\mathbf{p}_{\text{rel}} = \mathbf{p}_t - \mathbf{p}_s$$

Relative velocity:
$$\mathbf{v}_{\text{rel}} = \mathbf{v}_t - \mathbf{v}_s$$

Time to intercept (quadratic):
$$(|\mathbf{v}_{\text{rel}}|^2 - v_p^2) t^2 + 2(\mathbf{p}_{\text{rel}} \cdot \mathbf{v}_{\text{rel}}) t + |\mathbf{p}_{\text{rel}}|^2 = 0$$

Solve for $t$:
$$t = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

Where:
- $a = |\mathbf{v}_{\text{rel}}|^2 - v_p^2$
- $b = 2(\mathbf{p}_{\text{rel}} \cdot \mathbf{v}_{\text{rel}})$
- $c = |\mathbf{p}_{\text{rel}}|^2$

Intercept point:
$$\mathbf{p}_{\text{intercept}} = \mathbf{p}_t + \mathbf{v}_t \cdot t$$

---

## Multi-Scale Pathfinding

### Hierarchical Levels
- **Level 0**: Solar system (planets)
- **Level 1**: Planet system (moons)
- **Level 2**: Surface (terrain)

### Scale Transitions
At each level, use appropriate:
- **Coordinate system**: Relative to local body
- **Distance units**: AU, km, m
- **Time scale**: Years, days, seconds

### Path Composition
1. **High-level**: Navigate between planets
2. **Mid-level**: Navigate around moons
3. **Low-level**: Surface navigation

---

## Fuel Optimization

### Delta-V Budget
Total delta-V available:
$$\Delta v_{\text{total}} = \sum_i \Delta v_i$$

### Optimal Transfer
Minimize delta-V:
$$\min \sum_i |\Delta \mathbf{v}_i|$$

### Gravity Assists
Use planetary gravity to change velocity:
$$\Delta \mathbf{v}_{\text{out}} = \Delta \mathbf{v}_{\text{in}} + 2\mathbf{v}_{\text{planet}}$$

Can gain or lose velocity depending on approach.

---

## Time-Dependent Obstacles

### Moving Obstacles
Planets/moons move in orbits:
$$\mathbf{p}_{\text{obstacle}}(t) = \mathbf{p}_{\text{orbit}}(t)$$

### Predictive Avoidance
Check future positions:
$$\mathbf{p}_{\text{obstacle}}(t + \Delta t) = \mathbf{p}_{\text{orbit}}(t + \Delta t)$$

### Time Windows
Find time windows when path is clear:
$$t \in [t_1, t_2] \text{ such that path is obstacle-free}$$

---

## Exercises for Self-Learning

1. **3D A***: Implement 3D A* pathfinding
2. **Hohmann Transfer**: Calculate transfer orbit
3. **Obstacle Avoidance**: Implement sphere obstacle avoidance
4. **RRT**: Implement RRT for 3D space
5. **Intercept**: Calculate intercept trajectories

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- 3D pathfinding algorithms
- Orbital mechanics
- Trajectory optimization
- RRT algorithms
