# Pathfinding with Dual-Encoded Voxels

## Overview
Pathfinding systems that use dual-encoded voxel data structures, combining spatial representation with pathfinding information for efficient navigation in voxel environments.

## Learning Objectives
- Understand dual-encoded voxel systems
- Learn pathfinding in voxel spaces
- Master voxel-based navigation
- Optimize pathfinding for voxel worlds

---

## What are Dual-Encoded Voxels?

### Definition
Dual-encoded voxels store two types of information:
1. **Spatial Data**: Material, density, or occupancy
2. **Pathfinding Data**: Walkability, cost, connectivity

### Basic Concept
- **Single Voxel**: Contains both spatial and pathfinding info
- **Efficient**: No need for separate pathfinding data structure
- **Coherent**: Pathfinding matches spatial representation
- **Flexible**: Can encode various pathfinding properties

---

## Dual Encoding Schemes

### Scheme 1: Material + Walkability
Each voxel stores:
- **Material ID**: What the voxel is made of
- **Walkability**: Can agents walk through this voxel?

**Encoding**:
```
[Voxel Value: 16 bits]
    Bits 0-7: Material ID (0-255)
    Bit 8: Walkable flag
    Bits 9-15: Reserved
```

### Scheme 2: Density + Cost
Each voxel stores:
- **Density**: How solid is the voxel?
- **Path Cost**: Cost to traverse this voxel

**Encoding**:
```
[Voxel Value: 32 bits]
    Bits 0-15: Density (0-65535)
    Bits 16-31: Path cost (0-65535)
```

### Scheme 3: Occupancy + Connectivity
Each voxel stores:
- **Occupancy**: Is voxel filled?
- **Neighbor Connectivity**: Which neighbors are connected?

**Encoding**:
```
[Voxel Value: 16 bits]
    Bit 0: Occupied
    Bits 1-6: Neighbor connectivity (6 bits for 6 faces)
    Bits 7-15: Reserved
```

---

## Pathfinding Algorithms for Voxels

### A* in Voxel Space

#### Voxel Node
```csharp
public class VoxelNode
{
    public Vector3Int voxelCoord;
    public float gCost; // Cost from start
    public float hCost; // Heuristic to goal
    public float fCost { get { return gCost + hCost; } }
    public VoxelNode parent;
    
    public VoxelNode(Vector3Int coord)
    {
        voxelCoord = coord;
    }
}
```

#### A* Implementation
```csharp
public class VoxelAStar
{
    private VoxelData voxelData;
    
    public List<Vector3> FindPath(Vector3Int start, Vector3Int goal)
    {
        List<VoxelNode> openSet = new List<VoxelNode>();
        HashSet<Vector3Int> closedSet = new HashSet<Vector3Int>();
        
        VoxelNode startNode = new VoxelNode(start);
        startNode.gCost = 0;
        startNode.hCost = Heuristic(start, goal);
        
        openSet.Add(startNode);
        
        while (openSet.Count > 0)
        {
            VoxelNode current = GetLowestFCost(openSet);
            
            if (current.voxelCoord == goal)
            {
                return ReconstructPath(current);
            }
            
            openSet.Remove(current);
            closedSet.Add(current.voxelCoord);
            
            // Check 6-connected neighbors
            Vector3Int[] neighbors = GetNeighbors(current.voxelCoord);
            foreach (var neighborCoord in neighbors)
            {
                if (closedSet.Contains(neighborCoord))
                    continue;
                
                if (!IsWalkable(neighborCoord))
                    continue;
                
                float tentativeGCost = current.gCost + GetCost(current.voxelCoord, neighborCoord);
                
                VoxelNode neighborNode = FindInOpenSet(openSet, neighborCoord);
                if (neighborNode == null)
                {
                    neighborNode = new VoxelNode(neighborCoord);
                    openSet.Add(neighborNode);
                }
                else if (tentativeGCost >= neighborNode.gCost)
                {
                    continue;
                }
                
                neighborNode.parent = current;
                neighborNode.gCost = tentativeGCost;
                neighborNode.hCost = Heuristic(neighborCoord, goal);
            }
        }
        
        return null; // No path found
    }
    
    bool IsWalkable(Vector3Int coord)
    {
        // Check dual-encoded walkability
        VoxelValue voxel = voxelData.GetVoxel(coord);
        return voxel.IsWalkable();
    }
    
    float GetCost(Vector3Int from, Vector3Int to)
    {
        // Get path cost from dual-encoded data
        VoxelValue voxel = voxelData.GetVoxel(to);
        return voxel.GetPathCost();
    }
}
```

---

## Voxel Connectivity

### 6-Connected (Face-Adjacent)
Neighbors share a face:
```
Neighbors: (x±1, y, z), (x, y±1, z), (x, y, z±1)
```

### 18-Connected (Face + Edge)
Includes edge-adjacent:
```
Face neighbors + edge neighbors
```

### 26-Connected (Face + Edge + Corner)
Includes all adjacent voxels:
```
All 26 neighbors in 3×3×3 cube
```

### Connectivity Encoding
Store which neighbors are connected:
```csharp
public class VoxelConnectivity
{
    // 6 bits for 6 face neighbors
    public bool connectedRight;
    public bool connectedLeft;
    public bool connectedUp;
    public bool connectedDown;
    public bool connectedForward;
    public bool connectedBack;
    
    public byte Encode()
    {
        byte value = 0;
        if (connectedRight) value |= 0x01;
        if (connectedLeft) value |= 0x02;
        if (connectedUp) value |= 0x04;
        if (connectedDown) value |= 0x08;
        if (connectedForward) value |= 0x10;
        if (connectedBack) value |= 0x20;
        return value;
    }
    
    public void Decode(byte value)
    {
        connectedRight = (value & 0x01) != 0;
        connectedLeft = (value & 0x02) != 0;
        connectedUp = (value & 0x04) != 0;
        connectedDown = (value & 0x08) != 0;
        connectedForward = (value & 0x10) != 0;
        connectedBack = (value & 0x20) != 0;
    }
}
```

---

## Cost Functions

### Distance-Based Cost
```csharp
float GetDistanceCost(Vector3Int from, Vector3Int to)
{
    return Vector3Int.Distance(from, to);
}
```

### Material-Based Cost
```csharp
float GetMaterialCost(Vector3Int coord)
{
    VoxelValue voxel = voxelData.GetVoxel(coord);
    MaterialType material = voxel.GetMaterial();
    
    switch (material)
    {
        case MaterialType.Air: return 1.0f;
        case MaterialType.Grass: return 1.2f;
        case MaterialType.Dirt: return 1.5f;
        case MaterialType.Stone: return 2.0f;
        default: return 10.0f; // Impassable
    }
}
```

### Density-Based Cost
```csharp
float GetDensityCost(Vector3Int coord)
{
    VoxelValue voxel = voxelData.GetVoxel(coord);
    float density = voxel.GetDensity();
    
    // Higher density = higher cost
    return 1.0f + density / 1000f;
}
```

---

## Hierarchical Voxel Pathfinding

### Multi-Resolution Voxels
Different resolutions for different pathfinding scales:

**Level 0**: Full resolution (1m voxels)
**Level 1**: Coarse resolution (10m voxels)
**Level 2**: Very coarse (100m voxels)

### Hierarchical Pathfinding
1. **High-level**: Pathfind in coarse voxels
2. **Low-level**: Refine path in fine voxels

```csharp
public class HierarchicalVoxelPathfinder
{
    private VoxelData[] lodLevels;
    
    public List<Vector3> FindPath(Vector3 start, Vector3 end)
    {
        // High-level pathfinding
        List<Vector3Int> coarsePath = FindCoarsePath(start, end, lodLevels[2]);
        
        // Refine path
        List<Vector3> finePath = new List<Vector3> { start };
        
        for (int i = 0; i < coarsePath.Count - 1; i++)
        {
            List<Vector3> segment = RefinePath(coarsePath[i], coarsePath[i + 1], lodLevels[0]);
            finePath.AddRange(segment);
        }
        
        finePath.Add(end);
        return finePath;
    }
}
```

---

## Dynamic Voxel Updates

### Updating Pathfinding Data
When voxels change, update pathfinding:

```csharp
public void UpdateVoxel(Vector3Int coord, VoxelValue newValue)
{
    voxelData.SetVoxel(coord, newValue);
    
    // Invalidate pathfinding cache
    InvalidatePathfindingCache(coord);
    
    // Update connectivity of neighbors
    UpdateNeighborConnectivity(coord);
}
```

### Incremental Pathfinding
Only recalculate affected paths:

```csharp
public void OnVoxelChanged(Vector3Int coord)
{
    // Find all active paths
    List<Path> affectedPaths = FindPathsThroughVoxel(coord);
    
    // Recalculate only affected paths
    foreach (var path in affectedPaths)
    {
        RecalculatePath(path);
    }
}
```

---

## Optimization Techniques

### Spatial Hashing
Hash voxel coordinates for fast lookup:

```csharp
int HashVoxel(Vector3Int coord)
{
    // Simple hash function
    return coord.x * 73856093 ^ 
           coord.y * 19349663 ^ 
           coord.z * 83492791;
}
```

### Chunked Pathfinding
Divide voxel space into chunks:

```csharp
public class ChunkedVoxelPathfinder
{
    private Dictionary<Vector3Int, VoxelChunk> chunks;
    private float chunkSize = 32f;
    
    public List<Vector3> FindPath(Vector3 start, Vector3 end)
    {
        Vector3Int startChunk = WorldToChunk(start);
        Vector3Int endChunk = WorldToChunk(end);
        
        // Pathfind between chunks
        List<Vector3Int> chunkPath = FindChunkPath(startChunk, endChunk);
        
        // Pathfind within chunks
        return RefineChunkPath(chunkPath, start, end);
    }
}
```

### Path Smoothing
Smooth voxel paths:

```csharp
public List<Vector3> SmoothPath(List<Vector3> path)
{
    List<Vector3> smoothed = new List<Vector3>();
    
    for (int i = 0; i < path.Count; i++)
    {
        // Try to skip intermediate points
        int skipTo = i + 1;
        while (skipTo < path.Count && HasLineOfSight(path[i], path[skipTo]))
        {
            skipTo++;
        }
        
        smoothed.Add(path[i]);
        i = skipTo - 1;
    }
    
    return smoothed;
}
```

---

## Practical Examples

### Example 1: Minecraft-Style Pathfinding
```
Voxel Encoding: Material + Walkability
Pathfinding: A* in 6-connected voxel space
Optimization: Chunked pathfinding
Result: Efficient pathfinding in voxel worlds
```

### Example 2: Destructible Terrain
```
Voxel Encoding: Density + Cost
Pathfinding: Dynamic updates when terrain changes
Optimization: Incremental pathfinding
Result: Pathfinding adapts to terrain destruction
```

### Example 3: Multi-Agent Navigation
```
Voxel Encoding: Occupancy + Connectivity
Pathfinding: Flow field or potential fields
Optimization: Precomputed fields
Result: Efficient navigation for many agents
```

---

## Exercises for Self-Learning

1. **Dual Encoding**: Implement dual-encoded voxel system
2. **Voxel A***: Implement A* pathfinding in voxel space
3. **Connectivity**: Encode and use neighbor connectivity
4. **Hierarchical**: Create multi-resolution pathfinding
5. **Dynamic Updates**: Handle voxel changes efficiently

---

## Next Steps
- **Usage**: Review hierarchical pathfinding use cases (1-usage.md)
- **Math**: Understand pathfinding mathematics (2-math.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Voxel pathfinding algorithms
- A* pathfinding in 3D
- Dual encoding techniques
- Hierarchical pathfinding
