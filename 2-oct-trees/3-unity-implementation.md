# Oct Trees: Unity Implementation

## Overview
Practical implementation of oct trees in Unity for 3D game development, including C# code examples, Unity-specific optimizations, and 3D integration patterns.

## Learning Objectives
- Implement a complete oct tree in C# for Unity
- Integrate oct trees with Unity's 3D GameObject system
- Optimize for Unity's frame-based update cycle
- Handle dynamic 3D objects and scene changes
- Debug and visualize oct tree structure in 3D

---

## Basic Oct Tree Structure

### Bounds3D Class
```csharp
[System.Serializable]
public struct Bounds3D
{
    public Vector3 min;
    public Vector3 max;
    public Vector3 center;
    public Vector3 size;
    
    public Bounds3D(Vector3 min, Vector3 max)
    {
        this.min = min;
        this.max = max;
        this.center = (min + max) / 2f;
        this.size = max - min;
    }
    
    public bool Contains(Vector3 point)
    {
        return point.x >= min.x && point.x <= max.x &&
               point.y >= min.y && point.y <= max.y &&
               point.z >= min.z && point.z <= max.z;
    }
    
    public bool Intersects(Bounds3D other)
    {
        return !(max.x < other.min.x || min.x > other.max.x ||
                 max.y < other.min.y || min.y > other.max.y ||
                 max.z < other.min.z || min.z > other.max.z);
    }
}
```

### Oct TreeNode Class
```csharp
public class OctTreeNode
{
    public Bounds3D bounds;
    public List<GameObject> objects;
    public OctTreeNode[] children;
    public int depth;
    public bool isLeaf;
    
    // Constants
    private const int MAX_OBJECTS = 10;
    private const int MAX_DEPTH = 5;
    
    public OctTreeNode(Bounds3D bounds, int depth = 0)
    {
        this.bounds = bounds;
        this.depth = depth;
        this.objects = new List<GameObject>();
        this.children = new OctTreeNode[8];
        this.isLeaf = true;
    }
}
```

### Oct Tree Class
```csharp
public class OctTree
{
    private OctTreeNode root;
    private int maxObjects;
    private int maxDepth;
    
    public OctTree(Bounds3D bounds, int maxObjects = 10, int maxDepth = 5)
    {
        this.maxObjects = maxObjects;
        this.maxDepth = maxDepth;
        this.root = new OctTreeNode(bounds, 0);
    }
}
```

---

## Subdivision Logic

### Subdivide Method
```csharp
private void Subdivide(OctTreeNode node)
{
    if (node.depth >= maxDepth || node.objects.Count <= maxObjects)
        return;
    
    Vector3 center = node.bounds.center;
    Vector3 halfSize = node.bounds.size / 2f;
    Vector3 min = node.bounds.min;
    
    // Create eight child octants
    // Back-Bottom-Left (0)
    node.children[0] = new OctTreeNode(
        new Bounds3D(
            min,
            center
        ), 
        node.depth + 1);
    
    // Back-Bottom-Right (1)
    node.children[1] = new OctTreeNode(
        new Bounds3D(
            new Vector3(center.x, min.y, min.z),
            new Vector3(node.bounds.max.x, center.y, center.z)
        ), 
        node.depth + 1);
    
    // Back-Top-Left (2)
    node.children[2] = new OctTreeNode(
        new Bounds3D(
            new Vector3(min.x, center.y, min.z),
            new Vector3(center.x, node.bounds.max.y, center.z)
        ), 
        node.depth + 1);
    
    // Back-Top-Right (3)
    node.children[3] = new OctTreeNode(
        new Bounds3D(
            new Vector3(center.x, center.y, min.z),
            new Vector3(node.bounds.max.x, node.bounds.max.y, center.z)
        ), 
        node.depth + 1);
    
    // Front-Bottom-Left (4)
    node.children[4] = new OctTreeNode(
        new Bounds3D(
            new Vector3(min.x, min.y, center.z),
            new Vector3(center.x, center.y, node.bounds.max.z)
        ), 
        node.depth + 1);
    
    // Front-Bottom-Right (5)
    node.children[5] = new OctTreeNode(
        new Bounds3D(
            new Vector3(center.x, min.y, center.z),
            new Vector3(node.bounds.max.x, center.y, node.bounds.max.z)
        ), 
        node.depth + 1);
    
    // Front-Top-Left (6)
    node.children[6] = new OctTreeNode(
        new Bounds3D(
            new Vector3(min.x, center.y, center.z),
            new Vector3(center.x, node.bounds.max.y, node.bounds.max.z)
        ), 
        node.depth + 1);
    
    // Front-Top-Right (7)
    node.children[7] = new OctTreeNode(
        new Bounds3D(
            center,
            node.bounds.max
        ), 
        node.depth + 1);
    
    node.isLeaf = false;
    
    // Redistribute objects to children
    RedistributeObjects(node);
}
```

### Get Octant Helper
```csharp
private int GetOctant(OctTreeNode node, Vector3 point)
{
    Vector3 center = node.bounds.center;
    int octant = 0;
    
    if (point.x >= center.x) octant |= 1; // X bit
    if (point.y >= center.y) octant |= 2; // Y bit
    if (point.z >= center.z) octant |= 4; // Z bit
    
    return octant;
}
```

### Redistribute Objects
```csharp
private void RedistributeObjects(OctTreeNode node)
{
    List<GameObject> objectsToRedistribute = new List<GameObject>(node.objects);
    node.objects.Clear();
    
    foreach (GameObject obj in objectsToRedistribute)
    {
        Bounds objBounds = GetObjectBounds(obj);
        int octant = GetOctantForBounds(node, objBounds);
        
        if (octant != -1)
        {
            node.children[octant].objects.Add(obj);
        }
        else
        {
            // Object spans multiple octants, keep in parent
            node.objects.Add(obj);
        }
    }
    
    // Recursively subdivide children if needed
    for (int i = 0; i < 8; i++)
    {
        if (node.children[i].objects.Count > maxObjects)
        {
            Subdivide(node.children[i]);
        }
    }
}

private Bounds GetObjectBounds(GameObject obj)
{
    Renderer renderer = obj.GetComponent<Renderer>();
    if (renderer != null)
    {
        return renderer.bounds;
    }
    Collider collider = obj.GetComponent<Collider>();
    if (collider != null)
    {
        return collider.bounds;
    }
    // Fallback to point
    return new Bounds(obj.transform.position, Vector3.zero);
}

private int GetOctantForBounds(OctTreeNode node, Bounds objBounds)
{
    // Check if object fits entirely in one octant
    Vector3 objCenter = objBounds.center;
    int octant = GetOctant(node, objCenter);
    
    // Verify object is fully contained
    Bounds3D octantBounds = node.children[octant].bounds;
    if (octantBounds.Contains(objBounds.min) && octantBounds.Contains(objBounds.max))
    {
        return octant;
    }
    
    // Object spans multiple octants
    return -1;
}
```

---

## Insertion

### Insert Method
```csharp
public void Insert(GameObject obj)
{
    Bounds objBounds = GetObjectBounds(obj);
    if (!root.bounds.Intersects(new Bounds3D(objBounds.min, objBounds.max)))
    {
        Debug.LogWarning("Object outside oct tree bounds!");
        return;
    }
    
    InsertRecursive(root, obj);
}

private void InsertRecursive(OctTreeNode node, GameObject obj)
{
    if (node.isLeaf)
    {
        node.objects.Add(obj);
        
        // Subdivide if necessary
        if (node.objects.Count > maxObjects && node.depth < maxDepth)
        {
            Subdivide(node);
        }
    }
    else
    {
        Bounds objBounds = GetObjectBounds(obj);
        int octant = GetOctantForBounds(node, objBounds);
        
        if (octant != -1)
        {
            InsertRecursive(node.children[octant], obj);
        }
        else
        {
            // Object spans octants, add to current node
            node.objects.Add(obj);
        }
    }
}
```

---

## Query Operations

### Point Query
```csharp
public List<GameObject> Query(Vector3 point)
{
    List<GameObject> results = new List<GameObject>();
    QueryRecursive(root, point, results);
    return results;
}

private void QueryRecursive(OctTreeNode node, Vector3 point, List<GameObject> results)
{
    if (!node.bounds.Contains(point))
        return;
    
    // Add objects in this node
    results.AddRange(node.objects);
    
    // Query children if not leaf
    if (!node.isLeaf)
    {
        int octant = GetOctant(node, point);
        if (octant != -1)
        {
            QueryRecursive(node.children[octant], point, results);
        }
        else
        {
            // Point on boundary, check all children
            for (int i = 0; i < 8; i++)
            {
                QueryRecursive(node.children[i], point, results);
            }
        }
    }
}
```

### 3D Range Query
```csharp
public List<GameObject> QueryRange(Bounds3D range)
{
    List<GameObject> results = new List<GameObject>();
    QueryRangeRecursive(root, range, results);
    return results;
}

private void QueryRangeRecursive(OctTreeNode node, Bounds3D range, List<GameObject> results)
{
    if (!node.bounds.Intersects(range))
        return;
    
    // Check objects in this node
    foreach (GameObject obj in node.objects)
    {
        Bounds objBounds = GetObjectBounds(obj);
        Bounds3D objBounds3D = new Bounds3D(objBounds.min, objBounds.max);
        if (range.Intersects(objBounds3D))
        {
            results.Add(obj);
        }
    }
    
    // Query children
    if (!node.isLeaf)
    {
        for (int i = 0; i < 8; i++)
        {
            QueryRangeRecursive(node.children[i], range, results);
        }
    }
}
```

### Sphere Query
```csharp
public List<GameObject> QuerySphere(Vector3 center, float radius)
{
    Vector3 min = center - Vector3.one * radius;
    Vector3 max = center + Vector3.one * radius;
    Bounds3D bounds = new Bounds3D(min, max);
    
    List<GameObject> candidates = QueryRange(bounds);
    List<GameObject> results = new List<GameObject>();
    
    float radiusSquared = radius * radius;
    foreach (GameObject obj in candidates)
    {
        float distSquared = (obj.transform.position - center).sqrMagnitude;
        if (distSquared <= radiusSquared)
        {
            results.Add(obj);
        }
    }
    
    return results;
}
```

---

## Unity Integration

### MonoBehaviour Wrapper
```csharp
public class OctTreeManager : MonoBehaviour
{
    public Vector3 worldCenter = Vector3.zero;
    public Vector3 worldSize = new Vector3(200, 200, 200);
    public int maxObjectsPerNode = 10;
    public int maxDepth = 5;
    public bool showDebugGizmos = true;
    
    private OctTree octTree;
    private List<GameObject> trackedObjects = new List<GameObject>();
    
    void Start()
    {
        Bounds3D worldBounds = new Bounds3D(
            worldCenter - worldSize / 2f,
            worldCenter + worldSize / 2f
        );
        octTree = new OctTree(worldBounds, maxObjectsPerNode, maxDepth);
        
        // Register all objects with tag "Entity"
        GameObject[] entities = GameObject.FindGameObjectsWithTag("Entity");
        foreach (GameObject entity in entities)
        {
            RegisterObject(entity);
        }
    }
    
    void Update()
    {
        // Rebuild tree if objects moved (or use incremental updates)
        if (ShouldRebuild())
        {
            RebuildTree();
        }
    }
    
    public void RegisterObject(GameObject obj)
    {
        if (!trackedObjects.Contains(obj))
        {
            trackedObjects.Add(obj);
            octTree.Insert(obj);
        }
    }
    
    public void UnregisterObject(GameObject obj)
    {
        trackedObjects.Remove(obj);
        RebuildTree(); // Simple approach - could optimize with removal
    }
    
    private bool ShouldRebuild()
    {
        // Check if any objects moved significantly
        return false; // Implement based on your needs
    }
    
    private void RebuildTree()
    {
        Bounds3D worldBounds = new Bounds3D(
            worldCenter - worldSize / 2f,
            worldCenter + worldSize / 2f
        );
        octTree = new OctTree(worldBounds, maxObjectsPerNode, maxDepth);
        foreach (GameObject obj in trackedObjects)
        {
            if (obj != null)
            {
                octTree.Insert(obj);
            }
        }
    }
    
    public List<GameObject> QueryNearby(Vector3 position, float radius)
    {
        return octTree.QuerySphere(position, radius);
    }
}
```

---

## Debug Visualization

### Gizmo Drawing
```csharp
void OnDrawGizmos()
{
    if (!showDebugGizmos || octTree == null)
        return;
    
    DrawOctTreeGizmos(octTree.root);
}

private void DrawOctTreeGizmos(OctTreeNode node)
{
    // Draw node bounds
    Gizmos.color = node.isLeaf ? Color.green : Color.yellow;
    Gizmos.DrawWireCube(
        node.bounds.center,
        node.bounds.size
    );
    
    // Draw objects in node
    Gizmos.color = Color.red;
    foreach (GameObject obj in node.objects)
    {
        if (obj != null)
        {
            Gizmos.DrawSphere(obj.transform.position, 0.5f);
        }
    }
    
    // Recursively draw children
    if (!node.isLeaf)
    {
        for (int i = 0; i < 8; i++)
        {
            if (node.children[i] != null)
            {
                DrawOctTreeGizmos(node.children[i]);
            }
        }
    }
}
```

---

## Common Patterns

### 3D Collision Detection
```csharp
void CheckCollisions()
{
    foreach (GameObject obj in trackedObjects)
    {
        if (obj == null) continue;
        
        float checkRadius = GetCollisionRadius(obj);
        List<GameObject> nearby = QueryNearby(
            obj.transform.position, 
            checkRadius
        );
        
        foreach (GameObject other in nearby)
        {
            if (obj != other && IsColliding3D(obj, other))
            {
                HandleCollision(obj, other);
            }
        }
    }
}
```

### Frustum Culling
```csharp
List<GameObject> GetVisibleObjects(Camera camera)
{
    Plane[] planes = GeometryUtility.CalculateFrustumPlanes(camera);
    Bounds cameraBounds = GetCameraBounds(camera);
    Bounds3D queryBounds = new Bounds3D(
        cameraBounds.min,
        cameraBounds.max
    );
    
    List<GameObject> candidates = octTree.QueryRange(queryBounds);
    List<GameObject> visible = new List<GameObject>();
    
    foreach (GameObject obj in candidates)
    {
        Bounds objBounds = GetObjectBounds(obj);
        if (GeometryUtility.TestPlanesAABB(planes, objBounds))
        {
            visible.Add(obj);
        }
    }
    
    return visible;
}
```

---

## Exercises for Self-Learning

1. **Implement Basic Oct Tree**: Create the core structure and test with 100 3D objects
2. **Add 3D Visualization**: Implement Gizmo drawing to see tree structure in 3D
3. **Optimize Parameters**: Tune maxObjects and maxDepth for your use case
4. **Add Removal**: Implement efficient object removal without full rebuild
5. **Performance Test**: Compare oct tree queries vs. brute force for 1000+ 3D objects

---

## Next Steps
- **Math**: Review mathematical differences from quad trees (1-math-difference-to-quads.md)
- **Usage**: Review use cases (2-usage.md)

---

## References
- Unity Scripting API (3D)
- C# Collections documentation
- Unity Gizmos documentation (3D)
