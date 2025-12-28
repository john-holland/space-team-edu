# City Generation: City Physics

## Overview
Physics integration for procedurally generated cities, including collision detection, physics queries, and interaction between city elements and physics systems.

## Learning Objectives
- Integrate city geometry with physics engine
- Implement collision detection for buildings/roads
- Handle physics queries in city environment
- Optimize physics performance for large cities

---

## Physics Integration Overview

### Unity Physics Systems
- **Colliders**: Box, Mesh, Terrain colliders
- **Rigidbodies**: Dynamic physics objects
- **Character Controllers**: Player/NPC movement
- **Raycasting**: Line-of-sight, ground detection
- **Queries**: Overlap, sphere cast, box cast

### City Physics Challenges
- **Scale**: Large number of static objects
- **Performance**: Many colliders impact performance
- **Memory**: Collider data can be large
- **Updates**: Dynamic city changes require updates

---

## Building Collision

### Static Colliders
```csharp
public class BuildingPhysics : MonoBehaviour
{
    void AddCollider(Building building)
    {
        GameObject buildingObj = new GameObject("Building");
        buildingObj.transform.position = building.position;
        
        // Add box collider
        BoxCollider collider = buildingObj.AddComponent<BoxCollider>();
        collider.size = building.size;
        collider.center = new Vector3(0, building.size.y / 2f, 0);
        
        // Mark as static (optimization)
        buildingObj.isStatic = true;
    }
}
```

### Mesh Colliders (Complex Shapes)
```csharp
void AddMeshCollider(Building building, Mesh mesh)
{
    GameObject buildingObj = new GameObject("Building");
    MeshCollider meshCollider = buildingObj.AddComponent<MeshCollider>();
    meshCollider.sharedMesh = mesh;
    meshCollider.convex = false; // For static geometry
    buildingObj.isStatic = true;
}
```

### Collider Optimization
- **Use Box Colliders**: When possible (faster than mesh)
- **Mark Static**: Buildings don't move
- **Combine Colliders**: Reduce collider count
- **LOD Colliders**: Simpler colliders at distance

---

## Road Collision

### Road Surface Collision
```csharp
public class RoadPhysics
{
    void CreateRoadCollider(Road road)
    {
        GameObject roadObj = new GameObject("Road");
        
        // Create box collider for road surface
        BoxCollider collider = roadObj.AddComponent<BoxCollider>();
        
        Vector3 center = (road.start + road.end) / 2f;
        Vector3 direction = (road.end - road.start).normalized;
        float length = road.Length;
        
        collider.size = new Vector3(road.width, 0.1f, length);
        collider.center = center;
        
        // Rotate to match road direction
        roadObj.transform.rotation = Quaternion.LookRotation(direction);
        roadObj.isStatic = true;
    }
}
```

### Terrain Collision
For roads on terrain:
```csharp
void CreateRoadOnTerrain(Road road, Terrain terrain)
{
    // Use terrain collider (already exists)
    // Roads follow terrain height
    // Adjust road vertices to terrain height
}
```

---

## Physics Queries

### Ground Detection
```csharp
public class GroundDetection
{
    public static bool GetGroundHeight(Vector3 position, out float height, out Vector3 normal)
    {
        RaycastHit hit;
        if (Physics.Raycast(position + Vector3.up * 100f, Vector3.down, out hit, 200f))
        {
            height = hit.point.y;
            normal = hit.normal;
            return true;
        }
        
        height = 0f;
        normal = Vector3.up;
        return false;
    }
}
```

### Building Intersection Check
```csharp
bool IsPositionInBuilding(Vector3 position)
{
    Collider[] colliders = Physics.OverlapSphere(position, 0.1f);
    
    foreach (Collider col in colliders)
    {
        if (col.CompareTag("Building"))
            return true;
    }
    
    return false;
}
```

### Line of Sight
```csharp
bool HasLineOfSight(Vector3 from, Vector3 to)
{
    Vector3 direction = to - from;
    float distance = direction.magnitude;
    
    RaycastHit hit;
    if (Physics.Raycast(from, direction.normalized, out hit, distance))
    {
        // Check if hit building (not just terrain)
        return !hit.collider.CompareTag("Building");
    }
    
    return true; // No obstruction
}
```

---

## Character Controller Integration

### NavMesh for Pathfinding
```csharp
public class CityNavMesh
{
    void BakeNavMesh()
    {
        // Mark buildings as obstacles
        foreach (Building building in buildings)
        {
            GameObject obj = GetBuildingObject(building);
            NavMeshObstacle obstacle = obj.AddComponent<NavMeshObstacle>();
            obstacle.carving = true;
            obstacle.shape = NavMeshObstacleShape.Box;
            obstacle.size = building.size;
        }
        
        // Bake NavMesh
        NavMeshBuilder.BuildNavMesh();
    }
}
```

### Character Movement
```csharp
public class CityCharacterController : MonoBehaviour
{
    private CharacterController controller;
    private float gravity = -9.81f;
    private Vector3 velocity;
    
    void Update()
    {
        // Ground check
        if (controller.isGrounded && velocity.y < 0)
        {
            velocity.y = 0f;
        }
        
        // Apply gravity
        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);
        
        // Check for ground
        if (!controller.isGrounded)
        {
            // Raycast to find ground
            RaycastHit hit;
            if (Physics.Raycast(transform.position, Vector3.down, out hit, 2f))
            {
                // Adjust position to ground
            }
        }
    }
}
```

---

## Vehicle Physics

### Road Following
```csharp
public class VehicleOnRoad : MonoBehaviour
{
    void FollowRoad()
    {
        // Find nearest road
        Road nearestRoad = FindNearestRoad(transform.position);
        
        // Project position onto road
        Vector3 projected = ProjectOntoRoad(transform.position, nearestRoad);
        
        // Move towards projected position
        transform.position = Vector3.MoveTowards(
            transform.position,
            projected,
            speed * Time.deltaTime
        );
    }
    
    Vector3 ProjectOntoRoad(Vector3 position, Road road)
    {
        Vector3 roadDir = (road.end - road.start).normalized;
        Vector3 toPoint = position - road.start;
        float projection = Vector3.Dot(toPoint, roadDir);
        return road.start + roadDir * Mathf.Clamp(projection, 0, road.Length);
    }
}
```

### Collision Avoidance
```csharp
void AvoidBuildings()
{
    // Sphere cast ahead
    RaycastHit hit;
    float lookAhead = 10f;
    
    if (Physics.SphereCast(
        transform.position,
        vehicleRadius,
        transform.forward,
        out hit,
        lookAhead
    ))
    {
        if (hit.collider.CompareTag("Building"))
        {
            // Steer away
            Vector3 avoidance = Vector3.Cross(transform.forward, Vector3.up);
            transform.Rotate(0, avoidance.y * turnSpeed * Time.deltaTime, 0);
        }
    }
}
```

---

## Performance Optimization

### Collider Pooling
```csharp
public class ColliderPool
{
    private Queue<BoxCollider> pool = new Queue<BoxCollider>();
    
    BoxCollider GetCollider()
    {
        if (pool.Count > 0)
            return pool.Dequeue();
        
        GameObject obj = new GameObject("PooledCollider");
        return obj.AddComponent<BoxCollider>();
    }
    
    void ReturnCollider(BoxCollider collider)
    {
        collider.gameObject.SetActive(false);
        pool.Enqueue(collider);
    }
}
```

### LOD for Physics
```csharp
void UpdatePhysicsLOD(Vector3 cameraPosition)
{
    foreach (Building building in buildings)
    {
        float distance = Vector3.Distance(cameraPosition, building.position);
        
        if (distance > physicsLODDistance)
        {
            // Disable collider (or use simpler collider)
            building.collider.enabled = false;
        }
        else
        {
            building.collider.enabled = true;
        }
    }
}
```

### Spatial Partitioning
Use quad/oct tree for physics queries:
```csharp
public class PhysicsQuadTree
{
    private QuadTree<Collider> colliderTree;
    
    void BuildTree()
    {
        colliderTree = new QuadTree<Collider>(worldBounds);
        
        Collider[] allColliders = FindObjectsOfType<Collider>();
        foreach (Collider col in allColliders)
        {
            colliderTree.Insert(col.bounds.center, col);
        }
    }
    
    Collider[] QueryNearby(Vector3 position, float radius)
    {
        Rect queryRect = new Rect(
            position.x - radius,
            position.z - radius,
            radius * 2,
            radius * 2
        );
        
        return colliderTree.QueryRange(queryRect).ToArray();
    }
}
```

---

## Dynamic City Updates

### Building Destruction
```csharp
void DestroyBuilding(Building building)
{
    // Remove collider
    if (building.collider != null)
    {
        Destroy(building.collider);
    }
    
    // Add debris (rigidbodies)
    CreateDebris(building);
    
    // Update NavMesh
    NavMeshBuilder.BuildNavMesh();
}

void CreateDebris(Building building)
{
    // Create rigidbody pieces
    for (int i = 0; i < debrisCount; i++)
    {
        GameObject debris = GameObject.CreatePrimitive(PrimitiveType.Cube);
        debris.transform.position = building.position + Random.insideUnitSphere;
        Rigidbody rb = debris.AddComponent<Rigidbody>();
        rb.AddExplosionForce(explosionForce, building.position, explosionRadius);
    }
}
```

---

## Exercises for Self-Learning

1. **Collision Setup**: Add colliders to generated buildings
2. **Ground Detection**: Implement ground height queries
3. **Line of Sight**: Create line-of-sight system
4. **Vehicle Physics**: Implement vehicle movement on roads
5. **Performance**: Optimize physics for large city

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See city generation code (3-unity-implementation.md)
- **Vehicles**: Learn vehicle systems (5-vehicles-in-a-city.md)

---

## References
- Unity Physics API
- Character Controller documentation
- NavMesh system
- Performance optimization guides
