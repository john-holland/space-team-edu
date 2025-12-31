# Drivetrains, Steering Behaviors, and Vehicle Physics: Mathematics

## Overview
Mathematical foundations for steering behaviors, vehicle physics, and drivetrain simulation, including force calculations, kinematics, and dynamics.

## Learning Objectives
- Master steering force calculations
- Understand vehicle kinematics and dynamics
- Learn drivetrain mathematics
- Calculate friction, slip, and traction forces

---

## Steering Behavior Mathematics

### Basic Steering Force
Steering force is calculated as desired velocity minus current velocity:

$$\mathbf{F}_{\text{steer}} = \mathbf{v}_{\text{desired}} - \mathbf{v}_{\text{current}}$$

Where:
- $\mathbf{v}_{\text{desired}}$ = desired velocity vector
- $\mathbf{v}_{\text{current}}$ = current velocity vector

### Seek Behavior
Desired velocity points toward target:

$$\mathbf{v}_{\text{desired}} = \frac{\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}}{|\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}|} \cdot v_{\max}$$

Steering force:
$$\mathbf{F}_{\text{seek}} = \mathbf{v}_{\text{desired}} - \mathbf{v}$$

### Flee Behavior
Desired velocity points away from target:

$$\mathbf{v}_{\text{desired}} = -\frac{\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}}{|\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}|} \cdot v_{\max}$$

Steering force:
$$\mathbf{F}_{\text{flee}} = \mathbf{v}_{\text{desired}} - \mathbf{v}$$

### Arrive Behavior
Desired velocity decreases as agent approaches target:

$$d = |\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}|$$

$$v_{\text{desired}} = \begin{cases}
v_{\max} & \text{if } d > r_{\text{slow}} \\
v_{\max} \cdot \frac{d}{r_{\text{slow}}} & \text{if } d \leq r_{\text{slow}}
\end{cases}$$

Where $r_{\text{slow}}$ is the slowing radius.

Direction:
$$\mathbf{d} = \frac{\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}}{d}$$

Desired velocity:
$$\mathbf{v}_{\text{desired}} = \mathbf{d} \cdot v_{\text{desired}}$$

### Wander Behavior
Wander uses a point on a circle ahead of the agent:

$$\mathbf{wander}_{\text{target}} = \mathbf{p}_{\text{agent}} + \mathbf{v}_{\text{normalized}} \cdot d_{\text{ahead}} + \mathbf{random}_{\text{circle}}$$

Where:
- $d_{\text{ahead}}$ = distance ahead
- $\mathbf{random}_{\text{circle}}$ = random point on circle

Steering force:
$$\mathbf{F}_{\text{wander}} = \text{Seek}(\mathbf{wander}_{\text{target}})$$

---

## Pursuit and Evade

### Pursuit Behavior
Predict target's future position:

$$t = \frac{|\mathbf{p}_{\text{target}} - \mathbf{p}_{\text{agent}}|}{v_{\max}}$$

Predicted position:
$$\mathbf{p}_{\text{predicted}} = \mathbf{p}_{\text{target}} + \mathbf{v}_{\text{target}} \cdot t$$

Steering force:
$$\mathbf{F}_{\text{pursuit}} = \text{Seek}(\mathbf{p}_{\text{predicted}})$$

### Evade Behavior
Flee from predicted position:

$$\mathbf{F}_{\text{evade}} = \text{Flee}(\mathbf{p}_{\text{predicted}})$$

---

## Obstacle Avoidance

### Raycast-Based Avoidance
Cast ray ahead of agent:

$$\mathbf{ray}_{\text{start}} = \mathbf{p}_{\text{agent}}$$
$$\mathbf{ray}_{\text{direction}} = \mathbf{v}_{\text{normalized}}$$
$$\mathbf{ray}_{\text{length}} = v \cdot t_{\text{lookahead}}$$

If ray hits obstacle at distance $d$:

$$\mathbf{F}_{\text{avoid}} = \frac{\mathbf{hit}_{\text{normal}}}{d^2} \cdot \text{strength}$$

### Force Field Avoidance
Obstacle exerts repulsive force:

$$d = |\mathbf{p}_{\text{obstacle}} - \mathbf{p}_{\text{agent}}|$$

$$\mathbf{F}_{\text{repel}} = \begin{cases}
-\frac{\mathbf{p}_{\text{obstacle}} - \mathbf{p}_{\text{agent}}}{d^2} \cdot k & \text{if } d < r_{\text{influence}} \\
\mathbf{0} & \text{if } d \geq r_{\text{influence}}
\end{cases}$$

Where $k$ is repulsion strength.

---

## Flocking Behaviors

### Separation
Steer away from neighbors:

$$\mathbf{F}_{\text{separation}} = \sum_{i} -\frac{\mathbf{p}_i - \mathbf{p}_{\text{agent}}}{|\mathbf{p}_i - \mathbf{p}_{\text{agent}}|^2}$$

Where sum is over neighbors within separation radius.

### Alignment
Steer toward average heading of neighbors:

$$\mathbf{v}_{\text{average}} = \frac{1}{n} \sum_{i} \mathbf{v}_i$$

$$\mathbf{F}_{\text{alignment}} = \mathbf{v}_{\text{average}} - \mathbf{v}_{\text{agent}}$$

### Cohesion
Steer toward average position of neighbors:

$$\mathbf{p}_{\text{average}} = \frac{1}{n} \sum_{i} \mathbf{p}_i$$

$$\mathbf{F}_{\text{cohesion}} = \text{Seek}(\mathbf{p}_{\text{average}})$$

### Combined Flocking
$$\mathbf{F}_{\text{flock}} = w_s \mathbf{F}_{\text{separation}} + w_a \mathbf{F}_{\text{alignment}} + w_c \mathbf{F}_{\text{cohesion}}$$

Where $w_s$, $w_a$, $w_c$ are weights.

---

## Vehicle Physics Mathematics

### Kinematics

#### Position Update
$$\mathbf{p}(t+\Delta t) = \mathbf{p}(t) + \mathbf{v}(t) \Delta t + \frac{1}{2}\mathbf{a}(t) \Delta t^2$$

#### Velocity Update
$$\mathbf{v}(t+\Delta t) = \mathbf{v}(t) + \mathbf{a}(t) \Delta t$$

#### Acceleration
$$\mathbf{a} = \frac{\mathbf{F}_{\text{total}}}{m}$$

Where $m$ is mass.

### Dynamics

#### Force Calculation
Total force on vehicle:
$$\mathbf{F}_{\text{total}} = \mathbf{F}_{\text{engine}} + \mathbf{F}_{\text{friction}} + \mathbf{F}_{\text{air}} + \mathbf{F}_{\text{gravity}}$$

#### Engine Force
$$\mathbf{F}_{\text{engine}} = \mathbf{d}_{\text{forward}} \cdot T_{\text{engine}} \cdot \text{gear ratio}$$

Where $T_{\text{engine}}$ is engine torque.

#### Friction Force
$$\mathbf{F}_{\text{friction}} = -\mu \cdot m \cdot g \cdot \mathbf{v}_{\text{normalized}}$$

Where:
- $\mu$ = friction coefficient
- $g$ = gravity acceleration
- $m$ = mass

#### Air Resistance
$$\mathbf{F}_{\text{air}} = -\frac{1}{2} \rho C_d A |\mathbf{v}|^2 \mathbf{v}_{\text{normalized}}$$

Where:
- $\rho$ = air density
- $C_d$ = drag coefficient
- $A$ = frontal area

---

## Wheel Physics

### Wheel Slip
Slip ratio:
$$\lambda = \frac{\omega r - v}{v}$$

Where:
- $\omega$ = angular velocity
- $r$ = wheel radius
- $v$ = vehicle speed

### Friction Force (Pacejka Model)
Longitudinal force:
$$F_x = D \sin(C \arctan(B \lambda - E(B \lambda - \arctan(B \lambda))))$$

Where $B$, $C$, $D$, $E$ are tire parameters.

### Lateral Force (Slip Angle)
Slip angle:
$$\alpha = \arctan\left(\frac{v_y}{v_x}\right)$$

Lateral force:
$$F_y = D \sin(C \arctan(B \alpha - E(B \alpha - \arctan(B \alpha))))$$

### Combined Slip
When both longitudinal and lateral slip occur:
$$F_{\text{total}} = \sqrt{F_x^2 + F_y^2}$$

But limited by friction circle:
$$F_{\max} = \mu \cdot F_z$$

Where $F_z$ is normal force (weight on wheel).

---

## Drivetrain Mathematics

### Front-Wheel Drive
Power to front wheels only:
$$T_{\text{front}} = T_{\text{engine}} \cdot \text{gear ratio}$$
$$T_{\text{rear}} = 0$$

### Rear-Wheel Drive
Power to rear wheels only:
$$T_{\text{front}} = 0$$
$$T_{\text{rear}} = T_{\text{engine}} \cdot \text{gear ratio}$$

### All-Wheel Drive
Power distributed:
$$T_{\text{front}} = T_{\text{engine}} \cdot \text{gear ratio} \cdot \text{split}_{\text{front}}$$
$$T_{\text{rear}} = T_{\text{engine}} \cdot \text{gear ratio} \cdot \text{split}_{\text{rear}}$$

Where $\text{split}_{\text{front}} + \text{split}_{\text{rear}} = 1$.

### Differential
Open differential:
$$T_{\text{left}} = T_{\text{right}} = \frac{T_{\text{input}}}{2}$$

But wheel speeds can differ:
$$\omega_{\text{left}} \neq \omega_{\text{right}}$$

Locked differential:
$$\omega_{\text{left}} = \omega_{\text{right}}$$

---

## Suspension Mathematics

### Spring Force
$$F_{\text{spring}} = -k \cdot x$$

Where:
- $k$ = spring constant
- $x$ = compression distance

### Damping Force
$$F_{\text{damping}} = -c \cdot \dot{x}$$

Where:
- $c$ = damping coefficient
- $\dot{x}$ = compression velocity

### Total Suspension Force
$$F_{\text{suspension}} = F_{\text{spring}} + F_{\text{damping}}$$

### Wheel Load
Normal force on wheel:
$$F_z = mg + F_{\text{suspension}}$$

Where weight distribution affects each wheel differently.

---

## Steering Mathematics

### Ackermann Steering
For turning radius $R$:

Inner wheel angle:
$$\delta_{\text{inner}} = \arctan\left(\frac{L}{R - \frac{w}{2}}\right)$$

Outer wheel angle:
$$\delta_{\text{outer}} = \arctan\left(\frac{L}{R + \frac{w}{2}}\right)$$

Where:
- $L$ = wheelbase
- $w$ = track width

### Turning Radius
From steering angle $\delta$:
$$R = \frac{L}{\tan(\delta)}$$

### Angular Velocity
Vehicle angular velocity:
$$\omega = \frac{v}{R} = \frac{v \tan(\delta)}{L}$$

---

## Vehicle Dynamics

### Center of Mass
$$\mathbf{p}_{\text{CM}} = \frac{1}{m} \sum_i m_i \mathbf{p}_i$$

### Moment of Inertia
For rotation around axis:
$$I = \sum_i m_i r_i^2$$

Where $r_i$ is distance from axis.

### Angular Acceleration
$$\alpha = \frac{\tau}{I}$$

Where $\tau$ is torque.

### Oversteer/Understeer
**Understeer**: Front wheels lose traction first
**Oversteer**: Rear wheels lose traction first

Determined by:
- Weight distribution
- Tire grip
- Suspension setup

---

## Exercises for Self-Learning

1. **Seek Implementation**: Implement seek behavior mathematically
2. **Arrive Behavior**: Calculate arrive with slowing radius
3. **Flocking**: Implement separation, alignment, cohesion
4. **Wheel Physics**: Calculate slip and friction forces
5. **Drivetrain**: Simulate different drivetrain types

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Craig Reynolds steering behaviors
- Vehicle dynamics textbooks
- Tire friction models (Pacejka)
- Physics simulation resources

