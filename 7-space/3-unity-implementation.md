# Space: Unity Implementation

## Overview
Practical implementation of space simulation in Unity, including orbital mechanics, coordinate systems, and large-scale space environments.

## Learning Objectives
- Implement orbital mechanics in Unity
- Handle large-scale coordinate systems
- Create space environments with planets
- Optimize for performance at scale

---

## Basic Orbital System

### Celestial Body
```csharp
public class CelestialBody : MonoBehaviour
{
    [Header("Physical Properties")]
    public float mass = 1e20f; // kg
    public float radius = 1000f; // meters
    public Vector3 initialVelocity = Vector3.zero;
    
    [Header("Orbital Properties")]
    public CelestialBody centralBody; // Body this orbits around
    public bool useOrbitalMechanics = true;
    
    private Rigidbody rb;
    private Vector3 currentVelocity;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        if (rb == null)
            rb = gameObject.AddComponent<Rigidbody>();
        
        rb.mass = mass;
        rb.useGravity = false; // We'll calculate gravity manually
        currentVelocity = initialVelocity;
    }
    
    void FixedUpdate()
    {
        if (useOrbitalMechanics && centralBody != null)
        {
            UpdateOrbitalMotion();
        }
    }
    
    void UpdateOrbitalMotion()
    {
        // Calculate gravitational force
        Vector3 direction = centralBody.transform.position - transform.position;
        float distance = direction.magnitude;
        
        if (distance < 0.1f) return; // Avoid division by zero
        
        // F = G * m1 * m2 / r^2
        float G = 6.674e-11f;
        float forceMagnitude = G * mass * centralBody.mass / (distance * distance);
        Vector3 force = direction.normalized * forceMagnitude;
        
        // Apply force
        Vector3 acceleration = force / mass;
        currentVelocity += acceleration * Time.fixedDeltaTime;
        
        // Update position
        transform.position += currentVelocity * Time.fixedDeltaTime;
    }
    
    public Vector3 GetGravitationalAcceleration(Vector3 position)
    {
        Vector3 direction = transform.position - position;
        float distance = direction.magnitude;
        
        if (distance < 0.1f) return Vector3.zero;
        
        float G = 6.674e-11f;
        float accelerationMagnitude = G * mass / (distance * distance);
        return direction.normalized * accelerationMagnitude;
    }
}
```

---

## Spacecraft Controller

### Spacecraft
```csharp
public class Spacecraft : MonoBehaviour
{
    [Header("Spacecraft Properties")]
    public float mass = 1000f; // kg
    public float maxThrust = 10000f; // Newtons
    public float specificImpulse = 300f; // seconds
    
    [Header("Controls")]
    public KeyCode thrustForward = KeyCode.W;
    public KeyCode thrustBackward = KeyCode.S;
    public KeyCode thrustLeft = KeyCode.A;
    public KeyCode thrustRight = KeyCode.D;
    public KeyCode thrustUp = KeyCode.Space;
    public KeyCode thrustDown = KeyCode.LeftShift;
    
    private Rigidbody rb;
    private Vector3 velocity;
    private float fuel = 1000f; // kg
    private float maxFuel = 1000f;
    
    private List<CelestialBody> nearbyBodies = new List<CelestialBody>();
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        if (rb == null)
            rb = gameObject.AddComponent<Rigidbody>();
        
        rb.mass = mass;
        rb.useGravity = false;
        velocity = Vector3.zero;
    }
    
    void FixedUpdate()
    {
        // Get thrust input
        Vector3 thrustDirection = GetThrustInput();
        
        // Apply thrust
        if (thrustDirection.magnitude > 0 && fuel > 0)
        {
            float thrustMagnitude = maxThrust * thrustDirection.magnitude;
            Vector3 thrust = transform.TransformDirection(thrustDirection) * thrustMagnitude;
            
            // Consume fuel
            float fuelConsumption = thrustMagnitude / (specificImpulse * 9.81f) * Time.fixedDeltaTime;
            fuel = Mathf.Max(0, fuel - fuelConsumption);
            
            // Apply thrust
            Vector3 acceleration = thrust / (mass + fuel);
            velocity += acceleration * Time.fixedDeltaTime;
        }
        
        // Apply gravitational forces from nearby bodies
        ApplyGravitationalForces();
        
        // Update position
        transform.position += velocity * Time.fixedDeltaTime;
        
        // Update mass (decreases as fuel is consumed)
        rb.mass = mass + fuel;
    }
    
    Vector3 GetThrustInput()
    {
        Vector3 input = Vector3.zero;
        
        if (Input.GetKey(thrustForward)) input.z += 1f;
        if (Input.GetKey(thrustBackward)) input.z -= 1f;
        if (Input.GetKey(thrustRight)) input.x += 1f;
        if (Input.GetKey(thrustLeft)) input.x -= 1f;
        if (Input.GetKey(thrustUp)) input.y += 1f;
        if (Input.GetKey(thrustDown)) input.y -= 1f;
        
        return input.normalized;
    }
    
    void ApplyGravitationalForces()
    {
        FindNearbyBodies();
        
        foreach (var body in nearbyBodies)
        {
            Vector3 acceleration = body.GetGravitationalAcceleration(transform.position);
            velocity += acceleration * Time.fixedDeltaTime;
        }
    }
    
    void FindNearbyBodies()
    {
        nearbyBodies.Clear();
        CelestialBody[] allBodies = FindObjectsOfType<CelestialBody>();
        
        foreach (var body in allBodies)
        {
            float distance = Vector3.Distance(transform.position, body.transform.position);
            // Only consider bodies within reasonable distance
            if (distance < 100000f) // 100 km
            {
                nearbyBodies.Add(body);
            }
        }
    }
    
    public float GetVelocity()
    {
        return velocity.magnitude;
    }
    
    public float GetAltitude(CelestialBody body)
    {
        return Vector3.Distance(transform.position, body.transform.position) - body.radius;
    }
}
```

---

## Coordinate System Management

### Floating Origin System
```csharp
public class FloatingOrigin : MonoBehaviour
{
    public Transform referenceObject; // Usually the player/camera
    public float threshold = 10000f; // Reset when this far from origin
    
    private Vector3 originOffset = Vector3.zero;
    
    void LateUpdate()
    {
        if (referenceObject == null) return;
        
        Vector3 referencePosition = referenceObject.position;
        
        // Check if we're too far from origin
        if (referencePosition.magnitude > threshold)
        {
            // Shift everything back toward origin
            originOffset += referencePosition;
            
            // Move all relevant objects
            MoveAllObjects(-referencePosition);
            
            Debug.Log($"Floating origin shifted. New offset: {originOffset}");
        }
    }
    
    void MoveAllObjects(Vector3 offset)
    {
        // Move all celestial bodies
        CelestialBody[] bodies = FindObjectsOfType<CelestialBody>();
        foreach (var body in bodies)
        {
            body.transform.position += offset;
        }
        
        // Move all spacecraft
        Spacecraft[] spacecraft = FindObjectsOfType<Spacecraft>();
        foreach (var craft in spacecraft)
        {
            craft.transform.position += offset;
        }
        
        // Move camera
        Camera.main.transform.position += offset;
    }
    
    public Vector3 ToWorldPosition(Vector3 localPosition)
    {
        return localPosition + originOffset;
    }
    
    public Vector3 ToLocalPosition(Vector3 worldPosition)
    {
        return worldPosition - originOffset;
    }
}
```

---

## Solar System Setup

### Solar System Manager
```csharp
public class SolarSystemManager : MonoBehaviour
{
    [Header("Star")]
    public CelestialBody star;
    public float starMass = 1.989e30f; // Sun mass in kg
    public float starRadius = 696340000f; // Sun radius in meters
    
    [Header("Planets")]
    public CelestialBody[] planets;
    public float[] planetMasses;
    public float[] planetRadii;
    public float[] orbitalDistances; // Distance from star
    public float[] orbitalVelocities; // Initial orbital velocities
    
    void Start()
    {
        SetupStar();
        SetupPlanets();
    }
    
    void SetupStar()
    {
        if (star != null)
        {
            star.mass = starMass;
            star.radius = starRadius;
            star.transform.position = Vector3.zero;
            star.useOrbitalMechanics = false; // Star doesn't orbit
        }
    }
    
    void SetupPlanets()
    {
        if (planets == null || planets.Length == 0) return;
        
        for (int i = 0; i < planets.Length; i++)
        {
            if (planets[i] == null) continue;
            
            // Set properties
            if (i < planetMasses.Length)
                planets[i].mass = planetMasses[i];
            if (i < planetRadii.Length)
                planets[i].radius = planetRadii[i];
            
            // Position in orbit
            if (i < orbitalDistances.Length)
            {
                float angle = (360f / planets.Length) * i * Mathf.Deg2Rad;
                Vector3 position = new Vector3(
                    Mathf.Cos(angle) * orbitalDistances[i],
                    0,
                    Mathf.Sin(angle) * orbitalDistances[i]
                );
                planets[i].transform.position = position;
            }
            
            // Set initial velocity for orbit
            if (i < orbitalVelocities.Length && star != null)
            {
                Vector3 direction = (star.transform.position - planets[i].transform.position).normalized;
                Vector3 perpendicular = Vector3.Cross(direction, Vector3.up).normalized;
                planets[i].initialVelocity = perpendicular * orbitalVelocities[i];
            }
            
            // Set central body
            planets[i].centralBody = star;
        }
    }
}
```

---

## Orbital Visualization

### Orbit Trajectory Renderer
```csharp
public class OrbitTrajectory : MonoBehaviour
{
    public CelestialBody body;
    public int trajectoryPoints = 100;
    public float trajectoryTime = 100f; // seconds to simulate ahead
    public LineRenderer lineRenderer;
    
    void Start()
    {
        if (lineRenderer == null)
            lineRenderer = gameObject.AddComponent<LineRenderer>();
        
        lineRenderer.useWorldSpace = true;
        lineRenderer.loop = false;
    }
    
    void Update()
    {
        if (body == null || body.centralBody == null) return;
        
        DrawTrajectory();
    }
    
    void DrawTrajectory()
    {
        Vector3[] points = new Vector3[trajectoryPoints];
        Vector3 currentPos = body.transform.position;
        Vector3 currentVel = body.GetComponent<Rigidbody>().velocity;
        
        float dt = trajectoryTime / trajectoryPoints;
        
        for (int i = 0; i < trajectoryPoints; i++)
        {
            points[i] = currentPos;
            
            // Simulate one step
            Vector3 direction = body.centralBody.transform.position - currentPos;
            float distance = direction.magnitude;
            
            if (distance > 0.1f)
            {
                float G = 6.674e-11f;
                float accelerationMagnitude = G * body.centralBody.mass / (distance * distance);
                Vector3 acceleration = direction.normalized * accelerationMagnitude;
                
                currentVel += acceleration * dt;
                currentPos += currentVel * dt;
            }
        }
        
        lineRenderer.positionCount = trajectoryPoints;
        lineRenderer.SetPositions(points);
    }
}
```

---

## Time Scale Control

### Time Manager
```csharp
public class TimeManager : MonoBehaviour
{
    [Header("Time Control")]
    public float timeScale = 1f;
    public float minTimeScale = 0.1f;
    public float maxTimeScale = 100f;
    
    [Header("UI")]
    public UnityEngine.UI.Slider timeScaleSlider;
    public UnityEngine.UI.Text timeScaleText;
    
    void Start()
    {
        if (timeScaleSlider != null)
        {
            timeScaleSlider.minValue = minTimeScale;
            timeScaleSlider.maxValue = maxTimeScale;
            timeScaleSlider.value = timeScale;
            timeScaleSlider.onValueChanged.AddListener(OnTimeScaleChanged);
        }
    }
    
    void Update()
    {
        // Keyboard controls
        if (Input.GetKeyDown(KeyCode.Equals) || Input.GetKeyDown(KeyCode.Plus))
            IncreaseTimeScale();
        if (Input.GetKeyDown(KeyCode.Minus))
            DecreaseTimeScale();
        if (Input.GetKeyDown(KeyCode.Alpha0))
            timeScale = 1f;
        
        Time.timeScale = timeScale;
        
        if (timeScaleText != null)
            timeScaleText.text = $"Time Scale: {timeScale:F2}x";
    }
    
    void OnTimeScaleChanged(float value)
    {
        timeScale = value;
    }
    
    void IncreaseTimeScale()
    {
        timeScale = Mathf.Min(maxTimeScale, timeScale * 2f);
        if (timeScaleSlider != null)
            timeScaleSlider.value = timeScale;
    }
    
    void DecreaseTimeScale()
    {
        timeScale = Mathf.Max(minTimeScale, timeScale / 2f);
        if (timeScaleSlider != null)
            timeScaleSlider.value = timeScale;
    }
}
```

---

## Performance Optimization

### LOD System for Celestial Bodies
```csharp
public class CelestialBodyLOD : MonoBehaviour
{
    public CelestialBody body;
    public float[] lodDistances = { 1000f, 5000f, 50000f };
    public GameObject[] lodMeshes;
    
    private Transform cameraTransform;
    
    void Start()
    {
        cameraTransform = Camera.main.transform;
    }
    
    void Update()
    {
        if (body == null || cameraTransform == null) return;
        
        float distance = Vector3.Distance(cameraTransform.position, body.transform.position);
        
        int lodLevel = 0;
        for (int i = 0; i < lodDistances.Length; i++)
        {
            if (distance > lodDistances[i])
                lodLevel = i + 1;
        }
        
        lodLevel = Mathf.Min(lodLevel, lodMeshes.Length - 1);
        
        // Enable/disable LOD meshes
        for (int i = 0; i < lodMeshes.Length; i++)
        {
            if (lodMeshes[i] != null)
                lodMeshes[i].SetActive(i == lodLevel);
        }
    }
}
```

---

## Exercises for Self-Learning

1. **Orbital System**: Create a simple solar system with planets
2. **Spacecraft Control**: Implement spacecraft with thrust controls
3. **Floating Origin**: Add floating origin system for large scales
4. **Trajectory Visualization**: Draw orbital trajectories
5. **Time Control**: Implement time scale controls

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)
- **Physics**: Learn space physics (4-space-physics.md)
- **Physics Implementation**: See physics details (5-space-physics-unity-implementation.md)

---

## References
- Unity Rigidbody physics
- Orbital mechanics simulation
- Floating origin techniques
- Time scale management
