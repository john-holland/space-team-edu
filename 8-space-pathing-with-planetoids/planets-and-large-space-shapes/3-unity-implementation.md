# Planets and Large Space Shapes: Unity Implementation

## Overview
Practical implementation of planetary physics and large space structures in Unity, including gravity systems, atmospheric physics, and pathfinding for massive objects.

## Learning Objectives
- Implement planetary gravity systems
- Create atmospheric physics
- Handle large-scale coordinate systems
- Optimize for massive objects

---

## Planetary Gravity System

### Planetary Gravity Field
```csharp
public class PlanetaryGravity : MonoBehaviour
{
    [Header("Planet Properties")]
    public float mass = 5.972e24f; // Earth mass in kg
    public float radius = 6371000f; // Earth radius in meters
    public float gravitationalConstant = 6.674e-11f;
    
    [Header("Gravity Field")]
    public bool useNonSphericalGravity = false;
    public float oblatenessCoefficient = 0.001f; // J2
    
    public Vector3 GetGravitationalAcceleration(Vector3 position)
    {
        Vector3 direction = transform.position - position;
        float distance = direction.magnitude;
        
        if (distance < 0.1f) return Vector3.zero;
        
        if (useNonSphericalGravity)
        {
            return GetNonSphericalGravity(position, direction, distance);
        }
        else
        {
            // Standard spherical gravity
            float accelerationMagnitude = gravitationalConstant * mass / (distance * distance);
            return direction.normalized * accelerationMagnitude;
        }
    }
    
    Vector3 GetNonSphericalGravity(Vector3 position, Vector3 direction, float distance)
    {
        // Calculate latitude
        Vector3 toPosition = (position - transform.position).normalized;
        float latitude = Mathf.Asin(toPosition.y) * Mathf.Rad2Deg;
        float cosLat = Mathf.Cos(latitude * Mathf.Deg2Rad);
        
        // Base gravity
        float baseGravity = gravitationalConstant * mass / (distance * distance);
        
        // Oblateness correction
        float correction = 1f + (5f / 2f) * oblatenessCoefficient * (3f * cosLat * cosLat - 1f) / 2f;
        
        float accelerationMagnitude = baseGravity * correction;
        return direction.normalized * accelerationMagnitude;
    }
    
    public float GetSurfaceGravity()
    {
        return gravitationalConstant * mass / (radius * radius);
    }
    
    void OnDrawGizmos()
    {
        // Draw gravity field visualization
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(transform.position, radius);
        
        // Draw gravity vectors (simplified)
        for (int i = 0; i < 8; i++)
        {
            float angle = (360f / 8f) * i * Mathf.Deg2Rad;
            Vector3 pos = transform.position + new Vector3(
                Mathf.Cos(angle) * radius * 2f,
                0,
                Mathf.Sin(angle) * radius * 2f
            );
            Vector3 gravity = GetGravitationalAcceleration(pos);
            Gizmos.DrawRay(pos, gravity.normalized * 1000f);
        }
    }
}
```

---

## Atmospheric System

### Atmospheric Physics
```csharp
public class AtmosphericSystem : MonoBehaviour
{
    [Header("Atmosphere Properties")]
    public float surfacePressure = 101325f; // Pa (Earth sea level)
    public float scaleHeight = 8400f; // meters
    public float surfaceDensity = 1.225f; // kg/m³ (air at sea level)
    public float gasConstant = 287.05f; // J/(kg·K)
    public float surfaceTemperature = 288.15f; // K (15°C)
    
    [Header("Atmosphere Limits")]
    public float atmosphereHeight = 100000f; // meters
    
    public float GetPressureAtAltitude(float altitude)
    {
        if (altitude > atmosphereHeight)
            return 0f;
        
        return surfacePressure * Mathf.Exp(-altitude / scaleHeight);
    }
    
    public float GetDensityAtAltitude(float altitude)
    {
        if (altitude > atmosphereHeight)
            return 0f;
        
        return surfaceDensity * Mathf.Exp(-altitude / scaleHeight);
    }
    
    public float GetTemperatureAtAltitude(float altitude)
    {
        // Simplified - linear temperature lapse
        float lapseRate = 0.0065f; // K/m
        return surfaceTemperature - lapseRate * altitude;
    }
    
    public Vector3 GetDragForce(Rigidbody rb, float dragCoefficient, float crossSectionalArea)
    {
        float altitude = GetAltitude(rb.transform.position);
        float density = GetDensityAtAltitude(altitude);
        
        if (density < 0.001f) return Vector3.zero; // Negligible atmosphere
        
        Vector3 velocity = rb.velocity;
        float speed = velocity.magnitude;
        
        // Drag = 0.5 * density * v² * Cd * A
        float dragMagnitude = 0.5f * density * speed * speed * dragCoefficient * crossSectionalArea;
        Vector3 dragForce = -velocity.normalized * dragMagnitude;
        
        return dragForce;
    }
    
    float GetAltitude(Vector3 position)
    {
        PlanetaryGravity planet = GetComponent<PlanetaryGravity>();
        if (planet == null) return 0f;
        
        float distance = Vector3.Distance(position, planet.transform.position);
        return distance - planet.radius;
    }
    
    public bool IsInAtmosphere(Vector3 position)
    {
        float altitude = GetAltitude(position);
        return altitude >= 0 && altitude <= atmosphereHeight;
    }
}
```

---

## Crater Generation

### Crater System
```csharp
public class CraterGenerator : MonoBehaviour
{
    [Header("Crater Properties")]
    public int craterCount = 100;
    public float minCraterSize = 100f;
    public float maxCraterSize = 5000f;
    public AnimationCurve sizeDistribution = AnimationCurve.Linear(0, 0, 1, 1);
    
    private List<Crater> craters = new List<Crater>();
    
    void Start()
    {
        GenerateCraters();
    }
    
    void GenerateCraters()
    {
        PlanetaryGravity planet = GetComponent<PlanetaryGravity>();
        if (planet == null) return;
        
        float planetRadius = planet.radius;
        
        for (int i = 0; i < craterCount; i++)
        {
            // Random position on sphere
            Vector3 direction = Random.onUnitSphere;
            Vector3 position = planet.transform.position + direction * planetRadius;
            
            // Random size
            float size = Mathf.Lerp(minCraterSize, maxCraterSize, 
                                   sizeDistribution.Evaluate(Random.value));
            
            Crater crater = new Crater
            {
                position = position,
                radius = size,
                depth = size * 0.1f // Typical depth is 10% of diameter
            };
            
            craters.Add(crater);
        }
    }
    
    public float GetSurfaceHeight(Vector3 position)
    {
        PlanetaryGravity planet = GetComponent<PlanetaryGravity>();
        if (planet == null) return 0f;
        
        float baseHeight = planet.radius;
        
        // Subtract crater depths
        foreach (var crater in craters)
        {
            float distance = Vector3.Distance(position, crater.position);
            if (distance < crater.radius)
            {
                // Crater depth profile (parabolic)
                float normalizedDistance = distance / crater.radius;
                float depth = crater.depth * (1f - normalizedDistance * normalizedDistance);
                baseHeight -= depth;
            }
        }
        
        return baseHeight;
    }
    
    void OnDrawGizmos()
    {
        Gizmos.color = Color.red;
        foreach (var crater in craters)
        {
            Gizmos.DrawWireSphere(crater.position, crater.radius);
        }
    }
}

[System.Serializable]
public class Crater
{
    public Vector3 position;
    public float radius;
    public float depth;
}
```

---

## Large Space Structure

### Dyson Sphere
```csharp
public class DysonSphere : MonoBehaviour
{
    [Header("Structure Properties")]
    public Transform star; // Central star
    public float sphereRadius = 1.5e11f; // 1 AU in meters
    public int segments = 64;
    public Material sphereMaterial;
    
    [Header("Physics")]
    public float structuralIntegrity = 1e12f; // Pa
    public float massPerArea = 1000f; // kg/m²
    
    private GameObject sphereMesh;
    
    void Start()
    {
        GenerateSphere();
    }
    
    void GenerateSphere()
    {
        sphereMesh = new GameObject("DysonSphere");
        sphereMesh.transform.SetParent(transform);
        
        MeshFilter meshFilter = sphereMesh.AddComponent<MeshFilter>();
        MeshRenderer meshRenderer = sphereMesh.AddComponent<MeshRenderer>();
        meshRenderer.material = sphereMaterial;
        
        Mesh mesh = CreateSphereMesh(segments);
        meshFilter.mesh = mesh;
        
        // Add collider for pathfinding
        SphereCollider collider = sphereMesh.AddComponent<SphereCollider>();
        collider.radius = sphereRadius;
        collider.isTrigger = true;
    }
    
    Mesh CreateSphereMesh(int segments)
    {
        Mesh mesh = new Mesh();
        List<Vector3> vertices = new List<Vector3>();
        List<int> triangles = new List<int>();
        
        // Generate sphere vertices
        for (int y = 0; y <= segments; y++)
        {
            float theta = (float)y / segments * Mathf.PI;
            float sinTheta = Mathf.Sin(theta);
            float cosTheta = Mathf.Cos(theta);
            
            for (int x = 0; x <= segments; x++)
            {
                float phi = (float)x / segments * 2f * Mathf.PI;
                float sinPhi = Mathf.Sin(phi);
                float cosPhi = Mathf.Cos(phi);
                
                Vector3 vertex = new Vector3(
                    cosPhi * sinTheta,
                    cosTheta,
                    sinPhi * sinTheta
                ) * sphereRadius;
                
                vertices.Add(vertex);
            }
        }
        
        // Generate triangles
        for (int y = 0; y < segments; y++)
        {
            for (int x = 0; x < segments; x++)
            {
                int current = y * (segments + 1) + x;
                int next = current + segments + 1;
                
                triangles.Add(current);
                triangles.Add(next);
                triangles.Add(current + 1);
                
                triangles.Add(current + 1);
                triangles.Add(next);
                triangles.Add(next + 1);
            }
        }
        
        mesh.vertices = vertices.ToArray();
        mesh.triangles = triangles.ToArray();
        mesh.RecalculateNormals();
        
        return mesh;
    }
    
    public bool IsInsideSphere(Vector3 position)
    {
        if (star == null) return false;
        float distance = Vector3.Distance(position, star.position);
        return distance < sphereRadius;
    }
    
    public Vector3 GetSurfaceNormal(Vector3 position)
    {
        if (star == null) return Vector3.up;
        return (position - star.position).normalized;
    }
}
```

---

## Pathfinding Around Planets

### Planetary Pathfinder
```csharp
public class PlanetaryPathfinder : MonoBehaviour
{
    public PlanetaryGravity planet;
    public float safeAltitude = 1000f; // meters above surface
    public float pathfindingRadius = 100000f; // meters
    
    public List<Vector3> FindPathAroundPlanet(Vector3 start, Vector3 end)
    {
        if (planet == null) return new List<Vector3> { start, end };
        
        // Calculate safe altitudes
        float startAltitude = GetSafeAltitude(start);
        float endAltitude = GetSafeAltitude(end);
        
        // Create path that maintains safe altitude
        List<Vector3> path = new List<Vector3>();
        path.Add(start);
        
        // Intermediate waypoints
        int waypointCount = 10;
        for (int i = 1; i < waypointCount; i++)
        {
            float t = (float)i / waypointCount;
            Vector3 waypoint = Vector3.Lerp(start, end, t);
            waypoint = AdjustToSafeAltitude(waypoint, Mathf.Lerp(startAltitude, endAltitude, t));
            path.Add(waypoint);
        }
        
        path.Add(end);
        return path;
    }
    
    float GetSafeAltitude(Vector3 position)
    {
        if (planet == null) return safeAltitude;
        
        float distance = Vector3.Distance(position, planet.transform.position);
        float altitude = distance - planet.radius;
        
        return Mathf.Max(altitude, safeAltitude);
    }
    
    Vector3 AdjustToSafeAltitude(Vector3 position, float targetAltitude)
    {
        if (planet == null) return position;
        
        Vector3 direction = (position - planet.transform.position).normalized;
        float safeDistance = planet.radius + targetAltitude;
        return planet.transform.position + direction * safeDistance;
    }
    
    public bool IsPathSafe(List<Vector3> path)
    {
        if (planet == null) return true;
        
        foreach (var point in path)
        {
            float altitude = GetSafeAltitude(point);
            if (altitude < safeAltitude)
                return false;
        }
        
        return true;
    }
}
```

---

## Tidal Forces

### Tidal Force Calculator
```csharp
public class TidalForces : MonoBehaviour
{
    public CelestialBody primaryBody; // Planet
    public CelestialBody secondaryBody; // Moon
    public float gravitationalConstant = 6.674e-11f;
    
    public Vector3 GetTidalAcceleration(Vector3 position)
    {
        if (primaryBody == null || secondaryBody == null)
            return Vector3.zero;
        
        Vector3 toPrimary = primaryBody.transform.position - position;
        Vector3 toSecondary = secondaryBody.transform.position - position;
        
        float distToPrimary = toPrimary.magnitude;
        float distToSecondary = toSecondary.magnitude;
        
        // Tidal acceleration from secondary body
        Vector3 accelPrimary = toPrimary.normalized * 
            (gravitationalConstant * primaryBody.mass / (distToPrimary * distToPrimary));
        
        Vector3 accelSecondary = toSecondary.normalized * 
            (gravitationalConstant * secondaryBody.mass / (distToSecondary * distToSecondary));
        
        // Tidal component (difference)
        return accelSecondary - accelPrimary;
    }
    
    public float GetTidalHeight(Vector3 position)
    {
        // Calculate tidal bulge height
        Vector3 tidalAccel = GetTidalAcceleration(position);
        float magnitude = tidalAccel.magnitude;
        
        // Simplified calculation
        float height = magnitude * 1000f; // Scale factor
        return height;
    }
}
```

---

## Exercises for Self-Learning

1. **Planetary Gravity**: Implement gravity field for planet
2. **Atmospheric System**: Create atmospheric physics system
3. **Crater Generation**: Generate impact craters on planet
4. **Large Structures**: Create Dyson sphere or ring world
5. **Planetary Pathfinding**: Implement pathfinding around planets

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)

---

## References
- Planetary physics simulation
- Atmospheric physics
- Large structure engineering
- Tidal force calculations
