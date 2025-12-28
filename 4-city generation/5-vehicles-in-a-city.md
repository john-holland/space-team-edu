# City Generation: Vehicles in a City

## Overview
Vehicle systems for procedurally generated cities, including traffic simulation, pathfinding, vehicle spawning, and AI behavior.

## Learning Objectives
- Spawn and manage vehicles in generated cities
- Implement traffic simulation
- Create vehicle pathfinding on road networks
- Handle vehicle physics and movement
- Optimize vehicle systems for performance

---

## Vehicle System Architecture

### Vehicle Manager
```csharp
public class VehicleManager : MonoBehaviour
{
    [Header("Vehicle Settings")]
    public GameObject vehiclePrefab;
    public int maxVehicles = 100;
    public float spawnRate = 2f;
    
    [Header("Traffic Settings")]
    public float trafficDensity = 0.5f;
    public bool enableTrafficLights = true;
    
    private List<Vehicle> vehicles = new List<Vehicle>();
    private RoadNetwork roadNetwork;
    private float nextSpawnTime;
    
    void Update()
    {
        if (vehicles.Count < maxVehicles && Time.time >= nextSpawnTime)
        {
            SpawnVehicle();
            nextSpawnTime = Time.time + spawnRate;
        }
        
        UpdateVehicles();
    }
}
```

### Vehicle Class
```csharp
public class Vehicle : MonoBehaviour
{
    [Header("Vehicle Properties")]
    public float maxSpeed = 50f;
    public float acceleration = 10f;
    public float braking = 20f;
    public float turnSpeed = 90f;
    
    [Header("Current State")]
    public Road currentRoad;
    public float roadPosition; // 0 to 1 along road
    public float speed;
    public VehicleState state;
    
    private Vector3 targetPosition;
    private Rigidbody rb;
    
    public enum VehicleState
    {
        Driving,
        Stopped,
        Turning,
        Parking
    }
}
```

---

## Vehicle Spawning

### Spawn on Road Network
```csharp
void SpawnVehicle()
{
    // Select random road
    Road spawnRoad = roadNetwork.GetRandomRoad();
    if (spawnRoad == null) return;
    
    // Random position along road
    float t = Random.Range(0f, 1f);
    Vector3 spawnPosition = Vector3.Lerp(spawnRoad.start, spawnRoad.end, t);
    
    // Spawn vehicle
    GameObject vehicleObj = Instantiate(vehiclePrefab, spawnPosition, Quaternion.identity);
    Vehicle vehicle = vehicleObj.GetComponent<Vehicle>();
    
    if (vehicle != null)
    {
        vehicle.currentRoad = spawnRoad;
        vehicle.roadPosition = t;
        vehicle.targetPosition = GetDestination();
        vehicles.Add(vehicle);
    }
}
```

### Spawn at Intersections
```csharp
void SpawnAtIntersection(Intersection intersection)
{
    // Get roads connected to intersection
    List<Road> connectedRoads = roadNetwork.GetRoadsAtIntersection(intersection);
    
    if (connectedRoads.Count > 0)
    {
        Road road = connectedRoads[Random.Range(0, connectedRoads.Count)];
        SpawnVehicleOnRoad(road, 0f); // Start at intersection
    }
}
```

---

## Road Following

### Follow Road Segment
```csharp
public class RoadFollower
{
    public static void FollowRoad(Vehicle vehicle, float deltaTime)
    {
        Road road = vehicle.currentRoad;
        if (road == null) return;
        
        // Update position along road
        float roadLength = road.Length;
        float distance = vehicle.speed * deltaTime;
        vehicle.roadPosition += distance / roadLength;
        
        // Clamp to road bounds
        vehicle.roadPosition = Mathf.Clamp01(vehicle.roadPosition);
        
        // Calculate world position
        Vector3 position = Vector3.Lerp(road.start, road.end, vehicle.roadPosition);
        
        // Calculate direction
        Vector3 direction = (road.end - road.start).normalized;
        
        // Update transform
        vehicle.transform.position = position;
        vehicle.transform.rotation = Quaternion.LookRotation(direction);
        
        // Check if reached end of road
        if (vehicle.roadPosition >= 1f)
        {
            OnReachedRoadEnd(vehicle);
        }
    }
    
    static void OnReachedRoadEnd(Vehicle vehicle)
    {
        // Find next road at intersection
        Intersection intersection = FindIntersectionAt(vehicle.currentRoad.end);
        Road nextRoad = ChooseNextRoad(vehicle, intersection);
        
        if (nextRoad != null)
        {
            vehicle.currentRoad = nextRoad;
            vehicle.roadPosition = 0f;
        }
        else
        {
            // No valid next road, remove vehicle
            Destroy(vehicle.gameObject);
        }
    }
}
```

---

## Pathfinding for Vehicles

### A* on Road Network
```csharp
public class VehiclePathfinding
{
    public static List<Road> FindPath(RoadNetwork network, Vector3 start, Vector3 end)
    {
        Road startRoad = network.GetNearestRoad(start);
        Road endRoad = network.GetNearestRoad(end);
        
        if (startRoad == null || endRoad == null)
            return null;
        
        // A* pathfinding
        Dictionary<Road, Road> cameFrom = new Dictionary<Road, Road>();
        Dictionary<Road, float> gScore = new Dictionary<Road, float>();
        Dictionary<Road, float> fScore = new Dictionary<Road, float>();
        
        List<Road> openSet = new List<Road> { startRoad };
        HashSet<Road> closedSet = new HashSet<Road>();
        
        gScore[startRoad] = 0f;
        fScore[startRoad] = Heuristic(startRoad, endRoad);
        
        while (openSet.Count > 0)
        {
            Road current = GetLowestFScore(openSet, fScore);
            
            if (current == endRoad)
            {
                return ReconstructPath(cameFrom, current);
            }
            
            openSet.Remove(current);
            closedSet.Add(current);
            
            foreach (Road neighbor in network.GetConnectedRoads(current))
            {
                if (closedSet.Contains(neighbor))
                    continue;
                
                float tentativeGScore = gScore[current] + current.Length;
                
                if (!gScore.ContainsKey(neighbor) || tentativeGScore < gScore[neighbor])
                {
                    cameFrom[neighbor] = current;
                    gScore[neighbor] = tentativeGScore;
                    fScore[neighbor] = tentativeGScore + Heuristic(neighbor, endRoad);
                    
                    if (!openSet.Contains(neighbor))
                    {
                        openSet.Add(neighbor);
                    }
                }
            }
        }
        
        return null; // No path found
    }
    
    static float Heuristic(Road a, Road b)
    {
        return Vector3.Distance(a.end, b.start);
    }
    
    static List<Road> ReconstructPath(Dictionary<Road, Road> cameFrom, Road current)
    {
        List<Road> path = new List<Road> { current };
        
        while (cameFrom.ContainsKey(current))
        {
            current = cameFrom[current];
            path.Insert(0, current);
        }
        
        return path;
    }
}
```

---

## Traffic Simulation

### Vehicle Spacing
```csharp
public class TrafficSimulation
{
    public static void UpdateTraffic(List<Vehicle> vehicles)
    {
        // Sort vehicles by road and position
        vehicles.Sort((a, b) =>
        {
            if (a.currentRoad != b.currentRoad)
                return a.currentRoad.GetHashCode().CompareTo(b.currentRoad.GetHashCode());
            return a.roadPosition.CompareTo(b.roadPosition);
        });
        
        // Check spacing between vehicles
        for (int i = 1; i < vehicles.Count; i++)
        {
            Vehicle current = vehicles[i];
            Vehicle ahead = vehicles[i - 1];
            
            if (current.currentRoad == ahead.currentRoad)
            {
                float distance = (ahead.roadPosition - current.roadPosition) * current.currentRoad.Length;
                
                if (distance < minFollowingDistance)
                {
                    // Too close, slow down
                    current.speed = Mathf.Max(0, ahead.speed - brakingForce * Time.deltaTime);
                }
                else
                {
                    // Safe distance, accelerate
                    current.speed = Mathf.Min(current.maxSpeed, current.speed + acceleration * Time.deltaTime);
                }
            }
        }
    }
}
```

### Traffic Lights
```csharp
public class TrafficLight : MonoBehaviour
{
    public enum LightState
    {
        Red,
        Yellow,
        Green
    }
    
    public LightState currentState = LightState.Red;
    public float redDuration = 5f;
    public float yellowDuration = 2f;
    public float greenDuration = 5f;
    
    private float timer;
    
    void Update()
    {
        timer += Time.deltaTime;
        
        switch (currentState)
        {
            case LightState.Red:
                if (timer >= redDuration)
                {
                    currentState = LightState.Green;
                    timer = 0f;
                }
                break;
            case LightState.Yellow:
                if (timer >= yellowDuration)
                {
                    currentState = LightState.Red;
                    timer = 0f;
                }
                break;
            case LightState.Green:
                if (timer >= greenDuration)
                {
                    currentState = LightState.Yellow;
                    timer = 0f;
                }
                break;
        }
    }
    
    public bool CanProceed()
    {
        return currentState == LightState.Green;
    }
}
```

### Vehicle Response to Traffic Lights
```csharp
void CheckTrafficLight(Vehicle vehicle)
{
    // Find traffic light ahead
    TrafficLight light = FindTrafficLightAhead(vehicle);
    
    if (light != null)
    {
        float distanceToLight = GetDistanceToLight(vehicle, light);
        
        if (distanceToLight < stoppingDistance)
        {
            if (!light.CanProceed())
            {
                // Stop at light
                vehicle.state = Vehicle.VehicleState.Stopped;
                vehicle.speed = 0f;
            }
            else
            {
                // Proceed
                vehicle.state = Vehicle.VehicleState.Driving;
            }
        }
    }
}
```

---

## Vehicle Physics

### Simple Vehicle Physics
```csharp
public class VehiclePhysics : MonoBehaviour
{
    private Rigidbody rb;
    private float motorTorque;
    private float steeringAngle;
    
    void Update()
    {
        // Get input (or AI control)
        float motor = GetMotorInput();
        float steering = GetSteeringInput();
        
        // Apply forces
        ApplyMotorForce(motor);
        ApplySteering(steering);
    }
    
    void ApplyMotorForce(float input)
    {
        Vector3 force = transform.forward * input * motorPower;
        rb.AddForce(force);
        
        // Limit speed
        if (rb.velocity.magnitude > maxSpeed)
        {
            rb.velocity = rb.velocity.normalized * maxSpeed;
        }
    }
    
    void ApplySteering(float input)
    {
        steeringAngle = input * maxSteeringAngle;
        transform.Rotate(0, steeringAngle * Time.deltaTime, 0);
    }
}
```

---

## Vehicle AI Behavior

### Basic Driving AI
```csharp
public class VehicleAI : MonoBehaviour
{
    private Vehicle vehicle;
    private List<Road> path;
    private int currentPathIndex;
    
    void Update()
    {
        if (path == null || path.Count == 0)
        {
            // Find new destination
            Vector3 destination = GetRandomDestination();
            path = VehiclePathfinding.FindPath(roadNetwork, transform.position, destination);
            currentPathIndex = 0;
        }
        
        // Follow current road
        if (currentPathIndex < path.Count)
        {
            Road currentRoad = path[currentPathIndex];
            FollowRoad(currentRoad);
            
            // Check if reached end of road
            if (ReachedRoadEnd(currentRoad))
            {
                currentPathIndex++;
            }
        }
    }
    
    void FollowRoad(Road road)
    {
        // Simple road following
        Vector3 direction = (road.end - road.start).normalized;
        transform.rotation = Quaternion.LookRotation(direction);
        transform.position += transform.forward * speed * Time.deltaTime;
    }
}
```

---

## Performance Optimization

### Vehicle Pooling
```csharp
public class VehiclePool
{
    private Queue<Vehicle> pool = new Queue<Vehicle>();
    private GameObject vehiclePrefab;
    
    public Vehicle GetVehicle()
    {
        if (pool.Count > 0)
        {
            Vehicle vehicle = pool.Dequeue();
            vehicle.gameObject.SetActive(true);
            return vehicle;
        }
        
        return Instantiate(vehiclePrefab).GetComponent<Vehicle>();
    }
    
    public void ReturnVehicle(Vehicle vehicle)
    {
        vehicle.gameObject.SetActive(false);
        pool.Enqueue(vehicle);
    }
}
```

### LOD for Vehicles
```csharp
void UpdateVehicleLOD(Vector3 cameraPosition)
{
    foreach (Vehicle vehicle in vehicles)
    {
        float distance = Vector3.Distance(cameraPosition, vehicle.transform.position);
        
        if (distance > lodDistance)
        {
            // Disable detailed components
            vehicle.detailedMesh.SetActive(false);
            vehicle.simpleMesh.SetActive(true);
        }
        else
        {
            vehicle.detailedMesh.SetActive(true);
            vehicle.simpleMesh.SetActive(false);
        }
    }
}
```

---

## Exercises for Self-Learning

1. **Vehicle Spawning**: Implement vehicle spawning on road network
2. **Road Following**: Create road following system
3. **Pathfinding**: Implement A* pathfinding for vehicles
4. **Traffic Simulation**: Add traffic spacing and lights
5. **AI Behavior**: Create basic vehicle AI

---

## Next Steps
- **Usage**: Review city generation use cases (1-usage.md)
- **Implementation**: See city generation code (3-unity-implementation.md)
- **Physics**: Review city physics (4-city-physics.md)

---

## References
- A* pathfinding algorithm
- Traffic simulation techniques
- Vehicle physics
- Unity AI systems
