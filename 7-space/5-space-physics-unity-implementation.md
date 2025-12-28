# Space Physics: Unity Implementation

## Overview
Detailed implementation of space physics in Unity, including N-body simulation, numerical integration methods, and advanced orbital mechanics.

## Learning Objectives
- Implement N-body physics simulation
- Use different numerical integration methods
- Handle precision issues at large scales
- Optimize physics performance

---

## N-Body Physics System

### N-Body Simulator
```csharp
public class NBodySimulator : MonoBehaviour
{
    public CelestialBody[] bodies;
    public float gravitationalConstant = 6.674e-11f;
    public IntegrationMethod integrationMethod = IntegrationMethod.RK4;
    public float timeStep = 0.01f;
    
    public enum IntegrationMethod
    {
        Euler,
        Verlet,
        RK4
    }
    
    private struct BodyState
    {
        public Vector3 position;
        public Vector3 velocity;
        public float mass;
    }
    
    private BodyState[] currentStates;
    private BodyState[] previousStates; // For Verlet
    
    void Start()
    {
        InitializeStates();
    }
    
    void InitializeStates()
    {
        currentStates = new BodyState[bodies.Length];
        previousStates = new BodyState[bodies.Length];
        
        for (int i = 0; i < bodies.Length; i++)
        {
            currentStates[i].position = bodies[i].transform.position;
            currentStates[i].velocity = bodies[i].GetComponent<Rigidbody>().velocity;
            currentStates[i].mass = bodies[i].mass;
            
            previousStates[i] = currentStates[i];
        }
    }
    
    void FixedUpdate()
    {
        switch (integrationMethod)
        {
            case IntegrationMethod.Euler:
                IntegrateEuler();
                break;
            case IntegrationMethod.Verlet:
                IntegrateVerlet();
                break;
            case IntegrationMethod.RK4:
                IntegrateRK4();
                break;
        }
        
        UpdateTransforms();
    }
    
    void IntegrateEuler()
    {
        for (int i = 0; i < bodies.Length; i++)
        {
            Vector3 acceleration = CalculateAcceleration(i);
            currentStates[i].velocity += acceleration * timeStep;
            currentStates[i].position += currentStates[i].velocity * timeStep;
        }
    }
    
    void IntegrateVerlet()
    {
        for (int i = 0; i < bodies.Length; i++)
        {
            Vector3 acceleration = CalculateAcceleration(i);
            Vector3 newPosition = 2f * currentStates[i].position - previousStates[i].position + 
                                 acceleration * timeStep * timeStep;
            
            previousStates[i] = currentStates[i];
            currentStates[i].position = newPosition;
            currentStates[i].velocity = (newPosition - previousStates[i].position) / timeStep;
        }
    }
    
    void IntegrateRK4()
    {
        BodyState[] k1 = new BodyState[bodies.Length];
        BodyState[] k2 = new BodyState[bodies.Length];
        BodyState[] k3 = new BodyState[bodies.Length];
        BodyState[] k4 = new BodyState[bodies.Length];
        BodyState[] tempStates = new BodyState[bodies.Length];
        
        // k1
        for (int i = 0; i < bodies.Length; i++)
        {
            k1[i].velocity = CalculateAcceleration(i);
            k1[i].position = currentStates[i].velocity;
            tempStates[i] = currentStates[i];
        }
        
        // k2
        for (int i = 0; i < bodies.Length; i++)
        {
            tempStates[i].position += k1[i].position * (timeStep / 2f);
            tempStates[i].velocity += k1[i].velocity * (timeStep / 2f);
        }
        for (int i = 0; i < bodies.Length; i++)
        {
            k2[i].velocity = CalculateAcceleration(i, tempStates);
            k2[i].position = tempStates[i].velocity;
        }
        
        // k3
        for (int i = 0; i < bodies.Length; i++)
        {
            tempStates[i].position = currentStates[i].position + k2[i].position * (timeStep / 2f);
            tempStates[i].velocity = currentStates[i].velocity + k2[i].velocity * (timeStep / 2f);
        }
        for (int i = 0; i < bodies.Length; i++)
        {
            k3[i].velocity = CalculateAcceleration(i, tempStates);
            k3[i].position = tempStates[i].velocity;
        }
        
        // k4
        for (int i = 0; i < bodies.Length; i++)
        {
            tempStates[i].position = currentStates[i].position + k3[i].position * timeStep;
            tempStates[i].velocity = currentStates[i].velocity + k3[i].velocity * timeStep;
        }
        for (int i = 0; i < bodies.Length; i++)
        {
            k4[i].velocity = CalculateAcceleration(i, tempStates);
            k4[i].position = tempStates[i].velocity;
        }
        
        // Combine
        for (int i = 0; i < bodies.Length; i++)
        {
            currentStates[i].velocity += (k1[i].velocity + 2f * k2[i].velocity + 
                                         2f * k3[i].velocity + k4[i].velocity) * (timeStep / 6f);
            currentStates[i].position += (k1[i].position + 2f * k2[i].position + 
                                         2f * k3[i].position + k4[i].position) * (timeStep / 6f);
        }
    }
    
    Vector3 CalculateAcceleration(int bodyIndex)
    {
        return CalculateAcceleration(bodyIndex, currentStates);
    }
    
    Vector3 CalculateAcceleration(int bodyIndex, BodyState[] states)
    {
        Vector3 acceleration = Vector3.zero;
        BodyState currentBody = states[bodyIndex];
        
        for (int j = 0; j < bodies.Length; j++)
        {
            if (j == bodyIndex) continue;
            
            Vector3 direction = states[j].position - currentBody.position;
            float distance = direction.magnitude;
            
            if (distance < 0.1f) continue; // Avoid division by zero
            
            float forceMagnitude = gravitationalConstant * states[j].mass / (distance * distance);
            acceleration += direction.normalized * forceMagnitude;
        }
        
        return acceleration;
    }
    
    void UpdateTransforms()
    {
        for (int i = 0; i < bodies.Length; i++)
        {
            bodies[i].transform.position = currentStates[i].position;
            Rigidbody rb = bodies[i].GetComponent<Rigidbody>();
            if (rb != null)
                rb.velocity = currentStates[i].velocity;
        }
    }
}
```

---

## Adaptive Time Step

### Adaptive Integrator
```csharp
public class AdaptiveIntegrator : MonoBehaviour
{
    public float initialTimeStep = 0.01f;
    public float minTimeStep = 0.001f;
    public float maxTimeStep = 0.1f;
    public float tolerance = 1e-6f;
    
    private float currentTimeStep;
    
    void Start()
    {
        currentTimeStep = initialTimeStep;
    }
    
    public float AdaptiveStep(BodyState[] states, float dt)
    {
        // Take two half steps
        BodyState[] halfStepStates = IntegrateStep(states, dt / 2f);
        BodyState[] fullHalfStepStates = IntegrateStep(halfStepStates, dt / 2f);
        
        // Take one full step
        BodyState[] fullStepStates = IntegrateStep(states, dt);
        
        // Estimate error
        float error = EstimateError(fullStepStates, fullHalfStepStates);
        
        // Adjust time step
        if (error > tolerance)
        {
            // Error too large, reduce step
            currentTimeStep = Mathf.Max(minTimeStep, currentTimeStep * 0.9f);
            return AdaptiveStep(states, currentTimeStep);
        }
        else if (error < tolerance / 2f)
        {
            // Error small, can increase step
            currentTimeStep = Mathf.Min(maxTimeStep, currentTimeStep * 1.1f);
        }
        
        return currentTimeStep;
    }
    
    float EstimateError(BodyState[] state1, BodyState[] state2)
    {
        float maxError = 0f;
        for (int i = 0; i < state1.Length; i++)
        {
            float posError = Vector3.Distance(state1[i].position, state2[i].position);
            float velError = Vector3.Distance(state1[i].velocity, state2[i].velocity);
            maxError = Mathf.Max(maxError, Mathf.Max(posError, velError));
        }
        return maxError;
    }
    
    BodyState[] IntegrateStep(BodyState[] states, float dt)
    {
        // Simplified - would use actual integration method
        return states;
    }
}
```

---

## Precision Management

### Double Precision Positions
```csharp
public class HighPrecisionTransform : MonoBehaviour
{
    private Vector3d precisePosition;
    private Vector3d preciseVelocity;
    private Vector3 localOffset = Vector3.zero;
    
    public Vector3d PrecisePosition
    {
        get { return precisePosition; }
        set { precisePosition = value; UpdateUnityTransform(); }
    }
    
    void UpdateUnityTransform()
    {
        // Convert to Unity's float precision
        Vector3 unityPos = (Vector3)(precisePosition - localOffset);
        
        // Use floating origin if needed
        if (unityPos.magnitude > 10000f)
        {
            localOffset = precisePosition;
            unityPos = Vector3.zero;
        }
        
        transform.localPosition = unityPos;
    }
    
    public void AddVelocity(Vector3d velocity, double deltaTime)
    {
        precisePosition += velocity * deltaTime;
        UpdateUnityTransform();
    }
}

// Double precision vector (simplified - would use proper implementation)
[System.Serializable]
public struct Vector3d
{
    public double x, y, z;
    
    public Vector3d(double x, double y, double z)
    {
        this.x = x;
        this.y = y;
        this.z = z;
    }
    
    public static Vector3d operator +(Vector3d a, Vector3d b)
    {
        return new Vector3d(a.x + b.x, a.y + b.y, a.z + b.z);
    }
    
    public static Vector3d operator *(Vector3d a, double d)
    {
        return new Vector3d(a.x * d, a.y * d, a.z * d);
    }
    
    public static explicit operator Vector3(Vector3d v)
    {
        return new Vector3((float)v.x, (float)v.y, (float)v.z);
    }
}
```

---

## Collision Detection

### Sphere Collision
```csharp
public class CelestialBodyCollision : MonoBehaviour
{
    public CelestialBody body;
    public float collisionRadius;
    
    void OnTriggerEnter(Collider other)
    {
        CelestialBody otherBody = other.GetComponent<CelestialBody>();
        if (otherBody != null)
        {
            HandleCollision(body, otherBody);
        }
    }
    
    void HandleCollision(CelestialBody body1, CelestialBody body2)
    {
        // Simple collision - merge or bounce
        // In reality, would have more complex physics
        
        float totalMass = body1.mass + body2.mass;
        Vector3 centerOfMass = (body1.mass * body1.transform.position + 
                               body2.mass * body2.transform.position) / totalMass;
        
        // Merge smaller into larger
        if (body1.mass > body2.mass)
        {
            body1.mass = totalMass;
            body1.transform.position = centerOfMass;
            Destroy(body2.gameObject);
        }
        else
        {
            body2.mass = totalMass;
            body2.transform.position = centerOfMass;
            Destroy(body1.gameObject);
        }
    }
}
```

---

## Energy Conservation Check

### Energy Monitor
```csharp
public class EnergyMonitor : MonoBehaviour
{
    public CelestialBody[] bodies;
    public float gravitationalConstant = 6.674e-11f;
    
    private float initialEnergy;
    
    void Start()
    {
        initialEnergy = CalculateTotalEnergy();
    }
    
    void Update()
    {
        float currentEnergy = CalculateTotalEnergy();
        float energyError = Mathf.Abs(currentEnergy - initialEnergy) / initialEnergy;
        
        if (energyError > 0.01f) // 1% error
        {
            Debug.LogWarning($"Energy conservation error: {energyError * 100f:F2}%");
        }
    }
    
    float CalculateTotalEnergy()
    {
        float kinetic = 0f;
        float potential = 0f;
        
        for (int i = 0; i < bodies.Length; i++)
        {
            Rigidbody rb = bodies[i].GetComponent<Rigidbody>();
            if (rb != null)
            {
                kinetic += 0.5f * bodies[i].mass * rb.velocity.sqrMagnitude;
            }
            
            for (int j = i + 1; j < bodies.Length; j++)
            {
                float distance = Vector3.Distance(bodies[i].transform.position, 
                                                  bodies[j].transform.position);
                potential -= gravitationalConstant * bodies[i].mass * bodies[j].mass / distance;
            }
        }
        
        return kinetic + potential;
    }
}
```

---

## Exercises for Self-Learning

1. **N-Body Simulation**: Implement N-body physics with multiple integration methods
2. **Adaptive Time Step**: Add adaptive time stepping for better accuracy
3. **Precision Management**: Implement double precision for large scales
4. **Energy Conservation**: Monitor and maintain energy conservation
5. **Collision Handling**: Implement realistic collision physics

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)
- **Implementation**: See basic implementation (3-unity-implementation.md)
- **Physics**: Learn space physics theory (4-space-physics.md)

---

## References
- Numerical integration methods
- N-body problem solutions
- Precision management techniques
- Energy conservation in simulations

