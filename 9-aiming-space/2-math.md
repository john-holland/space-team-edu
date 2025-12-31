# Aiming in Space: Mathematics

## Overview
Mathematical foundations for aiming and targeting in 3D space, including leading targets, ballistic trajectories, and 3D vector calculations.

## Learning Objectives
- Master 3D vector mathematics for aiming
- Calculate target leading
- Solve ballistic trajectory equations
- Handle relative motion calculations

---

## 3D Vector Mathematics

### Position Vectors
Position in 3D space:
$$\mathbf{p} = (p_x, p_y, p_z)$$

### Velocity Vectors
Velocity in 3D space:
$$\mathbf{v} = (v_x, v_y, v_z)$$

### Distance Calculation
Distance between two points:
$$d = |\mathbf{p}_2 - \mathbf{p}_1| = \sqrt{(p_{2,x} - p_{1,x})^2 + (p_{2,y} - p_{1,y})^2 + (p_{2,z} - p_{1,z})^2}$$

### Direction Vector
Direction from point A to point B:
$$\mathbf{d} = \frac{\mathbf{p}_B - \mathbf{p}_A}{|\mathbf{p}_B - \mathbf{p}_A|}$$

---

## Simple Target Leading

### Basic Leading Formula
If target is at position $\mathbf{p}_T$ with velocity $\mathbf{v}_T$, and projectile travels at speed $v_p$:

Time to intercept:
$$t = \frac{|\mathbf{p}_T - \mathbf{p}_S|}{v_p}$$

Aim point:
$$\mathbf{p}_{\text{aim}} = \mathbf{p}_T + \mathbf{v}_T \cdot t$$

Where $\mathbf{p}_S$ is shooter position.

### Relative Velocity
If shooter also moves with velocity $\mathbf{v}_S$:

Relative velocity:
$$\mathbf{v}_{\text{rel}} = \mathbf{v}_T - \mathbf{v}_S$$

Relative position:
$$\mathbf{p}_{\text{rel}} = \mathbf{p}_T - \mathbf{p}_S$$

### Leading with Relative Motion
Time to intercept (relative):
$$t = \frac{|\mathbf{p}_{\text{rel}}|}{v_p}$$

Aim point:
$$\mathbf{p}_{\text{aim}} = \mathbf{p}_T + \mathbf{v}_{\text{rel}} \cdot t$$

---

## Ballistic Trajectory

### Projectile Motion with Gravity
Projectile position at time $t$:
$$\mathbf{p}(t) = \mathbf{p}_0 + \mathbf{v}_0 t + \frac{1}{2}\mathbf{g} t^2$$

Where:
- $\mathbf{p}_0$ = initial position
- $\mathbf{v}_0$ = initial velocity
- $\mathbf{g} = (0, -g, 0)$ = gravity (assuming Y is up)

### Velocity at Time t
$$\mathbf{v}(t) = \mathbf{v}_0 + \mathbf{g} t$$

### Solving for Intercept
To hit target at position $\mathbf{p}_T$ at time $t$:

$$\mathbf{p}_T = \mathbf{p}_0 + \mathbf{v}_0 t + \frac{1}{2}\mathbf{g} t^2$$

Solving for $\mathbf{v}_0$:
$$\mathbf{v}_0 = \frac{\mathbf{p}_T - \mathbf{p}_0 - \frac{1}{2}\mathbf{g} t^2}{t}$$

### Time of Flight
For given initial velocity magnitude $v_0$ and angle $\theta$:

Time to reach maximum height:
$$t_{\text{max}} = \frac{v_0 \sin\theta}{g}$$

Total time of flight:
$$t_{\text{total}} = \frac{2v_0 \sin\theta}{g}$$

---

## Moving Target Intercept

### Intercept Problem
Shooter at $\mathbf{p}_S$ with projectile speed $v_p$
Target at $\mathbf{p}_T$ with velocity $\mathbf{v}_T$
Find: Aim direction and time to intercept

### Quadratic Equation Solution
Distance traveled by projectile:
$$d_p = v_p t$$

Distance traveled by target:
$$d_T = |\mathbf{v}_T| t$$

Relative position at time $t$:
$$\mathbf{p}_{\text{rel}}(t) = \mathbf{p}_T + \mathbf{v}_T t - \mathbf{p}_S$$

For intercept:
$$|\mathbf{p}_{\text{rel}}(t)| = v_p t$$

Squaring both sides:
$$|\mathbf{p}_T + \mathbf{v}_T t - \mathbf{p}_S|^2 = (v_p t)^2$$

Expanding:
$$|\mathbf{p}_{\text{rel},0}|^2 + 2(\mathbf{p}_{\text{rel},0} \cdot \mathbf{v}_T) t + |\mathbf{v}_T|^2 t^2 = v_p^2 t^2$$

Rearranging:
$$(|\mathbf{v}_T|^2 - v_p^2) t^2 + 2(\mathbf{p}_{\text{rel},0} \cdot \mathbf{v}_T) t + |\mathbf{p}_{\text{rel},0}|^2 = 0$$

This is a quadratic: $at^2 + bt + c = 0$

Where:
- $a = |\mathbf{v}_T|^2 - v_p^2$
- $b = 2(\mathbf{p}_{\text{rel},0} \cdot \mathbf{v}_T)$
- $c = |\mathbf{p}_{\text{rel},0}|^2$

### Solution
$$t = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

Take positive solution (earliest intercept).

Aim direction:
$$\mathbf{d}_{\text{aim}} = \frac{\mathbf{p}_T + \mathbf{v}_T t - \mathbf{p}_S}{|\mathbf{p}_T + \mathbf{v}_T t - \mathbf{p}_S|}$$

---

## Ballistic Intercept with Gravity

### Combined Problem
Projectile affected by gravity, target moving.

Projectile position:
$$\mathbf{p}_p(t) = \mathbf{p}_S + \mathbf{v}_0 t + \frac{1}{2}\mathbf{g} t^2$$

Target position:
$$\mathbf{p}_T(t) = \mathbf{p}_{T,0} + \mathbf{v}_T t$$

For intercept:
$$\mathbf{p}_p(t) = \mathbf{p}_T(t)$$

$$\mathbf{p}_S + \mathbf{v}_0 t + \frac{1}{2}\mathbf{g} t^2 = \mathbf{p}_{T,0} + \mathbf{v}_T t$$

Solving for $\mathbf{v}_0$:
$$\mathbf{v}_0 = \frac{\mathbf{p}_{T,0} + \mathbf{v}_T t - \mathbf{p}_S - \frac{1}{2}\mathbf{g} t^2}{t}$$

### Iterative Solution
For complex cases, use iterative approach:

1. Guess time $t$
2. Calculate target position at $t$
3. Calculate required velocity to hit that position
4. Check if trajectory is valid
5. Adjust $t$ and repeat

---

## Angular Calculations

### Angle to Target
Angle from shooter to target:
$$\theta = \arctan2(p_{T,z} - p_{S,z}, p_{T,x} - p_{S,x})$$

### Elevation Angle
Elevation angle (pitch):
$$\phi = \arctan\left(\frac{p_{T,y} - p_{S,y}}{\sqrt{(p_{T,x} - p_{S,x})^2 + (p_{T,z} - p_{S,z})^2}}\right)$$

### 3D Rotation
Rotation to aim at target:
$$\mathbf{R} = \text{RotationMatrix}(\theta, \phi)$$

---

## Exercises for Self-Learning

1. **Simple Leading**: Implement basic target leading
2. **Ballistic Trajectory**: Calculate ballistic path
3. **Intercept Calculation**: Solve intercept problem
4. **3D Angles**: Calculate aiming angles in 3D
5. **Iterative Solution**: Implement iterative intercept solver

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Ballistic trajectory mathematics
- 3D vector mathematics
- Intercept calculations
- Projectile motion physics
