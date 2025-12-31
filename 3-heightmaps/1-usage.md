# Heightmaps: Usage and Applications

## Overview
Heightmaps are 2D arrays of elevation data used to represent terrain in games and simulations. They provide an efficient way to store and render complex landscapes.

## Learning Objectives
- Understand what heightmaps are and when to use them
- Identify common use cases in game development
- Learn different heightmap formats and their trade-offs
- Recognize performance characteristics and limitations

---

## What is a Heightmap?

### Definition
A heightmap is a 2D grid of height values where each pixel/element represents the elevation at that point in 3D space.

### Basic Concept
- **2D Grid**: Each cell contains a height value (usually 0-255 for 8-bit, or float for higher precision)
- **3D Terrain**: Height values are extruded along the Y-axis (or Z-axis) to create 3D terrain
- **Efficient Storage**: Compact representation of complex terrain geometry

---

## Common Use Cases

### 1. Terrain Generation and Rendering
**Problem**: Create and display large-scale terrain efficiently
- **Use Case**: Open-world games, flight simulators, strategy games
- **Benefit**: Simple data structure, easy to render, good compression
- **Example**: Skyrim's terrain, Minecraft's terrain generation

### 2. Procedural Terrain Generation
**Problem**: Generate realistic terrain algorithmically
- **Use Case**: Randomly generated worlds, infinite terrain
- **Benefit**: Can generate heightmaps using noise functions (Perlin, Simplex)
- **Example**: No Man's Sky procedural planets

### 3. Terrain Editing Tools
**Problem**: Allow designers to sculpt terrain
- **Use Case**: Level editors, world building tools
- **Benefit**: Intuitive 2D interface for 3D terrain editing
- **Example**: Unity Terrain Tools, Unreal Landscape Editor

### 4. Height-Based Gameplay
**Problem**: Use terrain height for gameplay mechanics
- **Use Case**: Line-of-sight calculations, pathfinding, water flow
- **Benefit**: Fast height lookups for gameplay systems
- **Example**: Artillery trajectory calculations, visibility systems

### 5. Texture Blending
**Problem**: Blend different textures based on terrain features
- **Use Case**: Realistic terrain texturing (grass on flat, rock on steep)
- **Benefit**: Use height/slope to determine texture weights
- **Example**: Grass on low/flat areas, snow on peaks

### 6. LOD (Level of Detail) Systems
**Problem**: Render terrain efficiently at different distances
- **Use Case**: Large open worlds with varying detail
- **Benefit**: Can generate multiple LOD levels from same heightmap
- **Example**: Far Cry, Assassin's Creed terrain rendering

---

## When to Use Heightmaps

### ✅ Good Use Cases
- **Large-scale terrain** (kilometers of terrain)
- **Relatively smooth terrain** (hills, valleys, rolling landscapes)
- **2.5D terrain** (height varies, but no overhangs/caves)
- **Procedural generation** needs
- **Terrain editing** workflows
- **Memory-efficient** terrain storage

### ❌ Poor Use Cases
- **Overhangs and caves** (heightmaps are 2.5D, not full 3D)
- **Very detailed geometry** (cliffs, complex rock formations)
- **Dynamic terrain** that changes frequently
- **Multi-layer terrain** (different materials at same X,Z)
- **Vertical structures** (buildings, bridges spanning terrain)

---

## Heightmap Formats

### 8-Bit Grayscale (0-255)
- **Storage**: 1 byte per pixel
- **Range**: 256 discrete height levels
- **Use Case**: Simple terrains, memory-constrained systems
- **Limitation**: Limited precision, visible stepping

### 16-Bit Grayscale (0-65535)
- **Storage**: 2 bytes per pixel
- **Range**: 65,536 discrete height levels
- **Use Case**: Most game engines, good balance
- **Limitation**: Still discrete, but much smoother

### 32-Bit Float
- **Storage**: 4 bytes per pixel
- **Range**: Continuous floating-point values
- **Use Case**: High-precision terrain, scientific simulations
- **Limitation**: Larger file size, may be overkill for games

### RAW Format
- **Raw binary data**, no header
- **Pros**: Simple, fast to load
- **Cons**: No metadata, must know dimensions

### Image Formats (PNG, TGA, etc.)
- **Standard image formats**
- **Pros**: Easy to edit in image editors, compression
- **Cons**: May lose precision, conversion overhead

---

## Performance Characteristics

### Memory Usage
- **8-bit**: $w \times h \times 1$ bytes
- **16-bit**: $w \times h \times 2$ bytes
- **32-bit float**: $w \times h \times 4$ bytes

Example: 1024×1024 heightmap
- 8-bit: 1 MB
- 16-bit: 2 MB
- 32-bit: 4 MB

### Rendering Performance
- **Vertex Count**: $w \times h$ vertices (can be reduced with LOD)
- **Triangle Count**: $2 \times (w-1) \times (h-1)$ triangles
- **Rendering**: Can use GPU instancing, chunking, LOD

### Query Performance
- **Height Lookup**: O(1) - direct array access
- **Bilinear Interpolation**: O(1) - 4 samples
- **Slope Calculation**: O(1) - gradient from neighbors

---

## Practical Examples

### Example 1: Open World Terrain
```
Scenario: 10km × 10km game world
Heightmap: 1024×1024 (10m per pixel)
Storage: 2 MB (16-bit)
Vertices: ~1 million (with LOD reduction)
Result: Smooth, detailed terrain with efficient storage
```

### Example 2: Procedural Generation
```
Scenario: Infinite terrain generation
Method: Perlin noise → heightmap
Heightmap: Generated on-demand in chunks
Result: Seamless, varied terrain without storage overhead
```

### Example 3: Terrain Editing
```
Scenario: Level designer sculpts terrain
Tool: Paint height values in 2D editor
Result: Intuitive workflow, easy to iterate
```

---

## Comparison with Alternatives

### Heightmap vs. Voxel Terrain
| Aspect | Heightmap | Voxel |
|--------|-----------|-------|
| **Overhangs** | ❌ No | ✅ Yes |
| **Storage** | ✅ Efficient | ⚠️ Larger |
| **Editing** | ✅ Easy (2D) | ⚠️ Complex (3D) |
| **Rendering** | ✅ Fast | ⚠️ Slower |
| **Use Case** | Terrain | Full 3D worlds |

### Heightmap vs. Mesh Terrain
| Aspect | Heightmap | Mesh |
|--------|-----------|------|
| **Storage** | ✅ Compact | ❌ Large |
| **Editing** | ✅ Height painting | ⚠️ Vertex editing |
| **Procedural** | ✅ Easy | ⚠️ Complex |
| **Flexibility** | ⚠️ Limited (2.5D) | ✅ Full 3D |

---

## Integration with Game Systems

### Physics Integration
- Use heightmap for ground collision
- Raycast against heightmap for ground detection
- Calculate slopes for movement/physics

### Pathfinding
- Heightmap provides walkable/non-walkable areas
- Slope-based pathfinding (avoid steep areas)
- Height differences affect path costs

### Water Systems
- Water level relative to heightmap
- Flow simulation based on height gradients
- Flooding calculations

### Vegetation Placement
- Place trees/grass based on height and slope
- Avoid placing on steep slopes
- Density based on height (e.g., more trees at certain elevations)

---

## Limitations and Workarounds

### Limitation: No Overhangs
**Problem**: Can't represent caves, bridges, overhangs
**Workaround**: Use additional geometry for overhangs, separate cave systems

### Limitation: Single Height per X,Z
**Problem**: Can't have multiple surfaces at same location
**Workaround**: Use decals, additional meshes for details

### Limitation: Discrete Sampling
**Problem**: Height values are sampled, may miss fine details
**Workaround**: Higher resolution heightmaps, normal maps for detail

### Limitation: Large File Sizes
**Problem**: High-resolution heightmaps are large
**Workaround**: Compression, streaming, procedural generation

---

## Exercises for Self-Learning

1. **Format Comparison**: Compare file sizes for different heightmap formats
2. **Use Case Analysis**: Identify 3 games that use heightmaps and how
3. **Memory Calculation**: Calculate memory for a 5km×5km world at different resolutions
4. **Limitation Research**: Find examples of games that overcame heightmap limitations
5. **Tool Research**: Research terrain editing tools and their heightmap support

---

## Next Steps
- **Math**: Learn heightmap mathematics and manifolds (2-math-matrixes-manifolds.md)
- **Implementation**: See Unity code examples (3-unity-implementation.md)
- **Engine Comparison**: Compare different engine approaches (4-engine-diffs-unreal-frostbite-lumberyard-etc.md)

---

## References and Further Reading
- Terrain rendering techniques
- Procedural generation algorithms
- Game engine terrain systems
- Heightmap editing tools documentation

