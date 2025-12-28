# Quad Trees: Unity Implementation

## Overview
Practical implementation of quad trees in Unity for game development, including C# code examples, Unity-specific optimizations, and integration patterns.

## Learning Objectives
- Implement a complete quad tree in C#
- Integrate quad trees with Unity's GameObject system
- Optimize for Unity's frame-based update cycle
- Handle dynamic objects and scene changes
- Debug and visualize quad tree structure

---

## Basic Quad Tree Structure

### Node Class
```csharp
public class QuadTreeNode
{
    public Rect bounds;
    public List<GameObject> objects;
    public QuadTreeNode[] children;
    public int depth;
    public bool isLeaf;
    
    // Constants
    private const int MAX_OBJECTS = 10;
    private const int MAX_DEPTH = 5;
    
    public QuadTreeNode(Rect bounds, int depth = 0)
    {
        this.bounds = bounds;
        this.depth = depth;
        this.objects = new List<GameObject>();
        this.children = new QuadTreeNode[4];
        this.isLeaf = true;
    }
}
```

### Quad Tree Class
```csharp
public class QuadTree
{
    private QuadTreeNode root;
    private int maxObjects;
    private int maxDepth;
    
    public QuadTree(Rect bounds, int maxObjects = 10, int maxDepth = 5)
    {
        this.maxObjects = maxObjects;
        this.maxDepth = maxDepth;
        this.root = new QuadTreeNode(bounds, 0);
    }
}
```

---

## Subdivision Logic

### Subdivide Method
```csharp
private void Subdivide(QuadTreeNode node)
{
    if (node.depth >= maxDepth || node.objects.Count <= maxObjects)
        return;
    
    float halfWidth = node.bounds.width / 2f;
    float halfHeight = node.bounds.height / 2f;
    float x = node.bounds.x;
    float y = node.bounds.y;
    
    // Create four child quadrants
    node.children[0] = new QuadTreeNode(
        new Rect(x, y, halfWidth, halfHeight), 
        node.depth + 1); // Bottom-Left
    
    node.children[1] = new QuadTreeNode(
        new Rect(x + halfWidth, y, halfWidth, halfHeight), 
        node.depth + 1); // Bottom-Right
    
    node.children[2] = new QuadTreeNode(
        new Rect(x, y + halfHeight, halfWidth, halfHeight), 
        node.depth + 1); // Top-Left
    
    node.children[3] = new QuadTreeNode(
        new Rect(x + halfWidth, y + halfHeight, halfWidth, halfHeight), 
        node.depth + 1); // Top-Right
    
    node.isLeaf = false;
    
    // Redistribute objects to children
    RedistributeObjects(node);
}
```

### Redistribute Objects
```csharp
private void RedistributeObjects(QuadTreeNode node)
{
    List<GameObject> objectsToRedistribute = new List<GameObject>(node.objects);
    node.objects.Clear();
    
    foreach (GameObject obj in objectsToRedistribute)
    {
        int quadrant = GetQuadrant(node, obj.transform.position);
        if (quadrant != -1)
        {
            node.children[quadrant].objects.Add(obj);
        }
        else
        {
            // Object spans multiple quadrants, keep in parent
            node.objects.Add(obj);
        }
    }
    
    // Recursively subdivide children if needed
    for (int i = 0; i < 4; i++)
    {
        if (node.children[i].objects.Count > maxObjects)
        {
            Subdivide(node.children[i]);
        }
    }
}
```

### Get Quadrant Helper
```csharp
private int GetQuadrant(QuadTreeNode node, Vector2 point)
{
    float centerX = node.bounds.x + node.bounds.width / 2f;
    float centerY = node.bounds.y + node.bounds.height / 2f;
    
    bool left = point.x < centerX;
    bool bottom = point.y < centerY;
    
    if (left && bottom) return 0;      // Bottom-Left
    if (!left && bottom) return 1;     // Bottom-Right
    if (left && !bottom) return 2;     // Top-Left
    return 3;                          // Top-Right
}
```

---

## Insertion

### Insert Method
```csharp
public void Insert(GameObject obj)
{
    if (!root.bounds.Contains(obj.transform.position))
    {
        Debug.LogWarning("Object outside quad tree bounds!");
        return;
    }
    
    InsertRecursive(root, obj);
}

private void InsertRecursive(QuadTreeNode node, GameObject obj)
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
        int quadrant = GetQuadrant(node, obj.transform.position);
        if (quadrant != -1)
        {
            InsertRecursive(node.children[quadrant], obj);
        }
        else
        {
            // Object spans quadrants, add to current node
            node.objects.Add(obj);
        }
    }
}
```

---

## Query Operations

### Point Query
```csharp
public List<GameObject> Query(Vector2 point)
{
    List<GameObject> results = new List<GameObject>();
    QueryRecursive(root, point, results);
    return results;
}

private void QueryRecursive(QuadTreeNode node, Vector2 point, List<GameObject> results)
{
    if (!node.bounds.Contains(point))
        return;
    
    // Add objects in this node
    results.AddRange(node.objects);
    
    // Query children if not leaf
    if (!node.isLeaf)
    {
        int quadrant = GetQuadrant(node, point);
        if (quadrant != -1)
        {
            QueryRecursive(node.children[quadrant], point, results);
        }
        else
        {
            // Point on boundary, check all children
            for (int i = 0; i < 4; i++)
            {
                QueryRecursive(node.children[i], point, results);
            }
        }
    }
}
```

### Range Query
```csharp
public List<GameObject> QueryRange(Rect range)
{
    List<GameObject> results = new List<GameObject>();
    QueryRangeRecursive(root, range, results);
    return results;
}

private void QueryRangeRecursive(QuadTreeNode node, Rect range, List<GameObject> results)
{
    if (!node.bounds.Overlaps(range))
        return;
    
    // Check objects in this node
    foreach (GameObject obj in node.objects)
    {
        if (range.Contains(obj.transform.position))
        {
            results.Add(obj);
        }
    }
    
    // Query children
    if (!node.isLeaf)
    {
        for (int i = 0; i < 4; i++)
        {
            QueryRangeRecursive(node.children[i], range, results);
        }
    }
}
```

---

## Unity Integration

### MonoBehaviour Wrapper
```csharp
public class QuadTreeManager : MonoBehaviour
{
    public Rect worldBounds = new Rect(-100, -100, 200, 200);
    public int maxObjectsPerNode = 10;
    public int maxDepth = 5;
    public bool showDebugGizmos = true;
    
    private QuadTree quadTree;
    private List<GameObject> trackedObjects = new List<GameObject>();
    
    void Start()
    {
        quadTree = new QuadTree(worldBounds, maxObjectsPerNode, maxDepth);
        
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
            quadTree.Insert(obj);
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
        // Could use dirty flags or position tracking
        return false; // Implement based on your needs
    }
    
    private void RebuildTree()
    {
        quadTree = new QuadTree(worldBounds, maxObjectsPerNode, maxDepth);
        foreach (GameObject obj in trackedObjects)
        {
            if (obj != null)
            {
                quadTree.Insert(obj);
            }
        }
    }
    
    public List<GameObject> QueryNearby(Vector2 position, float radius)
    {
        Rect queryRange = new Rect(
            position.x - radius, 
            position.y - radius, 
            radius * 2, 
            radius * 2
        );
        return quadTree.QueryRange(queryRange);
    }
}
```

---

## Dynamic Object Handling

### Object Tracking Component
```csharp
public class QuadTreeObject : MonoBehaviour
{
    private QuadTreeManager manager;
    private Vector3 lastPosition;
    private bool isRegistered = false;
    
    void Start()
    {
        manager = FindObjectOfType<QuadTreeManager>();
        if (manager != null)
        {
            manager.RegisterObject(gameObject);
            isRegistered = true;
            lastPosition = transform.position;
        }
    }
    
    void Update()
    {
        if (isRegistered && manager != null)
        {
            // Check if moved significantly
            if (Vector3.Distance(transform.position, lastPosition) > 0.1f)
            {
                // Trigger tree rebuild or use incremental update
                lastPosition = transform.position;
            }
        }
    }
    
    void OnDestroy()
    {
        if (manager != null && isRegistered)
        {
            manager.UnregisterObject(gameObject);
        }
    }
}
```

---

## Debug Visualization

### Gizmo Drawing
```csharp
void OnDrawGizmos()
{
    if (!showDebugGizmos || quadTree == null)
        return;
    
    DrawQuadTreeGizmos(quadTree.root);
}

private void DrawQuadTreeGizmos(QuadTreeNode node)
{
    // Draw node bounds
    Gizmos.color = node.isLeaf ? Color.green : Color.yellow;
    Gizmos.DrawWireCube(
        new Vector3(node.bounds.center.x, node.bounds.center.y, 0),
        new Vector3(node.bounds.width, node.bounds.height, 0)
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
        for (int i = 0; i < 4; i++)
        {
            if (node.children[i] != null)
            {
                DrawQuadTreeGizmos(node.children[i]);
            }
        }
    }
}
```

---

## Performance Optimizations

### Object Pooling
```csharp
// Pool node objects to reduce GC allocations
private Queue<QuadTreeNode> nodePool = new Queue<QuadTreeNode>();

private QuadTreeNode GetPooledNode(Rect bounds, int depth)
{
    QuadTreeNode node;
    if (nodePool.Count > 0)
    {
        node = nodePool.Dequeue();
        node.bounds = bounds;
        node.depth = depth;
        node.objects.Clear();
        node.isLeaf = true;
    }
    else
    {
        node = new QuadTreeNode(bounds, depth);
    }
    return node;
}
```

### Incremental Updates
```csharp
// Instead of full rebuild, update only changed objects
public void UpdateObject(GameObject obj)
{
    RemoveObject(obj);
    Insert(obj);
}

private void RemoveObject(GameObject obj)
{
    RemoveRecursive(root, obj);
}

private bool RemoveRecursive(QuadTreeNode node, GameObject obj)
{
    if (node.objects.Remove(obj))
        return true;
    
    if (!node.isLeaf)
    {
        for (int i = 0; i < 4; i++)
        {
            if (RemoveRecursive(node.children[i], obj))
                return true;
        }
    }
    
    return false;
}
```

---

## Common Patterns

### Collision Detection Pattern
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
            if (obj != other && IsColliding(obj, other))
            {
                HandleCollision(obj, other);
            }
        }
    }
}
```

### Frustum Culling Pattern
```csharp
List<GameObject> GetVisibleObjects(Camera camera)
{
    Plane[] planes = GeometryUtility.CalculateFrustumPlanes(camera);
    Bounds cameraBounds = GetCameraBounds(camera);
    Rect queryRect = new Rect(
        cameraBounds.min.x, 
        cameraBounds.min.y,
        cameraBounds.size.x, 
        cameraBounds.size.y
    );
    
    List<GameObject> candidates = quadTree.QueryRange(queryRect);
    List<GameObject> visible = new List<GameObject>();
    
    foreach (GameObject obj in candidates)
    {
        if (GeometryUtility.TestPlanesAABB(planes, GetBounds(obj)))
        {
            visible.Add(obj);
        }
    }
    
    return visible;
}
```

---

## Exercises for Self-Learning

1. **Implement Basic Quad Tree**: Create the core structure and test with 100 objects
2. **Add Visualization**: Implement Gizmo drawing to see tree structure
3. **Optimize for Your Use Case**: Tune maxObjects and maxDepth parameters
4. **Add Removal**: Implement efficient object removal without full rebuild
5. **Performance Test**: Compare quad tree queries vs. brute force for 1000+ objects

---

## Troubleshooting

### Common Issues
1. **Objects not found**: Check bounds initialization
2. **Poor performance**: Adjust maxObjects and maxDepth
3. **Memory issues**: Implement object pooling
4. **Stale data**: Ensure tree rebuilds when objects move

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand the mathematics (2-math.md)

---

## References
- Unity Scripting API
- C# Collections documentation
- Unity Gizmos documentation
