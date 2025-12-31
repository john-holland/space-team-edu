# Drivetrains, Steering Behaviors, and Vehicle Physics: Unity Implementation

## Overview
Practical implementation of steering behaviors and vehicle physics in Unity, including C# code examples, force-based movement, and drivetrain simulation.

## Learning Objectives
- Implement basic steering behaviors (seek, flee, arrive)
- Create vehicle physics with wheels and suspension
- Simulate different drivetrain types
- Combine multiple steering behaviors
- Optimize for performance

---

## Basic Steering Agent

### Steering Agent Class
```csharp
public class SteeringAgent : MonoBehaviour
{
    [Header("Steering Properties")]
    public float maxSpeed = 10f;
    public float maxForce = 10f;
    public float mass = 1f;
    
    [Header("Arrive Properties")]
    public float slowingRadius = 5f;
    
    private Vector3 velocity;
    private Vector3 acceleration;
    private Rigidbody rb;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        if (rb == null)
            rb = gameObject.AddComponent<Rigidbody>();
        
        rb.mass = mass;
        velocity = Vector3.zero;
        acceleration = Vector3.zero;
    }
    
    void Update()
    {
        // Calculate steering force
        Vector3 steering = CalculateSteering();
        
        // Apply force
        ApplyForce(steering);
        
        // Update velocity
        velocity += acceleration * Time.deltaTime;
        velocity = Vector3.ClampMagnitude(velocity, maxSpeed);
        
        // Update position
        transform.position += velocity * Time.deltaTime;
        
        // Rotate to face direction of movement
        if (velocity.magnitude > 0.1f)
        {
            transform.rotation = Quaternion.LookRotation(velocity);
        }
        
        // Reset acceleration
        acceleration = Vector3.zero;
    }
    
    void ApplyForce(Vector3 force)
    {
        acceleration += force / mass;
    }
    
    protected virtual Vector3 CalculateSteering()
    {
        return Vector3.zero; // Override in subclasses
    }
}
```

---

## Basic Steering Behaviors

### Seek Behavior
```csharp
public class SeekBehavior : SteeringAgent
{
    public Transform target;
    
    protected override Vector3 CalculateSteering()
    {
        if (target == null) return Vector3.zero;
        
        // Desired velocity points toward target
        Vector3 desired = (target.position - transform.position).normalized * maxSpeed;
        
        // Steering force
        Vector3 steer = desired - velocity;
        
        // Limit steering force
        return Vector3.ClampMagnitude(steer, maxForce);
    }
}
```

### Flee Behavior
```csharp
public class FleeBehavior : SteeringAgent
{
    public Transform target;
    public float panicDistance = 10f;
    
    protected override Vector3 CalculateSteering()
    {
        if (target == null) return Vector3.zero;
        
        float distance = Vector3.Distance(transform.position, target.position);
        
        // Only flee if within panic distance
        if (distance > panicDistance) return Vector3.zero;
        
        // Desired velocity points away from target
        Vector3 desired = -(target.position - transform.position).normalized * maxSpeed;
        
        // Steering force
        Vector3 steer = desired - velocity;
        
        return Vector3.ClampMagnitude(steer, maxForce);
    }
}
```

### Arrive Behavior
```csharp
public class ArriveBehavior : SteeringAgent
{
    public Transform target;
    
    protected override Vector3 CalculateSteering()
    {
        if (target == null) return Vector3.zero;
        
        Vector3 toTarget = target.position - transform.position;
        float distance = toTarget.magnitude;
        
        // If we're close enough, stop
        if (distance < 0.5f)
        {
            velocity = Vector3.zero;
            return Vector3.zero;
        }
        
        // Calculate desired speed based on distance
        float desiredSpeed;
        if (distance < slowingRadius)
        {
            // Slow down as we approach
            desiredSpeed = maxSpeed * (distance / slowingRadius);
        }
        else
        {
            desiredSpeed = maxSpeed;
        }
        
        // Desired velocity
        Vector3 desired = toTarget.normalized * desiredSpeed;
        
        // Steering force
        Vector3 steer = desired - velocity;
        
        return Vector3.ClampMagnitude(steer, maxForce);
    }
}
```

### Wander Behavior
```csharp
public class WanderBehavior : SteeringAgent
{
    public float wanderRadius = 3f;
    public float wanderDistance = 5f;
    public float wanderJitter = 1f;
    
    private Vector3 wanderTarget;
    
    void Start()
    {
        base.Start();
        // Initialize wander target on circle
        float angle = Random.Range(0f, 360f) * Mathf.Deg2Rad;
        wanderTarget = new Vector3(
            Mathf.Cos(angle) * wanderRadius,
            0,
            Mathf.Sin(angle) * wanderRadius
        );
    }
    
    protected override Vector3 CalculateSteering()
    {
        // Add jitter to wander target
        wanderTarget += new Vector3(
            Random.Range(-1f, 1f) * wanderJitter,
            0,
            Random.Range(-1f, 1f) * wanderJitter
        );
        
        // Normalize and scale to wander radius
        wanderTarget = wanderTarget.normalized * wanderRadius;
        
        // Calculate target position in world space
        Vector3 targetLocal = wanderTarget + Vector3.forward * wanderDistance;
        Vector3 targetWorld = transform.TransformPoint(targetLocal);
        
        // Seek the wander target
        Vector3 desired = (targetWorld - transform.position).normalized * maxSpeed;
        Vector3 steer = desired - velocity;
        
        return Vector3.ClampMagnitude(steer, maxForce);
    }
}
```

---

## Advanced Behaviors

### Pursuit Behavior
```csharp
public class PursuitBehavior : SteeringAgent
{
    public SteeringAgent target;
    
    protected override Vector3 CalculateSteering()
    {
        if (target == null) return Vector3.zero;
        
        // Predict future position
        Vector3 toTarget = target.transform.position - transform.position;
        float distance = toTarget.magnitude;
        float timeToReach = distance / maxSpeed;
        
        Vector3 predictedPosition = target.transform.position + target.velocity * timeToReach;
        
        // Seek predicted position
        Vector3 desired = (predictedPosition - transform.position).normalized * maxSpeed;
        Vector3 steer = desired - velocity;
        
        return Vector3.ClampMagnitude(steer, maxForce);
    }
}
```

### Obstacle Avoidance
```csharp
public class ObstacleAvoidance : SteeringAgent
{
    public float lookAheadDistance = 5f;
    public float avoidanceForce = 10f;
    public LayerMask obstacleLayer;
    
    protected override Vector3 CalculateSteering()
    {
        // Cast ray ahead
        RaycastHit hit;
        if (Physics.Raycast(
            transform.position,
            velocity.normalized,
            out hit,
            lookAheadDistance,
            obstacleLayer
        ))
        {
            // Calculate avoidance force
            Vector3 avoidance = hit.normal * avoidanceForce;
            
            // Weight by distance (closer = stronger)
            float distanceFactor = 1f - (hit.distance / lookAheadDistance);
            avoidance *= distanceFactor;
            
            return Vector3.ClampMagnitude(avoidance, maxForce);
        }
        
        return Vector3.zero;
    }
}
```

---

## Behavior Blending

### Weighted Blending
```csharp
public class BlendedSteering : SteeringAgent
{
    [System.Serializable]
    public class BehaviorWeight
    {
        public SteeringBehavior behavior;
        public float weight;
    }
    
    public List<BehaviorWeight> behaviors = new List<BehaviorWeight>();
    
    protected override Vector3 CalculateSteering()
    {
        Vector3 totalForce = Vector3.zero;
        
        foreach (var bw in behaviors)
        {
            if (bw.behavior != null)
            {
                Vector3 force = bw.behavior.CalculateSteering();
                totalForce += force * bw.weight;
            }
        }
        
        return Vector3.ClampMagnitude(totalForce, maxForce);
    }
}
```

### Prioritized Behaviors
```csharp
public class PrioritizedSteering : SteeringAgent
{
    public List<SteeringBehavior> behaviors = new List<SteeringBehavior>();
    public float threshold = 0.2f;
    
    protected override Vector3 CalculateSteering()
    {
        Vector3 totalForce = Vector3.zero;
        
        foreach (var behavior in behaviors)
        {
            Vector3 force = behavior.CalculateSteering();
            float magnitude = force.magnitude;
            
            if (magnitude > threshold)
            {
                // Use this behavior and stop
                return Vector3.ClampMagnitude(force, maxForce);
            }
            
            totalForce += force;
            
            // Check if we have enough force
            if (totalForce.magnitude > maxForce)
            {
                return Vector3.ClampMagnitude(totalForce, maxForce);
            }
        }
        
        return totalForce;
    }
}
```

---

## Vehicle Physics

### Basic Vehicle Controller
```csharp
public class VehicleController : MonoBehaviour
{
    [Header("Vehicle Properties")]
    public float motorTorque = 1500f;
    public float brakeTorque = 3000f;
    public float maxSteerAngle = 30f;
    public float centerOfMassOffset = -0.5f;
    
    [Header("Wheels")]
    public WheelCollider[] frontWheels;
    public WheelCollider[] rearWheels;
    public Transform[] wheelMeshes;
    
    private Rigidbody rb;
    private float motorInput;
    private float steerInput;
    private float brakeInput;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.centerOfMass = new Vector3(0, centerOfMassOffset, 0);
    }
    
    void Update()
    {
        // Get input
        motorInput = Input.GetAxis("Vertical");
        steerInput = Input.GetAxis("Horizontal");
        brakeInput = Input.GetKey(KeyCode.Space) ? 1f : 0f;
        
        // Apply steering
        float steerAngle = steerInput * maxSteerAngle;
        foreach (var wheel in frontWheels)
        {
            wheel.steerAngle = steerAngle;
        }
        
        // Apply motor
        float torque = motorInput * motorTorque;
        foreach (var wheel in frontWheels)
        {
            wheel.motorTorque = torque;
        }
        foreach (var wheel in rearWheels)
        {
            wheel.motorTorque = torque;
        }
        
        // Apply brakes
        float brake = brakeInput * brakeTorque;
        foreach (var wheel in frontWheels)
        {
            wheel.brakeTorque = brake;
        }
        foreach (var wheel in rearWheels)
        {
            wheel.brakeTorque = brake;
        }
        
        // Update wheel meshes
        UpdateWheelMeshes();
    }
    
    void UpdateWheelMeshes()
    {
        for (int i = 0; i < wheelMeshes.Length; i++)
        {
            WheelCollider wheel = i < frontWheels.Length ? frontWheels[i] : rearWheels[i - frontWheels.Length];
            
            Vector3 position;
            Quaternion rotation;
            wheel.GetWorldPose(out position, out rotation);
            
            wheelMeshes[i].position = position;
            wheelMeshes[i].rotation = rotation;
        }
    }
}
```

### Drivetrain Types
```csharp
public enum DrivetrainType
{
    FrontWheelDrive,
    RearWheelDrive,
    AllWheelDrive
}

public class DrivetrainController : MonoBehaviour
{
    public DrivetrainType drivetrainType = DrivetrainType.FrontWheelDrive;
    public float frontPowerSplit = 0.6f; // For AWD
    
    private VehicleController vehicle;
    
    void ApplyTorque(float torque)
    {
        switch (drivetrainType)
        {
            case DrivetrainType.FrontWheelDrive:
                ApplyTorqueToWheels(vehicle.frontWheels, torque);
                ApplyTorqueToWheels(vehicle.rearWheels, 0f);
                break;
                
            case DrivetrainType.RearWheelDrive:
                ApplyTorqueToWheels(vehicle.frontWheels, 0f);
                ApplyTorqueToWheels(vehicle.rearWheels, torque);
                break;
                
            case DrivetrainType.AllWheelDrive:
                ApplyTorqueToWheels(vehicle.frontWheels, torque * frontPowerSplit);
                ApplyTorqueToWheels(vehicle.rearWheels, torque * (1f - frontPowerSplit));
                break;
        }
    }
    
    void ApplyTorqueToWheels(WheelCollider[] wheels, float torque)
    {
        foreach (var wheel in wheels)
        {
            wheel.motorTorque = torque;
        }
    }
}
```

---

## Flocking Implementation

### Flocking Agent
```csharp
public class FlockingAgent : SteeringAgent
{
    public float separationRadius = 2f;
    public float alignmentRadius = 5f;
    public float cohesionRadius = 5f;
    
    public float separationWeight = 1.5f;
    public float alignmentWeight = 1f;
    public float cohesionWeight = 1f;
    
    private List<FlockingAgent> neighbors = new List<FlockingAgent>();
    
    protected override Vector3 CalculateSteering()
    {
        FindNeighbors();
        
        Vector3 separation = CalculateSeparation() * separationWeight;
        Vector3 alignment = CalculateAlignment() * alignmentWeight;
        Vector3 cohesion = CalculateCohesion() * cohesionWeight;
        
        return Vector3.ClampMagnitude(separation + alignment + cohesion, maxForce);
    }
    
    void FindNeighbors()
    {
        neighbors.Clear();
        FlockingAgent[] allAgents = FindObjectsOfType<FlockingAgent>();
        
        foreach (var agent in allAgents)
        {
            if (agent != this)
            {
                float distance = Vector3.Distance(transform.position, agent.transform.position);
                if (distance < cohesionRadius)
                {
                    neighbors.Add(agent);
                }
            }
        }
    }
    
    Vector3 CalculateSeparation()
    {
        Vector3 steer = Vector3.zero;
        int count = 0;
        
        foreach (var neighbor in neighbors)
        {
            float distance = Vector3.Distance(transform.position, neighbor.transform.position);
            if (distance < separationRadius && distance > 0)
            {
                Vector3 diff = (transform.position - neighbor.transform.position).normalized;
                diff /= distance; // Weight by distance
                steer += diff;
                count++;
            }
        }
        
        if (count > 0)
        {
            steer /= count;
            steer = steer.normalized * maxSpeed;
            steer -= velocity;
        }
        
        return steer;
    }
    
    Vector3 CalculateAlignment()
    {
        Vector3 sum = Vector3.zero;
        int count = 0;
        
        foreach (var neighbor in neighbors)
        {
            float distance = Vector3.Distance(transform.position, neighbor.transform.position);
            if (distance < alignmentRadius)
            {
                sum += neighbor.velocity;
                count++;
            }
        }
        
        if (count > 0)
        {
            sum /= count;
            sum = sum.normalized * maxSpeed;
            return sum - velocity;
        }
        
        return Vector3.zero;
    }
    
    Vector3 CalculateCohesion()
    {
        Vector3 sum = Vector3.zero;
        int count = 0;
        
        foreach (var neighbor in neighbors)
        {
            float distance = Vector3.Distance(transform.position, neighbor.transform.position);
            if (distance < cohesionRadius)
            {
                sum += neighbor.transform.position;
                count++;
            }
        }
        
        if (count > 0)
        {
            sum /= count;
            return Seek(sum);
        }
        
        return Vector3.zero;
    }
    
    Vector3 Seek(Vector3 target)
    {
        Vector3 desired = (target - transform.position).normalized * maxSpeed;
        return desired - velocity;
    }
}
```

---

## Exercises for Self-Learning

1. **Basic Behaviors**: Implement seek, flee, arrive behaviors
2. **Obstacle Avoidance**: Add obstacle avoidance to steering agent
3. **Flocking**: Create flocking system with multiple agents
4. **Vehicle Physics**: Implement basic vehicle with wheels
5. **Drivetrain**: Add different drivetrain types to vehicle

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)
- **Topologies**: Learn drivetrain topologies (10-5-steeringphysics-topologies/)

---

## References
- Unity WheelCollider documentation
- Unity Rigidbody physics
- Steering behaviors implementation guides
- Vehicle physics tutorials

