# Space Pathing with Planetoids: Unity Implementation

## Overview
Practical implementation of pathfinding in 3D space with planets and obstacles, including trajectory planning, obstacle avoidance, and multi-scale pathfinding.

## Learning Objectives
- Implement 3D pathfinding in space
- Handle pathfinding around obstacles (planets, moons)
- Create trajectory planning systems
- Optimize for large-scale environments

---

## 3D A* Pathfinding

### 3D Path Node
```csharp
public class SpacePathNode
{
    public Vector3 position;
    public float gCost; // Cost from start
    public float hCost; // Heuristic to goal
    public float fCost { get { return gCost + hCost; } }
    public SpacePathNode parent;
    public bool walkable = true;
    
    public SpacePathNode(Vector3 pos)
    {
        position = pos;
    }
}
```

### 3D A* Pathfinder
```csharp
public class SpaceAStar : MonoBehaviour
{
    public float nodeSpacing = 100f; // meters
    public float searchRadius = 10000f; // meters
    public LayerMask obstacleLayer;
    
    public List<Vector3> FindPath(Vector3 start, Vector3 goal)
    {
        List<SpacePathNode> openSet = new List<SpacePathNode>();
        HashSet<SpacePathNode> closedSet = new HashSet<SpacePathNode>();
        
        SpacePathNode startNode = new SpacePathNode(start);
        SpacePathNode goalNode = new SpacePathNode(goal);
        
        startNode.gCost = 0;
        startNode.hCost = Heuristic(startNode, goalNode);
        
        openSet.Add(startNode);
        
        while (openSet.Count > 0)
        {
            SpacePathNode current = GetLowestFCost(openSet);
            
            if (Vector3.Distance(current.position, goal) < nodeSpacing)
            {
                return ReconstructPath(current);
            }
            
            openSet.Remove(current);
            closedSet.Add(current);
            
            foreach (var neighbor in GetNeighbors(current))
            {
                if (closedSet.Contains(neighbor) || !neighbor.walkable)
                    continue;
                
                float tentativeGCost = current.gCost + Distance(current, neighbor);
                
                if (!openSet.Contains(neighbor))
                {
                    openSet.Add(neighbor);
                }
                else if (tentativeGCost >= neighbor.gCost)
                {
                    continue;
                }
                
                neighbor.parent = current;
                neighbor.gCost = tentativeGCost;
                neighbor.hCost = Heuristic(neighbor, goalNode);
            }
        }
        
        return null; // No path found
    }
    
    List<SpacePathNode> GetNeighbors(SpacePathNode node)
    {
        List<SpacePathNode> neighbors = new List<SpacePathNode>();
        
        // 6-connected neighbors (face-adjacent)
        Vector3[] directions = new Vector3[]
        {
            Vector3.right, Vector3.left,
            Vector3.up, Vector3.down,
            Vector3.forward, Vector3.back
        };
        
        foreach (var dir in directions)
        {
            Vector3 neighborPos = node.position + dir * nodeSpacing;
            
            // Check if position is valid (not in obstacle)
            if (IsValidPosition(neighborPos))
            {
                SpacePathNode neighbor = new SpacePathNode(neighborPos);
                neighbor.walkable = true;
                neighbors.Add(neighbor);
            }
        }
        
        return neighbors;
    }
    
    bool IsValidPosition(Vector3 position)
    {
        // Check for obstacles (planets, etc.)
        Collider[] obstacles = Physics.OverlapSphere(position, nodeSpacing * 0.5f, obstacleLayer);
        return obstacles.Length == 0;
    }
    
    float Heuristic(SpacePathNode a, SpacePathNode b)
    {
        return Vector3.Distance(a.position, b.position);
    }
    
    float Distance(SpacePathNode a, SpacePathNode b)
    {
        return Vector3.Distance(a.position, b.position);
    }
    
    SpacePathNode GetLowestFCost(List<SpacePathNode> nodes)
    {
        SpacePathNode lowest = nodes[0];
        foreach (var node in nodes)
        {
            if (node.fCost < lowest.fCost)
                lowest = node;
        }
        return lowest;
    }
    
    List<Vector3> ReconstructPath(SpacePathNode endNode)
    {
        List<Vector3> path = new List<Vector3>();
        SpacePathNode current = endNode;
        
        while (current != null)
        {
            path.Add(current.position);
            current = current.parent;
        }
        
        path.Reverse();
        return path;
    }
}
```

---

## Trajectory Planning

### Orbital Trajectory Planner
```csharp
public class TrajectoryPlanner : MonoBehaviour
{
    public CelestialBody centralBody;
    public float gravitationalConstant = 6.674e-11f;
    
    public Trajectory CalculateHohmannTransfer(Vector3 startPos, float startRadius, 
                                                float targetRadius)
    {
        Trajectory trajectory = new Trajectory();
        
        // Calculate transfer orbit
        float semiMajorAxis = (startRadius + targetRadius) / 2f;
        
        // Velocity at periapsis (start)
        float v1 = Mathf.Sqrt(gravitationalConstant * centralBody.mass * 
                              (2f / startRadius - 1f / semiMajorAxis));
        
        // Velocity at apoapsis (target)
        float v2 = Mathf.Sqrt(gravitationalConstant * centralBody.mass * 
                              (2f / targetRadius - 1f / semiMajorAxis));
        
        // Circular velocities
        float vCircular1 = Mathf.Sqrt(gravitationalConstant * centralBody.mass / startRadius);
        float vCircular2 = Mathf.Sqrt(gravitationalConstant * centralBody.mass / targetRadius);
        
        // Delta-V required
        trajectory.deltaV1 = v1 - vCircular1;
        trajectory.deltaV2 = vCircular2 - v2;
        trajectory.totalDeltaV = Mathf.Abs(trajectory.deltaV1) + Mathf.Abs(trajectory.deltaV2);
        
        // Transfer time
        trajectory.transferTime = Mathf.PI * Mathf.Sqrt(Mathf.Pow(semiMajorAxis, 3f) / 
                                                        (gravitationalConstant * centralBody.mass));
        
        return trajectory;
    }
    
    public List<Vector3> GenerateTrajectoryPoints(Trajectory trajectory, Vector3 startPos, 
                                                  int numPoints = 100)
    {
        List<Vector3> points = new List<Vector3>();
        
        // Generate points along trajectory
        for (int i = 0; i < numPoints; i++)
        {
            float t = (float)i / (numPoints - 1);
            float angle = t * 2f * Mathf.PI; // Full orbit
            
            // Simplified - would use actual orbital mechanics
            float radius = trajectory.semiMajorAxis;
            Vector3 pos = new Vector3(
                Mathf.Cos(angle) * radius,
                0,
                Mathf.Sin(angle) * radius
            );
            
            points.Add(pos);
        }
        
        return points;
    }
}

[System.Serializable]
public class Trajectory
{
    public float deltaV1;
    public float deltaV2;
    public float totalDeltaV;
    public float transferTime;
    public float semiMajorAxis;
}
```

---

## Obstacle Avoidance

### Planetoid Obstacle
```csharp
public class PlanetoidObstacle : MonoBehaviour
{
    public CelestialBody planet;
    public float safeDistance = 1000f; // meters above surface
    
    public bool IsPositionSafe(Vector3 position)
    {
        if (planet == null) return true;
        
        float distance = Vector3.Distance(position, planet.transform.position);
        float minDistance = planet.radius + safeDistance;
        
        return distance >= minDistance;
    }
    
    public Vector3 GetAvoidanceDirection(Vector3 position, Vector3 direction)
    {
        if (planet == null) return direction;
        
        Vector3 toPlanet = (planet.transform.position - position).normalized;
        Vector3 perpendicular = Vector3.Cross(toPlanet, direction).normalized;
        
        // Steer perpendicular to planet
        return Vector3.Slerp(direction, perpendicular, 0.5f).normalized;
    }
    
    public float GetDistanceToSurface(Vector3 position)
    {
        if (planet == null) return float.MaxValue;
        
        float distance = Vector3.Distance(position, planet.transform.position);
        return distance - planet.radius;
    }
}
```

### Pathfinding with Obstacles
```csharp
public class ObstacleAwarePathfinder : SpaceAStar
{
    public PlanetoidObstacle[] obstacles;
    
    protected override bool IsValidPosition(Vector3 position)
    {
        // Check base validity
        if (!base.IsValidPosition(position))
            return false;
        
        // Check obstacles
        foreach (var obstacle in obstacles)
        {
            if (obstacle != null && !obstacle.IsPositionSafe(position))
                return false;
        }
        
        return true;
    }
    
    protected override float GetCost(SpacePathNode from, SpacePathNode to)
    {
        float baseCost = Distance(from, to);
        
        // Add penalty for being close to obstacles
        float obstaclePenalty = 0f;
        foreach (var obstacle in obstacles)
        {
            if (obstacle != null)
            {
                float distance = obstacle.GetDistanceToSurface(to.position);
                if (distance < 5000f) // Within 5km
                {
                    obstaclePenalty += (5000f - distance) / 100f; // Penalty increases as distance decreases
                }
            }
        }
        
        return baseCost + obstaclePenalty;
    }
}
```

---

## RRT Pathfinding

### RRT Implementation
```csharp
public class SpaceRRT : MonoBehaviour
{
    public float stepSize = 1000f; // meters
    public int maxIterations = 1000;
    public float goalRadius = 500f; // meters
    public LayerMask obstacleLayer;
    
    private List<RRTNode> tree = new List<RRTNode>();
    
    public List<Vector3> FindPath(Vector3 start, Vector3 goal)
    {
        tree.Clear();
        tree.Add(new RRTNode(start, null));
        
        for (int i = 0; i < maxIterations; i++)
        {
            // Sample random point
            Vector3 randomPoint = SampleRandomPoint();
            
            // Find nearest node
            RRTNode nearest = FindNearest(randomPoint);
            
            // Extend toward random point
            Vector3 direction = (randomPoint - nearest.position).normalized;
            Vector3 newPosition = nearest.position + direction * stepSize;
            
            // Check if valid
            if (IsValidPosition(newPosition) && HasLineOfSight(nearest.position, newPosition))
            {
                RRTNode newNode = new RRTNode(newPosition, nearest);
                tree.Add(newNode);
                
                // Check if reached goal
                if (Vector3.Distance(newPosition, goal) < goalRadius)
                {
                    return ReconstructPath(newNode, goal);
                }
            }
        }
        
        return null; // No path found
    }
    
    Vector3 SampleRandomPoint()
    {
        // Sample in search space (could bias toward goal)
        return new Vector3(
            Random.Range(-10000f, 10000f),
            Random.Range(-10000f, 10000f),
            Random.Range(-10000f, 10000f)
        );
    }
    
    RRTNode FindNearest(Vector3 point)
    {
        RRTNode nearest = tree[0];
        float minDist = Vector3.Distance(point, nearest.position);
        
        foreach (var node in tree)
        {
            float dist = Vector3.Distance(point, node.position);
            if (dist < minDist)
            {
                minDist = dist;
                nearest = node;
            }
        }
        
        return nearest;
    }
    
    bool IsValidPosition(Vector3 position)
    {
        Collider[] obstacles = Physics.OverlapSphere(position, stepSize * 0.5f, obstacleLayer);
        return obstacles.Length == 0;
    }
    
    bool HasLineOfSight(Vector3 from, Vector3 to)
    {
        Vector3 direction = to - from;
        float distance = direction.magnitude;
        return !Physics.Raycast(from, direction.normalized, distance, obstacleLayer);
    }
    
    List<Vector3> ReconstructPath(RRTNode endNode, Vector3 goal)
    {
        List<Vector3> path = new List<Vector3> { goal };
        RRTNode current = endNode;
        
        while (current != null)
        {
            path.Insert(0, current.position);
            current = current.parent;
        }
        
        return path;
    }
}

public class RRTNode
{
    public Vector3 position;
    public RRTNode parent;
    
    public RRTNode(Vector3 pos, RRTNode parent)
    {
        position = pos;
        this.parent = parent;
    }
}
```

---

## Multi-Scale Pathfinding

### Hierarchical Space Pathfinder
```csharp
public class HierarchicalSpacePathfinder : MonoBehaviour
{
    public enum PathfindingScale
    {
        SolarSystem,  // Between planets
        Planetary,    // Around planet
        Orbital,      // In orbit
        Surface       // On surface
    }
    
    public List<Vector3> FindPath(Vector3 start, Vector3 end, PathfindingScale scale)
    {
        switch (scale)
        {
            case PathfindingScale.SolarSystem:
                return FindSolarSystemPath(start, end);
            case PathfindingScale.Planetary:
                return FindPlanetaryPath(start, end);
            case PathfindingScale.Orbital:
                return FindOrbitalPath(start, end);
            case PathfindingScale.Surface:
                return FindSurfacePath(start, end);
            default:
                return null;
        }
    }
    
    List<Vector3> FindSolarSystemPath(Vector3 start, Vector3 end)
    {
        // High-level: Navigate between planets
        // Use trajectory planning
        TrajectoryPlanner planner = GetComponent<TrajectoryPlanner>();
        Trajectory trajectory = planner.CalculateHohmannTransfer(start, 
                                                                 Vector3.Distance(start, Vector3.zero),
                                                                 Vector3.Distance(end, Vector3.zero));
        return planner.GenerateTrajectoryPoints(trajectory, start);
    }
    
    List<Vector3> FindPlanetaryPath(Vector3 start, Vector3 end)
    {
        // Mid-level: Navigate around planet, avoid moons
        SpaceAStar pathfinder = GetComponent<SpaceAStar>();
        return pathfinder.FindPath(start, end);
    }
    
    List<Vector3> FindOrbitalPath(Vector3 start, Vector3 end)
    {
        // Low-level: Navigate in orbit
        // Simplified orbital mechanics
        return new List<Vector3> { start, end };
    }
    
    List<Vector3> FindSurfacePath(Vector3 start, Vector3 end)
    {
        // Surface-level: Use standard pathfinding
        // Could use NavMesh or A* on surface
        return new List<Vector3> { start, end };
    }
}
```

---

## Path Visualization

### Trajectory Renderer
```csharp
public class TrajectoryRenderer : MonoBehaviour
{
    public LineRenderer lineRenderer;
    public Material trajectoryMaterial;
    public float lineWidth = 10f;
    
    void Start()
    {
        if (lineRenderer == null)
            lineRenderer = gameObject.AddComponent<LineRenderer>();
        
        lineRenderer.material = trajectoryMaterial;
        lineRenderer.startWidth = lineWidth;
        lineRenderer.endWidth = lineWidth;
        lineRenderer.useWorldSpace = true;
    }
    
    public void DrawPath(List<Vector3> path)
    {
        if (path == null || path.Count == 0) return;
        
        lineRenderer.positionCount = path.Count;
        lineRenderer.SetPositions(path.ToArray());
    }
    
    public void ClearPath()
    {
        lineRenderer.positionCount = 0;
    }
}
```

---

## Exercises for Self-Learning

1. **3D A***: Implement 3D A* pathfinding in space
2. **Trajectory Planning**: Create Hohmann transfer calculator
3. **Obstacle Avoidance**: Add planet/moon avoidance
4. **RRT**: Implement RRT for complex spaces
5. **Multi-Scale**: Create hierarchical pathfinding system

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)

---

## References
- 3D pathfinding algorithms
- Orbital mechanics
- RRT algorithms
- Trajectory optimization
