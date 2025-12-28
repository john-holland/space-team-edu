# Heightmaps: Unity Implementation

## Overview
Practical implementation of heightmaps in Unity, including terrain generation, height lookups, normal calculations, and integration with Unity's Terrain system.

## Learning Objectives
- Load and process heightmap data in Unity
- Generate terrain meshes from heightmaps
- Implement height queries and interpolation
- Calculate normals, slopes, and terrain properties
- Integrate with Unity Terrain system

---

## Loading Heightmap Data

### Loading from Texture
```csharp
public class HeightmapLoader
{
    public static float[,] LoadHeightmapFromTexture(Texture2D texture)
    {
        int width = texture.width;
        int height = texture.height;
        float[,] heightmap = new float[width, height];
        
        Color[] pixels = texture.GetPixels();
        
        for (int y = 0; y < height; y++)
        {
            for (int x = 0; x < width; x++)
            {
                // Grayscale value (0-1)
                float heightValue = pixels[y * width + x].grayscale;
                heightmap[x, y] = heightValue;
            }
        }
        
        return heightmap;
    }
    
    public static float[,] LoadHeightmapFromRaw(string filePath, int width, int height, bool is16Bit = true)
    {
        float[,] heightmap = new float[width, height];
        byte[] data = File.ReadAllBytes(filePath);
        
        if (is16Bit)
        {
            for (int y = 0; y < height; y++)
            {
                for (int x = 0; x < width; x++)
                {
                    int index = (y * width + x) * 2;
                    ushort heightValue = BitConverter.ToUInt16(data, index);
                    heightmap[x, y] = heightValue / 65535f; // Normalize to 0-1
                }
            }
        }
        else
        {
            for (int y = 0; y < height; y++)
            {
                for (int x = 0; x < width; x++)
                {
                    int index = y * width + x;
                    heightmap[x, y] = data[index] / 255f; // Normalize to 0-1
                }
            }
        }
        
        return heightmap;
    }
}
```

---

## Heightmap to Terrain Mesh

### Mesh Generation
```csharp
public class HeightmapToMesh
{
    public static Mesh GenerateMesh(float[,] heightmap, float scaleX, float scaleY, float scaleZ)
    {
        int width = heightmap.GetLength(0);
        int height = heightmap.GetLength(1);
        
        Mesh mesh = new Mesh();
        List<Vector3> vertices = new List<Vector3>();
        List<int> triangles = new List<int>();
        List<Vector2> uvs = new List<Vector2>();
        
        // Generate vertices
        for (int z = 0; z < height; z++)
        {
            for (int x = 0; x < width; x++)
            {
                float y = heightmap[x, z] * scaleY;
                vertices.Add(new Vector3(x * scaleX, y, z * scaleZ));
                uvs.Add(new Vector2(x / (float)width, z / (float)height));
            }
        }
        
        // Generate triangles
        for (int z = 0; z < height - 1; z++)
        {
            for (int x = 0; x < width - 1; x++)
            {
                int current = z * width + x;
                int next = current + width;
                
                // First triangle
                triangles.Add(current);
                triangles.Add(next + 1);
                triangles.Add(next);
                
                // Second triangle
                triangles.Add(current);
                triangles.Add(current + 1);
                triangles.Add(next + 1);
            }
        }
        
        mesh.vertices = vertices.ToArray();
        mesh.triangles = triangles.ToArray();
        mesh.uv = uvs.ToArray();
        mesh.RecalculateNormals();
        mesh.RecalculateBounds();
        
        return mesh;
    }
}
```

---

## Height Queries with Interpolation

### Bilinear Interpolation
```csharp
public class HeightmapQuery
{
    private float[,] heightmap;
    private int width;
    private int height;
    private float scaleX;
    private float scaleZ;
    private float heightScale;
    
    public HeightmapQuery(float[,] heightmap, float scaleX, float scaleZ, float heightScale)
    {
        this.heightmap = heightmap;
        this.width = heightmap.GetLength(0);
        this.height = heightmap.GetLength(1);
        this.scaleX = scaleX;
        this.scaleZ = scaleZ;
        this.heightScale = heightScale;
    }
    
    public float GetHeight(float worldX, float worldZ)
    {
        // Convert world to grid coordinates
        float gridX = worldX / scaleX;
        float gridZ = worldZ / scaleZ;
        
        // Clamp to valid range
        gridX = Mathf.Clamp(gridX, 0, width - 1);
        gridZ = Mathf.Clamp(gridZ, 0, height - 1);
        
        // Get integer and fractional parts
        int x0 = Mathf.FloorToInt(gridX);
        int z0 = Mathf.FloorToInt(gridZ);
        int x1 = Mathf.Min(x0 + 1, width - 1);
        int z1 = Mathf.Min(z0 + 1, height - 1);
        
        float fx = gridX - x0;
        float fz = gridZ - z0;
        
        // Get corner heights
        float h00 = heightmap[x0, z0];
        float h10 = heightmap[x1, z0];
        float h01 = heightmap[x0, z1];
        float h11 = heightmap[x1, z1];
        
        // Bilinear interpolation
        float h0 = Mathf.Lerp(h00, h10, fx);
        float h1 = Mathf.Lerp(h01, h11, fx);
        float height = Mathf.Lerp(h0, h1, fz);
        
        return height * heightScale;
    }
}
```

---

## Normal Calculation

### Surface Normal from Gradient
```csharp
public class HeightmapNormals
{
    public static Vector3 CalculateNormal(float[,] heightmap, int x, int z, float scaleX, float scaleZ, float heightScale)
    {
        int width = heightmap.GetLength(0);
        int height = heightmap.GetLength(1);
        
        // Get neighboring heights (with bounds checking)
        float hL = (x > 0) ? heightmap[x - 1, z] : heightmap[x, z];
        float hR = (x < width - 1) ? heightmap[x + 1, z] : heightmap[x, z];
        float hD = (z > 0) ? heightmap[x, z - 1] : heightmap[x, z];
        float hU = (z < height - 1) ? heightmap[x, z + 1] : heightmap[x, z];
        
        // Calculate gradients (central difference)
        float dx = (hR - hL) / (2f * scaleX) * heightScale;
        float dz = (hU - hD) / (2f * scaleZ) * heightScale;
        
        // Normal vector (pointing up)
        Vector3 normal = new Vector3(-dx, 1f, -dz).normalized;
        
        return normal;
    }
    
    public static Vector3[] CalculateAllNormals(float[,] heightmap, float scaleX, float scaleZ, float heightScale)
    {
        int width = heightmap.GetLength(0);
        int height = heightmap.GetLength(1);
        Vector3[] normals = new Vector3[width * height];
        
        for (int z = 0; z < height; z++)
        {
            for (int x = 0; x < width; x++)
            {
                normals[z * width + x] = CalculateNormal(heightmap, x, z, scaleX, scaleZ, heightScale);
            }
        }
        
        return normals;
    }
}
```

---

## Slope Calculation

### Slope and Angle
```csharp
public class HeightmapSlope
{
    public static float CalculateSlope(float[,] heightmap, int x, int z, float scaleX, float scaleZ, float heightScale)
    {
        int width = heightmap.GetLength(0);
        int height = heightmap.GetLength(1);
        
        // Get gradients
        float hL = (x > 0) ? heightmap[x - 1, z] : heightmap[x, z];
        float hR = (x < width - 1) ? heightmap[x + 1, z] : heightmap[x, z];
        float hD = (z > 0) ? heightmap[x, z - 1] : heightmap[x, z];
        float hU = (z < height - 1) ? heightmap[x, z + 1] : heightmap[x, z];
        
        float dx = (hR - hL) / (2f * scaleX) * heightScale;
        float dz = (hU - hD) / (2f * scaleZ) * heightScale;
        
        // Slope magnitude
        float slope = Mathf.Sqrt(dx * dx + dz * dz);
        
        return slope;
    }
    
    public static float CalculateSlopeAngle(float[,] heightmap, int x, int z, float scaleX, float scaleZ, float heightScale)
    {
        float slope = CalculateSlope(heightmap, x, z, scaleX, scaleZ, heightScale);
        return Mathf.Atan(slope) * Mathf.Rad2Deg; // Convert to degrees
    }
    
    public static float[,] GenerateSlopeMap(float[,] heightmap, float scaleX, float scaleZ, float heightScale)
    {
        int width = heightmap.GetLength(0);
        int height = heightmap.GetLength(1);
        float[,] slopeMap = new float[width, height];
        
        for (int z = 0; z < height; z++)
        {
            for (int x = 0; x < width; x++)
            {
                slopeMap[x, z] = CalculateSlope(heightmap, x, z, scaleX, scaleZ, heightScale);
            }
        }
        
        return slopeMap;
    }
}
```

---

## Unity Terrain Integration

### Applying Heightmap to Unity Terrain
```csharp
public class TerrainHeightmapApplier
{
    public static void ApplyHeightmapToTerrain(TerrainData terrainData, float[,] heightmap)
    {
        int width = heightmap.GetLength(0);
        int height = heightmap.GetLength(1);
        
        // Unity terrain expects heights in range [0, 1]
        float[,] normalizedHeights = new float[width, height];
        
        // Find min/max for normalization
        float min = float.MaxValue;
        float max = float.MinValue;
        
        for (int x = 0; x < width; x++)
        {
            for (int z = 0; z < height; z++)
            {
                if (heightmap[x, z] < min) min = heightmap[x, z];
                if (heightmap[x, z] > max) max = heightmap[x, z];
            }
        }
        
        // Normalize to [0, 1]
        float range = max - min;
        if (range > 0)
        {
            for (int x = 0; x < width; x++)
            {
                for (int z = 0; z < height; z++)
                {
                    normalizedHeights[x, z] = (heightmap[x, z] - min) / range;
                }
            }
        }
        
        // Apply to terrain
        terrainData.SetHeights(0, 0, normalizedHeights);
    }
}
```

---

## Procedural Heightmap Generation

### Perlin Noise Heightmap
```csharp
public class ProceduralHeightmap
{
    public static float[,] GeneratePerlinNoise(int width, int height, float scale, int octaves, float persistence, float lacunarity)
    {
        float[,] heightmap = new float[width, height];
        
        for (int x = 0; x < width; x++)
        {
            for (int z = 0; z < height; z++)
            {
                float amplitude = 1f;
                float frequency = scale;
                float noiseValue = 0f;
                float maxValue = 0f;
                
                for (int i = 0; i < octaves; i++)
                {
                    float sampleX = x / frequency;
                    float sampleZ = z / frequency;
                    float perlinValue = Mathf.PerlinNoise(sampleX, sampleZ) * 2f - 1f;
                    
                    noiseValue += perlinValue * amplitude;
                    maxValue += amplitude;
                    
                    amplitude *= persistence;
                    frequency *= lacunarity;
                }
                
                heightmap[x, z] = noiseValue / maxValue;
            }
        }
        
        return heightmap;
    }
}
```

---

## Heightmap Manager Component

### Complete MonoBehaviour
```csharp
public class HeightmapManager : MonoBehaviour
{
    [Header("Heightmap Settings")]
    public Texture2D heightmapTexture;
    public string rawHeightmapPath;
    public bool is16Bit = true;
    public int heightmapWidth = 512;
    public int heightmapHeight = 512;
    
    [Header("Terrain Settings")]
    public float terrainWidth = 100f;
    public float terrainHeight = 50f;
    public float terrainLength = 100f;
    
    private float[,] heightmap;
    private HeightmapQuery heightQuery;
    
    void Start()
    {
        LoadHeightmap();
        CreateTerrainMesh();
    }
    
    void LoadHeightmap()
    {
        if (heightmapTexture != null)
        {
            heightmap = HeightmapLoader.LoadHeightmapFromTexture(heightmapTexture);
        }
        else if (!string.IsNullOrEmpty(rawHeightmapPath))
        {
            heightmap = HeightmapLoader.LoadHeightmapFromRaw(
                rawHeightmapPath, 
                heightmapWidth, 
                heightmapHeight, 
                is16Bit
            );
        }
        
        float scaleX = terrainWidth / heightmap.GetLength(0);
        float scaleZ = terrainLength / heightmap.GetLength(1);
        heightQuery = new HeightmapQuery(heightmap, scaleX, scaleZ, terrainHeight);
    }
    
    void CreateTerrainMesh()
    {
        float scaleX = terrainWidth / heightmap.GetLength(0);
        float scaleZ = terrainLength / heightmap.GetLength(1);
        
        Mesh mesh = HeightmapToMesh.GenerateMesh(heightmap, scaleX, terrainHeight, scaleZ);
        
        MeshFilter meshFilter = GetComponent<MeshFilter>();
        if (meshFilter == null)
            meshFilter = gameObject.AddComponent<MeshFilter>();
        
        MeshRenderer meshRenderer = GetComponent<MeshRenderer>();
        if (meshRenderer == null)
            meshRenderer = gameObject.AddComponent<MeshRenderer>();
        
        meshFilter.mesh = mesh;
    }
    
    public float GetHeight(Vector3 worldPosition)
    {
        if (heightQuery == null) return 0f;
        return heightQuery.GetHeight(worldPosition.x, worldPosition.z);
    }
    
    public Vector3 GetNormal(Vector3 worldPosition)
    {
        // Convert world to grid coordinates and calculate normal
        // Implementation similar to HeightmapNormals.CalculateNormal
        return Vector3.up; // Placeholder
    }
}
```

---

## Exercises for Self-Learning

1. **Load Heightmap**: Load a heightmap image and display as terrain
2. **Height Query**: Implement height lookup with bilinear interpolation
3. **Normal Calculation**: Calculate and visualize surface normals
4. **Procedural Generation**: Generate terrain using Perlin noise
5. **Slope Map**: Create a slope visualization texture

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand the mathematics (2-math-matrixes-manifolds.md)

---

## References
- Unity Terrain API
- Unity Mesh API
- Procedural generation tutorials
- Heightmap file formats
