# Cubic Units: Unity Implementation - Data File Import

## Overview
Practical implementation of voxel/cubic unit systems in Unity, including loading from files, mesh generation, and integration with Unity systems.

## Learning Objectives
- Load voxel data from files (including SGU format)
- Generate meshes from voxel data
- Implement voxel queries and operations
- Optimize for Unity performance

---

## Loading Voxel Data

### Basic Voxel Data Structure
```csharp
public class VoxelData
{
    public int width, height, depth;
    public byte[,,] voxels; // Or float[,,] for density
    
    public VoxelData(int w, int h, int d)
    {
        width = w;
        height = h;
        depth = d;
        voxels = new byte[w, h, d];
    }
    
    public byte GetVoxel(int x, int y, int z)
    {
        if (x < 0 || x >= width || y < 0 || y >= height || z < 0 || z >= depth)
            return 0; // Air/empty
        return voxels[x, y, z];
    }
    
    public void SetVoxel(int x, int y, int z, byte value)
    {
        if (x >= 0 && x < width && y >= 0 && y < height && z >= 0 && z < depth)
            voxels[x, y, z] = value;
    }
}
```

### Loading from Raw File
```csharp
public class VoxelLoader
{
    public static VoxelData LoadFromRaw(string filePath, int width, int height, int depth, bool is16Bit = false)
    {
        VoxelData data = new VoxelData(width, height, depth);
        byte[] fileData = File.ReadAllBytes(filePath);
        
        if (is16Bit)
        {
            for (int z = 0; z < depth; z++)
            {
                for (int y = 0; y < height; y++)
                {
                    for (int x = 0; x < width; x++)
                    {
                        int index = (z * height * width + y * width + x) * 2;
                        ushort value = BitConverter.ToUInt16(fileData, index);
                        data.SetVoxel(x, y, z, (byte)(value / 256)); // Convert to 8-bit
                    }
                }
            }
        }
        else
        {
            for (int z = 0; z < depth; z++)
            {
                for (int y = 0; y < height; y++)
                {
                    for (int x = 0; x < width; x++)
                    {
                        int index = z * height * width + y * width + x;
                        data.SetVoxel(x, y, z, fileData[index]);
                    }
                }
            }
        }
        
        return data;
    }
}
```

### Loading from Texture
```csharp
public static VoxelData LoadFromTexture(Texture2D texture, int depth)
{
    int width = texture.width;
    int height = texture.height;
    VoxelData data = new VoxelData(width, height, depth);
    
    Color[] pixels = texture.GetPixels();
    
    // Treat each pixel as a slice, or use grayscale for height
    for (int z = 0; z < depth; z++)
    {
        for (int y = 0; y < height; y++)
        {
            for (int x = 0; x < width; x++)
            {
                int pixelIndex = y * width + x;
                float heightValue = pixels[pixelIndex].grayscale;
                byte voxelValue = (byte)(heightValue * depth);
                
                // Fill from bottom up to height value
                for (int fillZ = 0; fillZ <= voxelValue && fillZ < depth; fillZ++)
                {
                    data.SetVoxel(x, y, fillZ, 255); // Solid
                }
            }
        }
    }
    
    return data;
}
```

---

## SGU Format Loading

### SGU File Header
```csharp
[System.Serializable]
public struct SGUHeader
{
    public uint magic;           // 'SGU '
    public ushort version;
    public ushort flags;
    public uint width;
    public uint height;
    public uint depth;
    public uint voxelSize;
    public uint dataOffset;
    public uint metadataSize;
    public uint checksum;
    
    public bool IsValid()
    {
        return magic == 0x20554753; // 'SGU ' in little-endian
    }
}
```

### SGU Loader
```csharp
public class SGULoader
{
    public static VoxelData LoadSGU(string filePath)
    {
        byte[] fileData = File.ReadAllBytes(filePath);
        
        // Read header
        SGUHeader header = new SGUHeader();
        int offset = 0;
        
        header.magic = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.version = BitConverter.ToUInt16(fileData, offset); offset += 2;
        header.flags = BitConverter.ToUInt16(fileData, offset); offset += 2;
        header.width = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.height = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.depth = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.voxelSize = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.dataOffset = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.metadataSize = BitConverter.ToUInt32(fileData, offset); offset += 4;
        header.checksum = BitConverter.ToUInt32(fileData, offset);
        
        if (!header.IsValid())
        {
            Debug.LogError("Invalid SGU file!");
            return null;
        }
        
        // Read voxel data
        VoxelData data = new VoxelData((int)header.width, (int)header.height, (int)header.depth);
        
        bool compressed = (header.flags & 0x01) != 0;
        int dataStart = (int)header.dataOffset;
        
        if (compressed)
        {
            // Decompress (simplified - would use actual decompression library)
            byte[] compressedData = new byte[fileData.Length - dataStart];
            Array.Copy(fileData, dataStart, compressedData, 0, compressedData.Length);
            byte[] decompressed = DecompressLZ4(compressedData); // Implement based on library
            LoadVoxelData(data, decompressed, header);
        }
        else
        {
            LoadVoxelData(data, fileData, header, dataStart);
        }
        
        return data;
    }
    
    static void LoadVoxelData(VoxelData data, byte[] fileData, SGUHeader header, int startOffset = 0)
    {
        int offset = startOffset;
        for (int z = 0; z < header.depth; z++)
        {
            for (int y = 0; y < header.height; y++)
            {
                for (int x = 0; x < header.width; x++)
                {
                    byte value = fileData[offset++];
                    data.SetVoxel(x, y, z, value);
                }
            }
        }
    }
}
```

---

## Mesh Generation (Marching Cubes)

### Marching Cubes Implementation
```csharp
public class MarchingCubes
{
    private static int[] edgeTable = new int[256] { /* Lookup table */ };
    private static int[][] triTable = new int[256][] { /* Lookup table */ };
    
    public static Mesh GenerateMesh(VoxelData data, byte threshold = 128)
    {
        List<Vector3> vertices = new List<Vector3>();
        List<int> triangles = new List<int>();
        
        for (int x = 0; x < data.width - 1; x++)
        {
            for (int y = 0; y < data.height - 1; y++)
            {
                for (int z = 0; z < data.depth - 1; z++)
                {
                    ProcessCube(data, x, y, z, threshold, vertices, triangles);
                }
            }
        }
        
        Mesh mesh = new Mesh();
        mesh.vertices = vertices.ToArray();
        mesh.triangles = triangles.ToArray();
        mesh.RecalculateNormals();
        mesh.RecalculateBounds();
        
        return mesh;
    }
    
    static void ProcessCube(VoxelData data, int x, int y, int z, byte threshold, 
                           List<Vector3> vertices, List<int> triangles)
    {
        // Get 8 corner values
        byte[] cornerValues = new byte[8];
        cornerValues[0] = data.GetVoxel(x, y, z);
        cornerValues[1] = data.GetVoxel(x + 1, y, z);
        cornerValues[2] = data.GetVoxel(x + 1, y, z + 1);
        cornerValues[3] = data.GetVoxel(x, y, z + 1);
        cornerValues[4] = data.GetVoxel(x, y + 1, z);
        cornerValues[5] = data.GetVoxel(x + 1, y + 1, z);
        cornerValues[6] = data.GetVoxel(x + 1, y + 1, z + 1);
        cornerValues[7] = data.GetVoxel(x, y + 1, z + 1);
        
        // Calculate cube index
        int cubeIndex = 0;
        for (int i = 0; i < 8; i++)
        {
            if (cornerValues[i] > threshold)
                cubeIndex |= 1 << i;
        }
        
        // Get edge table value
        int edges = edgeTable[cubeIndex];
        if (edges == 0) return; // No triangles
        
        // Calculate vertex positions on edges
        Vector3[] edgeVertices = new Vector3[12];
        for (int i = 0; i < 12; i++)
        {
            if ((edges & (1 << i)) != 0)
            {
                edgeVertices[i] = InterpolateVertex(x, y, z, i, cornerValues, threshold);
            }
        }
        
        // Generate triangles
        int[] triIndices = triTable[cubeIndex];
        for (int i = 0; triIndices[i] != -1; i += 3)
        {
            int v0 = vertices.Count;
            vertices.Add(edgeVertices[triIndices[i]]);
            vertices.Add(edgeVertices[triIndices[i + 1]]);
            vertices.Add(edgeVertices[triIndices[i + 2]]);
            
            triangles.Add(v0);
            triangles.Add(v0 + 1);
            triangles.Add(v0 + 2);
        }
    }
    
    static Vector3 InterpolateVertex(int x, int y, int z, int edge, byte[] corners, byte threshold)
    {
        // Interpolate vertex position on edge based on threshold
        // Implementation depends on edge number (0-11)
        // Returns world position
        return new Vector3(x, y, z); // Simplified
    }
}
```

---

## Voxel Queries

### Height Query
```csharp
public class VoxelQueries
{
    public static float GetHeight(VoxelData data, float worldX, float worldZ, float voxelSize)
    {
        int x = Mathf.FloorToInt(worldX / voxelSize);
        int z = Mathf.FloorToInt(worldZ / voxelSize);
        
        // Find topmost solid voxel
        for (int y = data.height - 1; y >= 0; y--)
        {
            if (data.GetVoxel(x, y, z) > 0)
            {
                return (y + 1) * voxelSize;
            }
        }
        
        return 0f;
    }
    
    public static bool IsSolid(VoxelData data, Vector3 worldPos, float voxelSize)
    {
        int x = Mathf.FloorToInt(worldPos.x / voxelSize);
        int y = Mathf.FloorToInt(worldPos.y / voxelSize);
        int z = Mathf.FloorToInt(worldPos.z / voxelSize);
        
        return data.GetVoxel(x, y, z) > 0;
    }
}
```

---

## Chunked Voxel System

### Voxel Chunk
```csharp
public class VoxelChunk : MonoBehaviour
{
    public VoxelData data;
    public MeshFilter meshFilter;
    public MeshCollider meshCollider;
    public float voxelSize = 1f;
    
    public void GenerateMesh()
    {
        Mesh mesh = MarchingCubes.GenerateMesh(data);
        meshFilter.mesh = mesh;
        if (meshCollider != null)
            meshCollider.sharedMesh = mesh;
    }
    
    public void LoadFromFile(string filePath)
    {
        data = VoxelLoader.LoadFromRaw(filePath, 32, 32, 32);
        GenerateMesh();
    }
}
```

### Chunk Manager
```csharp
public class VoxelChunkManager : MonoBehaviour
{
    public GameObject chunkPrefab;
    public float chunkSize = 32f;
    public float voxelSize = 1f;
    public int renderDistance = 3;
    
    private Dictionary<Vector3Int, VoxelChunk> chunks = new Dictionary<Vector3Int, VoxelChunk>();
    private Vector3Int lastChunkPos;
    
    void Update()
    {
        Vector3 playerPos = Camera.main.transform.position;
        Vector3Int currentChunk = WorldToChunk(playerPos);
        
        if (currentChunk != lastChunkPos)
        {
            UpdateChunks(currentChunk);
            lastChunkPos = currentChunk;
        }
    }
    
    Vector3Int WorldToChunk(Vector3 worldPos)
    {
        return new Vector3Int(
            Mathf.FloorToInt(worldPos.x / chunkSize),
            Mathf.FloorToInt(worldPos.y / chunkSize),
            Mathf.FloorToInt(worldPos.z / chunkSize)
        );
    }
    
    void UpdateChunks(Vector3Int centerChunk)
    {
        // Unload distant chunks
        List<Vector3Int> toRemove = new List<Vector3Int>();
        foreach (var kvp in chunks)
        {
            float distance = Vector3Int.Distance(kvp.Key, centerChunk);
            if (distance > renderDistance)
            {
                Destroy(kvp.Value.gameObject);
                toRemove.Add(kvp.Key);
            }
        }
        foreach (var key in toRemove)
            chunks.Remove(key);
        
        // Load nearby chunks
        for (int x = -renderDistance; x <= renderDistance; x++)
        {
            for (int y = -renderDistance; y <= renderDistance; y++)
            {
                for (int z = -renderDistance; z <= renderDistance; z++)
                {
                    Vector3Int chunkPos = centerChunk + new Vector3Int(x, y, z);
                    if (!chunks.ContainsKey(chunkPos))
                    {
                        LoadChunk(chunkPos);
                    }
                }
            }
        }
    }
    
    void LoadChunk(Vector3Int chunkPos)
    {
        GameObject chunkObj = Instantiate(chunkPrefab);
        chunkObj.transform.position = new Vector3(
            chunkPos.x * chunkSize,
            chunkPos.y * chunkSize,
            chunkPos.z * chunkSize
        );
        
        VoxelChunk chunk = chunkObj.GetComponent<VoxelChunk>();
        // Load chunk data (from file or generate)
        chunks[chunkPos] = chunk;
    }
}
```

---

## Performance Optimization

### Mesh Simplification
```csharp
public static Mesh SimplifyMesh(Mesh mesh, float quality)
{
    // Use Unity's Mesh.CombineMeshes or third-party simplification
    // This is a placeholder - would use actual simplification algorithm
    return mesh;
}
```

### LOD System
```csharp
public class VoxelLOD : MonoBehaviour
{
    public VoxelData[] lodLevels; // Different resolutions
    public float[] lodDistances;
    
    void Update()
    {
        float distance = Vector3.Distance(transform.position, Camera.main.transform.position);
        
        int lodLevel = 0;
        for (int i = 0; i < lodDistances.Length; i++)
        {
            if (distance > lodDistances[i])
                lodLevel = i + 1;
        }
        
        lodLevel = Mathf.Min(lodLevel, lodLevels.Length - 1);
        // Switch to appropriate LOD mesh
    }
}
```

---

## Exercises for Self-Learning

1. **File Loading**: Implement loading from different voxel formats
2. **Mesh Generation**: Implement marching cubes algorithm
3. **Chunking**: Create chunked voxel system
4. **Queries**: Implement height and collision queries
5. **Optimization**: Add LOD and mesh simplification

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)
- **SGU Format**: Learn SGU format (standard-galactic-units/)

---

## References
- Unity Mesh API
- Marching Cubes algorithm
- File I/O in Unity
- Mesh optimization techniques
