# Cubic Units: Usage and Applications

## Overview
Cubic units (voxels) are 3D pixels that represent discrete volumes in space. This document covers use cases, applications, and when to use cubic/voxel-based systems.

## Learning Objectives
- Understand what cubic units/voxels are
- Identify use cases and applications
- Learn different voxel systems and formats
- Recognize performance characteristics

---

## What are Cubic Units/Voxels?

### Definition
A voxel (volumetric pixel) is a 3D cubic unit that represents a discrete volume in 3D space, similar to how a pixel represents a 2D point.

### Basic Concept
- **3D Grid**: Space divided into uniform cubes
- **Discrete Values**: Each voxel has properties (material, density, color, etc.)
- **Volume Representation**: Can represent solid volumes, not just surfaces
- **Efficient Storage**: Can compress empty space

---

## Common Use Cases

### 1. Voxel-Based Games
**Problem**: Create destructible, modifiable worlds
- **Use Case**: Minecraft, Terraria, 7 Days to Die
- **Benefit**: Full 3D modification, infinite possibilities
- **Example**: Players can dig, build, modify terrain

### 2. Medical Imaging
**Problem**: Represent 3D scan data (CT, MRI)
- **Use Case**: Medical visualization, diagnosis
- **Benefit**: Accurate 3D representation of anatomy
- **Example**: 3D reconstruction from CT scans

### 3. Scientific Visualization
**Problem**: Visualize 3D scientific data
- **Use Case**: Fluid simulation, molecular structures, geological data
- **Benefit**: Natural representation of volumetric data
- **Example**: Cloud visualization, molecular models

### 4. 3D Printing Preparation
**Problem**: Prepare models for 3D printing
- **Use Case**: Slicing software, model preparation
- **Benefit**: Convert surfaces to printable volumes
- **Example**: Convert mesh to voxel, then to print layers

### 5. Terrain Systems
**Problem**: Create modifiable terrain
- **Use Case**: Voxel terrain engines
- **Benefit**: Can dig, build, modify terrain in real-time
- **Example**: Voxel-based terrain in games

### 6. Volume Rendering
**Problem**: Render volumetric effects
- **Use Case**: Clouds, fog, smoke, fire
- **Benefit**: Realistic volumetric rendering
- **Example**: Volumetric clouds in games

---

## When to Use Voxels

### ✅ Good Use Cases
- **Destructible environments** (digging, building)
- **Volumetric data** (medical scans, scientific data)
- **Modifiable worlds** (player can change terrain)
- **Complex 3D shapes** (overhangs, caves, interiors)
- **Procedural generation** (infinite worlds)
- **Volume effects** (clouds, fog)

### ❌ Poor Use Cases
- **Static environments** (heightmap better)
- **Simple surfaces** (mesh more efficient)
- **High-detail surfaces** (too many voxels needed)
- **Real-time performance critical** (can be expensive)
- **Artistic control** (harder to achieve specific look)

---

## Voxel Data Formats

### Dense Voxel Grid
- **Storage**: 3D array, one value per voxel
- **Memory**: $w \times h \times d$ values
- **Use**: Small volumes, simple data
- **Limitation**: Very memory-intensive for large volumes

### Sparse Voxel Octree (SVO)
- **Storage**: Octree, only store non-empty voxels
- **Memory**: Much less than dense (only occupied voxels)
- **Use**: Large volumes with mostly empty space
- **Benefit**: Efficient compression

### Run-Length Encoding (RLE)
- **Storage**: Store runs of same value
- **Memory**: Compressed, good for uniform regions
- **Use**: Medical imaging, simple voxel data
- **Benefit**: Good compression ratio

### Chunked Storage
- **Storage**: Divide space into chunks
- **Memory**: Load only visible chunks
- **Use**: Infinite worlds (Minecraft-style)
- **Benefit**: Streaming, memory management

---

## Performance Characteristics

### Memory Usage
- **Dense**: $O(n^3)$ where $n$ is resolution
- **Sparse**: $O(k)$ where $k$ is number of filled voxels
- **Example**: 256³ dense = 16M voxels, sparse might be 1M

### Rendering Performance
- **Raycasting**: $O(n)$ per ray
- **Mesh Extraction**: $O(n^3)$ but done once
- **GPU Rendering**: Can be very fast with proper optimization

### Query Performance
- **Point Query**: $O(1)$ for dense, $O(\log n)$ for octree
- **Range Query**: $O(k)$ where $k$ is results
- **Nearest Neighbor**: $O(\log n)$ for octree

---

## Comparison with Alternatives

### Voxels vs. Heightmaps
| Aspect | Voxels | Heightmaps |
|--------|--------|------------|
| **Overhangs** | ✅ Yes | ❌ No |
| **Caves** | ✅ Yes | ❌ No |
| **Memory** | ⚠️ Can be large | ✅ Efficient |
| **Modification** | ✅ Easy | ⚠️ Limited |
| **Use Case** | Full 3D | Terrain only |

### Voxels vs. Meshes
| Aspect | Voxels | Meshes |
|--------|--------|--------|
| **Storage** | ⚠️ Can be large | ✅ Compact |
| **Modification** | ✅ Easy | ❌ Hard |
| **Rendering** | ⚠️ Can be slow | ✅ Fast |
| **Detail** | ⚠️ Discrete | ✅ Smooth |

---

## Practical Examples

### Example 1: Minecraft-Style Game
```
Scenario: Infinite voxel world
Storage: Chunked sparse voxel octree
Resolution: 16×16×16 chunks
Result: Efficient storage, fast queries, modifiable
```

### Example 2: Medical Visualization
```
Scenario: CT scan data
Storage: Dense voxel grid or compressed
Resolution: 512×512×512 (typical)
Result: Accurate 3D representation of anatomy
```

### Example 3: Volumetric Clouds
```
Scenario: Realistic cloud rendering
Storage: Sparse voxel octree
Resolution: Variable, high detail near camera
Result: Realistic volumetric clouds
```

---

## Integration Considerations

### Physics Integration
- Voxels can be used for collision detection
- Can represent destructible geometry
- Good for terrain modification

### Rendering Integration
- Extract mesh from voxels (marching cubes)
- Direct voxel rendering (raycasting)
- Hybrid approaches

### Streaming
- Load chunks on-demand
- Unload distant chunks
- Efficient memory management

---

## Exercises for Self-Learning

1. **Format Comparison**: Compare memory usage of dense vs. sparse voxels
2. **Use Case Analysis**: Identify 3 applications using voxels
3. **Storage Design**: Design voxel storage for infinite world
4. **Rendering Research**: Research voxel rendering techniques
5. **Tool Research**: Find voxel editing tools and formats

---

## Next Steps
- **Math**: Learn voxel mathematics (2-math.md)
- **Implementation**: See Unity code examples (3-unity-implementation-data-file-import.md)
- **Standard Format**: Learn standard data formats (standard-galactic-units/)

---

## References and Further Reading
- Voxel rendering techniques
- Sparse voxel octree papers
- Minecraft-style game development
- Medical imaging formats
