# Hierarchical Pathfinding: Unity Implementation

## Overview
Practical implementation of hierarchical pathfinding in Unity, including region-based pathfinding, multi-level A*, and integration with Unity's NavMesh system.

## Learning Objectives
- Implement region-based hierarchical pathfinding
- Create abstract graph from detailed graph
- Integrate with Unity NavMesh
- Optimize for large environments

---

## Basic Hierarchical Pathfinding

### Region Class
```csharp
public class PathfindingRegion
{
    public int regionID;
    public Bounds bounds;
    public List<PathNode> nodes = new List<PathNode>();
    public List<PathfindingRegion> neighbors = new List<PathfindingRegion>();
    
    public PathfindingRegion(int id, Bounds b)
    {
        regionID = id;
        bounds = b;
    }
    
    public bool Contains(Vector3 position)
    {
        return bounds.Contains(position);
    }
}
```

### Path Node
```csharp
public class PathNode
{
    public Vector3 position;
    public List<PathNode> connections = new List<PathNode>();
    public float cost = 1f;
    public PathfindingRegion region;
    
    public PathNode(Vector3 pos, PathfindingRegion reg)
    {
        position = pos;
        region = reg;
    }
}
```

---

## Region Graph Creation

### Region Divider
```csharp
public class RegionDivider
{
    public static List<PathfindingRegion> DivideIntoRegions(Bounds worldBounds, int regionsPerAxis)
    {
        List<PathfindingRegion> regions = new List<PathfindingRegion>();
        
        Vector3 regionSize = new Vector3(
            worldBounds.size.x / regionsPerAxis,
            worldBounds.size.y / regionsPerAxis,
            worldBounds.size.z / regionsPerAxis
        );
        
        int regionID = 0;
        for (int x = 0; x < regionsPerAxis; x++)
        {
            for (int z = 0; z < regionsPerAxis; z++)
            {
                Vector3 regionCenter = new Vector3(
                    worldBounds.min.x + (x + 0.5f) * regionSize.x,
                    worldBounds.center.y,
                    worldBounds.min.z + (z + 0.5f) * regionSize.z
                );
                
                Bounds regionBounds = new Bounds(regionCenter, regionSize);
                PathfindingRegion region = new PathfindingRegion(regionID++, regionBounds);
                regions.Add(region);
            }
        }
        
        // Connect neighboring regions
        ConnectNeighbors(regions, regionsPerAxis);
        
        return regions;
    }
    
    static void ConnectNeighbors(List<PathfindingRegion> regions, int regionsPerAxis)
    {
        for (int i = 0; i < regions.Count; i++)
        {
            int x = i / regionsPerAxis;
            int z = i % regionsPerAxis;
            
            // Check 4 neighbors (N, S, E, W)
            if (x > 0) regions[i].neighbors.Add(regions[i - regionsPerAxis]);
            if (x < regionsPerAxis - 1) regions[i].neighbors.Add(regions[i + regionsPerAxis]);
            if (z > 0) regions[i].neighbors.Add(regions[i - 1]);
            if (z < regionsPerAxis - 1) regions[i].neighbors.Add(regions[i + 1]);
        }
    }
}
```

---

## Hierarchical Pathfinding System

### Hierarchical Pathfinder
```csharp
public class HierarchicalPathfinder : MonoBehaviour
{
    public Bounds worldBounds;
    public int regionsPerAxis = 10;
    public float nodeSpacing = 5f;
    
    private List<PathfindingRegion> regions;
    private Dictionary<PathfindingRegion, List<PathNode>> regionNodes;
    
    void Start()
    {
        InitializeRegions();
        GenerateNodes();
    }
    
    void InitializeRegions()
    {
        regions = RegionDivider.DivideIntoRegions(worldBounds, regionsPerAxis);
        regionNodes = new Dictionary<PathfindingRegion, List<PathNode>>();
    }
    
    void GenerateNodes()
    {
        // Generate nodes within each region (simplified - would use NavMesh or waypoints)
        foreach (var region in regions)
        {
            List<PathNode> nodes = new List<PathNode>();
            // Generate grid of nodes in region
            for (float x = region.bounds.min.x; x < region.bounds.max.x; x += nodeSpacing)
            {
                for (float z = region.bounds.min.z; z < region.bounds.max.z; z += nodeSpacing)
                {
                    Vector3 pos = new Vector3(x, region.bounds.center.y, z);
                    // Check if position is valid (not in obstacle)
                    if (IsValidPosition(pos))
                    {
                        PathNode node = new PathNode(pos, region);
                        nodes.Add(node);
                    }
                }
            }
            regionNodes[region] = nodes;
        }
        
        // Connect nodes within and between regions
        ConnectNodes();
    }
    
    bool IsValidPosition(Vector3 pos)
    {
        // Raycast down to check if on ground
        RaycastHit hit;
        if (Physics.Raycast(pos + Vector3.up * 10f, Vector3.down, out hit, 20f))
        {
            return true;
        }
        return false;
    }
    
    void ConnectNodes()
    {
        foreach (var region in regions)
        {
            List<PathNode> nodes = regionNodes[region];
            
            // Connect nodes within region
            for (int i = 0; i < nodes.Count; i++)
            {
                for (int j = i + 1; j < nodes.Count; j++)
                {
                    float distance = Vector3.Distance(nodes[i].position, nodes[j].position);
                    if (distance < nodeSpacing * 1.5f && HasLineOfSight(nodes[i].position, nodes[j].position))
                    {
                        nodes[i].connections.Add(nodes[j]);
                        nodes[j].connections.Add(nodes[i]);
                    }
                }
            }
            
            // Connect to nodes in neighboring regions
            foreach (var neighbor in region.neighbors)
            {
                ConnectRegions(region, neighbor);
            }
        }
    }
    
    void ConnectRegions(PathfindingRegion region1, PathfindingRegion region2)
    {
        List<PathNode> nodes1 = regionNodes[region1];
        List<PathNode> nodes2 = regionNodes[region2];
        
        // Find closest nodes on boundary
        PathNode closest1 = null;
        PathNode closest2 = null;
        float minDist = float.MaxValue;
        
        foreach (var node1 in nodes1)
        {
            foreach (var node2 in nodes2)
            {
                float dist = Vector3.Distance(node1.position, node2.position);
                if (dist < minDist && HasLineOfSight(node1.position, node2.position))
                {
                    minDist = dist;
                    closest1 = node1;
                    closest2 = node2;
                }
            }
        }
        
        if (closest1 != null && closest2 != null)
        {
            closest1.connections.Add(closest2);
            closest2.connections.Add(closest1);
        }
    }
    
    bool HasLineOfSight(Vector3 from, Vector3 to)
    {
        Vector3 direction = to - from;
        float distance = direction.magnitude;
        return !Physics.Raycast(from, direction.normalized, distance);
    }
}
```

---

## Hierarchical A* Pathfinding

### Hierarchical A* Implementation
```csharp
public class HierarchicalAStar
{
    public static List<Vector3> FindPath(HierarchicalPathfinder pathfinder, Vector3 start, Vector3 end)
    {
        // Find regions containing start and end
        PathfindingRegion startRegion = pathfinder.GetRegionAt(start);
        PathfindingRegion endRegion = pathfinder.GetRegionAt(end);
        
        if (startRegion == null || endRegion == null)
            return null;
        
        // High-level: Find path through regions
        List<PathfindingRegion> regionPath = FindRegionPath(pathfinder, startRegion, endRegion);
        
        if (regionPath == null || regionPath.Count == 0)
            return null;
        
        // Low-level: Find detailed path within each region
        List<Vector3> detailedPath = new List<Vector3> { start };
        
        for (int i = 0; i < regionPath.Count - 1; i++)
        {
            PathfindingRegion currentRegion = regionPath[i];
            PathfindingRegion nextRegion = regionPath[i + 1];
            
            // Find path within current region
            Vector3 regionStart = (i == 0) ? start : GetRegionExit(currentRegion, nextRegion);
            Vector3 regionEnd = GetRegionEntry(nextRegion, currentRegion);
            
            List<Vector3> regionPath = FindPathInRegion(pathfinder, currentRegion, regionStart, regionEnd);
            if (regionPath != null)
            {
                detailedPath.AddRange(regionPath);
            }
        }
        
        detailedPath.Add(end);
        return detailedPath;
    }
    
    static List<PathfindingRegion> FindRegionPath(HierarchicalPathfinder pathfinder, 
                                                  PathfindingRegion start, PathfindingRegion end)
    {
        // A* on region graph
        Dictionary<PathfindingRegion, PathfindingRegion> cameFrom = new Dictionary<PathfindingRegion, PathfindingRegion>();
        Dictionary<PathfindingRegion, float> gScore = new Dictionary<PathfindingRegion, float>();
        Dictionary<PathfindingRegion, float> fScore = new Dictionary<PathfindingRegion, float>();
        
        List<PathfindingRegion> openSet = new List<PathfindingRegion> { start };
        HashSet<PathfindingRegion> closedSet = new HashSet<PathfindingRegion>();
        
        gScore[start] = 0f;
        fScore[start] = Heuristic(start, end);
        
        while (openSet.Count > 0)
        {
            PathfindingRegion current = GetLowestFScore(openSet, fScore);
            
            if (current == end)
            {
                return ReconstructRegionPath(cameFrom, current);
            }
            
            openSet.Remove(current);
            closedSet.Add(current);
            
            foreach (var neighbor in current.neighbors)
            {
                if (closedSet.Contains(neighbor))
                    continue;
                
                float tentativeGScore = gScore[current] + RegionDistance(current, neighbor);
                
                if (!gScore.ContainsKey(neighbor) || tentativeGScore < gScore[neighbor])
                {
                    cameFrom[neighbor] = current;
                    gScore[neighbor] = tentativeGScore;
                    fScore[neighbor] = tentativeGScore + Heuristic(neighbor, end);
                    
                    if (!openSet.Contains(neighbor))
                    {
                        openSet.Add(neighbor);
                    }
                }
            }
        }
        
        return null; // No path found
    }
    
    static float Heuristic(PathfindingRegion a, PathfindingRegion b)
    {
        return Vector3.Distance(a.bounds.center, b.bounds.center);
    }
    
    static float RegionDistance(PathfindingRegion a, PathfindingRegion b)
    {
        return Vector3.Distance(a.bounds.center, b.bounds.center);
    }
    
    static PathfindingRegion GetLowestFScore(List<PathfindingRegion> openSet, 
                                             Dictionary<PathfindingRegion, float> fScore)
    {
        PathfindingRegion lowest = openSet[0];
        float lowestScore = fScore[lowest];
        
        foreach (var region in openSet)
        {
            if (fScore.ContainsKey(region) && fScore[region] < lowestScore)
            {
                lowest = region;
                lowestScore = fScore[region];
            }
        }
        
        return lowest;
    }
    
    static List<PathfindingRegion> ReconstructRegionPath(Dictionary<PathfindingRegion, PathfindingRegion> cameFrom, 
                                                         PathfindingRegion current)
    {
        List<PathfindingRegion> path = new List<PathfindingRegion> { current };
        
        while (cameFrom.ContainsKey(current))
        {
            current = cameFrom[current];
            path.Insert(0, current);
        }
        
        return path;
    }
    
    static List<Vector3> FindPathInRegion(HierarchicalPathfinder pathfinder, 
                                         PathfindingRegion region, Vector3 start, Vector3 end)
    {
        // A* within region using nodes
        // Simplified - would use actual node-based A*
        return new List<Vector3> { start, end };
    }
    
    static Vector3 GetRegionExit(PathfindingRegion region, PathfindingRegion next)
    {
        // Find point on boundary between regions
        Vector3 direction = (next.bounds.center - region.bounds.center).normalized;
        return region.bounds.center + direction * (region.bounds.size.magnitude / 2f);
    }
    
    static Vector3 GetRegionEntry(PathfindingRegion region, PathfindingRegion previous)
    {
        return GetRegionExit(region, previous);
    }
}
```

---

## Unity NavMesh Integration

### NavMesh Hierarchical Pathfinding
```csharp
public class NavMeshHierarchicalPathfinder : MonoBehaviour
{
    public void FindPath(Vector3 start, Vector3 end)
    {
        // Use Unity NavMesh for low-level pathfinding
        UnityEngine.AI.NavMeshPath path = new UnityEngine.AI.NavMeshPath();
        
        if (UnityEngine.AI.NavMesh.CalculatePath(start, end, UnityEngine.AI.NavMesh.AllAreas, path))
        {
            // Path found - can use for detailed navigation
            Vector3[] corners = path.corners;
            
            // Optionally simplify path or use for hierarchical system
        }
    }
}
```

---

## Performance Optimization

### Caching
```csharp
public class PathCache
{
    private Dictionary<string, List<Vector3>> cache = new Dictionary<string, List<Vector3>>();
    
    public List<Vector3> GetCachedPath(Vector3 start, Vector3 end)
    {
        string key = $"{start}_{end}";
        if (cache.ContainsKey(key))
            return cache[key];
        return null;
    }
    
    public void CachePath(Vector3 start, Vector3 end, List<Vector3> path)
    {
        string key = $"{start}_{end}";
        cache[key] = path;
    }
}
```

### Async Pathfinding
```csharp
public class AsyncPathfinder : MonoBehaviour
{
    public void FindPathAsync(Vector3 start, Vector3 end, System.Action<List<Vector3>> callback)
    {
        StartCoroutine(FindPathCoroutine(start, end, callback));
    }
    
    IEnumerator FindPathCoroutine(Vector3 start, Vector3 end, System.Action<List<Vector3>> callback)
    {
        List<Vector3> path = HierarchicalAStar.FindPath(pathfinder, start, end);
        yield return null; // Allow frame to complete
        callback?.Invoke(path);
    }
}
```

---

## Exercises for Self-Learning

1. **Region Division**: Implement region division algorithm
2. **Hierarchical A***: Implement full hierarchical A* pathfinding
3. **NavMesh Integration**: Integrate with Unity NavMesh
4. **Optimization**: Add caching and async pathfinding
5. **Visualization**: Visualize regions and paths in editor

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)

---

## References
- Unity NavMesh API
- A* pathfinding algorithm
- Hierarchical pathfinding papers
- Unity coroutines
