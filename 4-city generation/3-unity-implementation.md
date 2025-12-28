# City Generation: Unity Implementation

## Overview
Practical implementation of procedural city generation in Unity, including road networks, building placement, zoning, and integration with Unity systems.

## Learning Objectives
- Generate road networks procedurally
- Place buildings algorithmically
- Implement zoning systems
- Create city visualization
- Optimize for performance

---

## Basic City Generator Structure

### City Generator Class
```csharp
public class CityGenerator : MonoBehaviour
{
    [Header("City Parameters")]
    public int cityWidth = 1000;
    public int cityHeight = 1000;
    public float roadSpacing = 50f;
    
    [Header("Road Settings")]
    public float mainRoadWidth = 10f;
    public float sideRoadWidth = 6f;
    
    [Header("Building Settings")]
    public float minBuildingSize = 10f;
    public float maxBuildingSize = 30f;
    public float buildingHeightMin = 5f;
    public float buildingHeightMax = 50f;
    
    private List<Road> roads = new List<Road>();
    private List<Building> buildings = new List<Building>();
    private Dictionary<Vector2Int, ZoneType> zones = new Dictionary<Vector2Int, ZoneType>();
    
    void Start()
    {
        GenerateCity();
    }
    
    void GenerateCity()
    {
        GenerateRoadNetwork();
        GenerateZones();
        GenerateBuildings();
    }
}
```

---

## Road Network Generation

### Road Class
```csharp
[System.Serializable]
public class Road
{
    public Vector3 start;
    public Vector3 end;
    public float width;
    public RoadType type;
    
    public Road(Vector3 start, Vector3 end, float width, RoadType type)
    {
        this.start = start;
        this.end = end;
        this.width = width;
        this.type = type;
    }
    
    public float Length => Vector3.Distance(start, end);
}

public enum RoadType
{
    Highway,
    MainRoad,
    SideStreet,
    Alley
}
```

### Grid-Based Road Generation
```csharp
void GenerateRoadNetwork()
{
    roads.Clear();
    
    // Generate main grid roads
    for (int x = 0; x <= cityWidth; x += (int)roadSpacing)
    {
        // Vertical roads
        roads.Add(new Road(
            new Vector3(x, 0, 0),
            new Vector3(x, 0, cityHeight),
            mainRoadWidth,
            RoadType.MainRoad
        ));
    }
    
    for (int z = 0; z <= cityHeight; z += (int)roadSpacing)
    {
        // Horizontal roads
        roads.Add(new Road(
            new Vector3(0, 0, z),
            new Vector3(cityWidth, 0, z),
            mainRoadWidth,
            RoadType.MainRoad
        ));
    }
    
    // Generate side streets (smaller grid)
    GenerateSideStreets();
}

void GenerateSideStreets()
{
    int sideStreetSpacing = (int)roadSpacing / 2;
    
    for (int x = sideStreetSpacing; x < cityWidth; x += (int)roadSpacing)
    {
        for (int z = 0; z <= cityHeight; z += (int)roadSpacing)
        {
            roads.Add(new Road(
                new Vector3(x, 0, z),
                new Vector3(x, 0, Mathf.Min(z + sideStreetSpacing, cityHeight)),
                sideRoadWidth,
                RoadType.SideStreet
            ));
        }
    }
}
```

### L-System Road Generation
```csharp
public class LSystemRoadGenerator
{
    private string axiom = "F";
    private Dictionary<char, string> rules = new Dictionary<char, string>();
    private List<Road> roads = new List<Road>();
    
    public LSystemRoadGenerator()
    {
        rules['F'] = "F[+F]F[-F]F";
    }
    
    public List<Road> Generate(int iterations, float length, float angle)
    {
        string result = axiom;
        
        for (int i = 0; i < iterations; i++)
        {
            result = ApplyRules(result);
        }
        
        Interpret(result, length, angle);
        return roads;
    }
    
    string ApplyRules(string input)
    {
        string output = "";
        foreach (char c in input)
        {
            if (rules.ContainsKey(c))
                output += rules[c];
            else
                output += c;
        }
        return output;
    }
    
    void Interpret(string commands, float length, float angle)
    {
        Stack<RoadState> stateStack = new Stack<RoadState>();
        RoadState current = new RoadState(Vector3.zero, Vector3.forward, length);
        
        foreach (char c in commands)
        {
            switch (c)
            {
                case 'F':
                    Vector3 end = current.position + current.direction * current.length;
                    roads.Add(new Road(current.position, end, 5f, RoadType.MainRoad));
                    current.position = end;
                    break;
                case '+':
                    current.direction = Quaternion.Euler(0, angle, 0) * current.direction;
                    break;
                case '-':
                    current.direction = Quaternion.Euler(0, -angle, 0) * current.direction;
                    break;
                case '[':
                    stateStack.Push(new RoadState(current));
                    current.length *= 0.7f; // Reduce length
                    break;
                case ']':
                    current = stateStack.Pop();
                    break;
            }
        }
    }
}

class RoadState
{
    public Vector3 position;
    public Vector3 direction;
    public float length;
    
    public RoadState(Vector3 pos, Vector3 dir, float len)
    {
        position = pos;
        direction = dir;
        length = len;
    }
    
    public RoadState(RoadState other)
    {
        position = other.position;
        direction = other.direction;
        length = other.length;
    }
}
```

---

## Zone Generation

### Zone Types
```csharp
public enum ZoneType
{
    Residential,
    Commercial,
    Industrial,
    Park,
    Mixed
}
```

### Voronoi-Based Zoning
```csharp
void GenerateZones()
{
    zones.Clear();
    
    // Generate zone seeds
    List<Vector2> seeds = new List<Vector2>();
    int numZones = 20;
    
    for (int i = 0; i < numZones; i++)
    {
        seeds.Add(new Vector2(
            Random.Range(0, cityWidth),
            Random.Range(0, cityHeight)
        ));
    }
    
    // Assign zone types to seeds
    ZoneType[] zoneTypes = new ZoneType[seeds.Count];
    for (int i = 0; i < seeds.Count; i++)
    {
        zoneTypes[i] = (ZoneType)Random.Range(0, System.Enum.GetValues(typeof(ZoneType)).Length);
    }
    
    // Create Voronoi diagram (simplified grid-based)
    int gridSize = 10;
    for (int x = 0; x < cityWidth; x += gridSize)
    {
        for (int z = 0; z < cityHeight; z += gridSize)
        {
            Vector2 point = new Vector2(x, z);
            int closestSeed = FindClosestSeed(point, seeds);
            zones[new Vector2Int(x, z)] = zoneTypes[closestSeed];
        }
    }
}

int FindClosestSeed(Vector2 point, List<Vector2> seeds)
{
    int closest = 0;
    float minDist = float.MaxValue;
    
    for (int i = 0; i < seeds.Count; i++)
    {
        float dist = Vector2.Distance(point, seeds[i]);
        if (dist < minDist)
        {
            minDist = dist;
            closest = i;
        }
    }
    
    return closest;
}
```

---

## Building Generation

### Building Class
```csharp
[System.Serializable]
public class Building
{
    public Vector3 position;
    public Vector3 size;
    public ZoneType zone;
    public GameObject mesh;
    
    public Building(Vector3 pos, Vector3 sz, ZoneType z)
    {
        position = pos;
        size = sz;
        zone = z;
    }
}
```

### Poisson Disk Sampling for Buildings
```csharp
void GenerateBuildings()
{
    buildings.Clear();
    
    // Divide city into blocks (areas between roads)
    List<Rect> blocks = GetCityBlocks();
    
    foreach (Rect block in blocks)
    {
        ZoneType zone = GetZoneAt(block.center);
        GenerateBuildingsInBlock(block, zone);
    }
}

List<Rect> GetCityBlocks()
{
    List<Rect> blocks = new List<Rect>();
    
    // Simplified: assume grid roads create rectangular blocks
    for (int x = 0; x < cityWidth; x += (int)roadSpacing)
    {
        for (int z = 0; z < cityHeight; z += (int)roadSpacing)
        {
            float blockSize = roadSpacing - mainRoadWidth;
            if (blockSize > 0)
            {
                blocks.Add(new Rect(x, z, blockSize, blockSize));
            }
        }
    }
    
    return blocks;
}

void GenerateBuildingsInBlock(Rect block, ZoneType zone)
{
    List<Vector2> candidates = new List<Vector2>();
    int maxAttempts = 30;
    float minDistance = minBuildingSize * 1.5f;
    
    for (int i = 0; i < maxAttempts; i++)
    {
        Vector2 candidate = new Vector2(
            Random.Range(block.xMin, block.xMax),
            Random.Range(block.yMin, block.yMax)
        );
        
        bool valid = true;
        foreach (Building existing in buildings)
        {
            float dist = Vector2.Distance(candidate, new Vector2(existing.position.x, existing.position.z));
            if (dist < minDistance)
            {
                valid = false;
                break;
            }
        }
        
        if (valid)
        {
            Vector3 size = GetBuildingSize(zone);
            float height = GetBuildingHeight(zone);
            buildings.Add(new Building(
                new Vector3(candidate.x, 0, candidate.y),
                new Vector3(size.x, height, size.z),
                zone
            ));
        }
    }
}

Vector3 GetBuildingSize(ZoneType zone)
{
    float size = Random.Range(minBuildingSize, maxBuildingSize);
    
    // Adjust based on zone
    switch (zone)
    {
        case ZoneType.Residential:
            size *= 0.8f; // Smaller residential
            break;
        case ZoneType.Commercial:
            size *= 1.2f; // Larger commercial
            break;
        case ZoneType.Industrial:
            size *= 1.5f; // Largest industrial
            break;
    }
    
    return new Vector3(size, 0, size);
}

float GetBuildingHeight(ZoneType zone)
{
    float baseHeight = Random.Range(buildingHeightMin, buildingHeightMax);
    
    // Adjust based on zone
    switch (zone)
    {
        case ZoneType.Residential:
            baseHeight *= 0.6f; // Shorter residential
            break;
        case ZoneType.Commercial:
            baseHeight *= 1.3f; // Taller commercial
            break;
        case ZoneType.Industrial:
            baseHeight *= 0.8f; // Medium industrial
            break;
    }
    
    return baseHeight;
}
```

---

## Visualization

### Road Visualization
```csharp
void OnDrawGizmos()
{
    // Draw roads
    Gizmos.color = Color.gray;
    foreach (Road road in roads)
    {
        Vector3 center = (road.start + road.end) / 2f;
        Vector3 direction = (road.end - road.start).normalized;
        float length = road.Length;
        
        Gizmos.DrawCube(center, new Vector3(road.width, 0.1f, length));
    }
    
    // Draw buildings
    foreach (Building building in buildings)
    {
        Gizmos.color = GetZoneColor(building.zone);
        Gizmos.DrawCube(
            building.position + Vector3.up * building.size.y / 2f,
            building.size
        );
    }
}

Color GetZoneColor(ZoneType zone)
{
    switch (zone)
    {
        case ZoneType.Residential: return Color.green;
        case ZoneType.Commercial: return Color.blue;
        case ZoneType.Industrial: return Color.red;
        case ZoneType.Park: return Color.yellow;
        default: return Color.white;
    }
}
```

---

## Mesh Generation

### Building Mesh Generation
```csharp
Mesh CreateBuildingMesh(Building building)
{
    Mesh mesh = new Mesh();
    
    Vector3 size = building.size;
    Vector3[] vertices = new Vector3[8];
    
    // Bottom face
    vertices[0] = new Vector3(-size.x/2, 0, -size.z/2);
    vertices[1] = new Vector3(size.x/2, 0, -size.z/2);
    vertices[2] = new Vector3(size.x/2, 0, size.z/2);
    vertices[3] = new Vector3(-size.x/2, 0, size.z/2);
    
    // Top face
    vertices[4] = new Vector3(-size.x/2, size.y, -size.z/2);
    vertices[5] = new Vector3(size.x/2, size.y, -size.z/2);
    vertices[6] = new Vector3(size.x/2, size.y, size.z/2);
    vertices[7] = new Vector3(-size.x/2, size.y, size.z/2);
    
    int[] triangles = new int[]
    {
        // Bottom
        0, 2, 1, 0, 3, 2,
        // Top
        4, 5, 6, 4, 6, 7,
        // Sides
        0, 1, 5, 0, 5, 4,
        1, 2, 6, 1, 6, 5,
        2, 3, 7, 2, 7, 6,
        3, 0, 4, 3, 4, 7
    };
    
    mesh.vertices = vertices;
    mesh.triangles = triangles;
    mesh.RecalculateNormals();
    
    return mesh;
}
```

---

## Exercises for Self-Learning

1. **Basic Generator**: Implement simple grid-based city generator
2. **Road Network**: Add L-system road generation
3. **Zoning**: Implement Voronoi-based zoning
4. **Buildings**: Add Poisson disk sampling for buildings
5. **Visualization**: Create 3D visualization of generated city

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math-proc-gen-quad-oct-tree-design.md)
- **Physics**: Learn city physics (4-city-physics.md)
- **Vehicles**: Learn vehicle systems (5-vehicles-in-a-city.md)

---

## References
- Unity Mesh API
- Procedural generation algorithms
- L-systems implementation
- Poisson disk sampling
