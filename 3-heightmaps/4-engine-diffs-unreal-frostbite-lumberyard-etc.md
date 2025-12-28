# Heightmaps: Engine Differences (Unreal, Frostbite, Lumberyard, etc.)

## Overview
Different game engines implement heightmap/terrain systems differently. This document compares approaches across major engines.

## Learning Objectives
- Understand different engine terrain architectures
- Compare storage formats and compression
- Learn LOD and streaming strategies
- Recognize performance optimizations used

---

## Unity Terrain System

### Architecture
- **Component-Based**: Terrain GameObject with TerrainData asset
- **Splatmap System**: Multiple textures blended by height/slope
- **Tree/Detail System**: Built-in vegetation placement
- **LOD**: Automatic mesh LOD based on distance

### Heightmap Storage
- **Format**: 16-bit normalized floats (0-1 range)
- **Resolution**: Up to 4097×4097 per terrain tile
- **Compression**: Optional compression in TerrainData asset
- **Streaming**: Not built-in, requires custom implementation

### Key Features
- **Splatmaps**: 4 texture layers with blending
- **Detail Meshes**: Grass/rocks as instanced meshes
- **Tree System**: Billboard LOD for trees
- **Terrain Tools**: Built-in painting/editing tools

### Limitations
- **Single Terrain**: One heightmap per Terrain component
- **No Streaming**: Large worlds require multiple Terrain objects
- **Memory**: Full heightmap loaded in memory

---

## Unreal Engine Landscape System

### Architecture
- **Component-Based**: Landscape actor with LandscapeComponent
- **Material System**: Complex material graphs for texturing
- **Foliage System**: Separate foliage tool for vegetation
- **LOD**: Hierarchical LOD (HLOD) system

### Heightmap Storage
- **Format**: 16-bit integers (0-65535)
- **Resolution**: Up to 8129×8129 per landscape
- **Compression**: RLE compression for height data
- **Streaming**: Built-in world partitioning and streaming

### Key Features
- **Material Layers**: Weight-blended material layers
- **Virtual Texturing**: Runtime Virtual Texturing (RVT) for large terrains
- **Nanite**: Can use Nanite for high-detail geometry (UE5)
- **Landscape Splines**: Roads/rivers as splines that modify terrain

### Advantages
- **Large Worlds**: Better support for massive terrains
- **Streaming**: Automatic level streaming
- **Performance**: Highly optimized rendering pipeline

---

## Frostbite Engine (Battlefield series)

### Architecture
- **Hybrid System**: Combines heightmaps with mesh decals
- **MegaTexture**: Virtual texturing system
- **Destruction**: Dynamic terrain modification
- **Streaming**: Advanced streaming system

### Heightmap Storage
- **Format**: 16-bit with compression
- **Resolution**: Variable, supports very large terrains
- **Compression**: Custom compression algorithms
- **Streaming**: Granular streaming of terrain chunks

### Key Features
- **Destruction**: Real-time terrain deformation
- **MegaTexture**: Single large texture for entire terrain
- **LOD**: Sophisticated LOD system with smooth transitions
- **Physics Integration**: Heightmap used for physics queries

### Innovations
- **Virtual Texturing**: Pioneered virtual texturing approach
- **Streaming**: Advanced streaming for seamless large worlds
- **Destruction**: Real-time terrain modification

---

## Lumberyard (Amazon)

### Architecture
- **Component-Based**: Similar to Unity/Unreal
- **Terrain System**: Heightmap-based with material layers
- **Vegetation**: Separate vegetation system
- **Streaming**: Built-in streaming support

### Heightmap Storage
- **Format**: 16-bit heightmaps
- **Resolution**: Configurable, supports large terrains
- **Compression**: Standard compression
- **Streaming**: Sector-based streaming

### Key Features
- **Material Layers**: Weight-painted material layers
- **Vegetation**: Instanced vegetation system
- **LOD**: Automatic LOD generation
- **Cloud Integration**: AWS integration for asset streaming

---

## CryEngine

### Architecture
- **Terrain System**: Heightmap-based terrain
- **Material System**: Layered materials
- **Vegetation**: Instanced vegetation
- **LOD**: Automatic LOD system

### Heightmap Storage
- **Format**: 16-bit heightmaps
- **Resolution**: High resolution support
- **Compression**: Standard compression
- **Streaming**: Sector-based streaming

### Key Features
- **High Quality**: Focus on visual quality
- **Large Worlds**: Good support for large terrains
- **Material System**: Advanced material blending

---

## Comparison Table

| Feature | Unity | Unreal | Frostbite | Lumberyard | CryEngine |
|---------|-------|--------|-----------|------------|-----------|
| **Max Resolution** | 4097×4097 | 8129×8129 | Variable | Variable | High |
| **Streaming** | Manual | Built-in | Advanced | Built-in | Built-in |
| **LOD System** | Automatic | HLOD | Advanced | Automatic | Automatic |
| **Material Layers** | 4 splatmaps | Material layers | MegaTexture | Material layers | Material layers |
| **Destruction** | ❌ | ⚠️ Limited | ✅ Yes | ❌ | ⚠️ Limited |
| **Virtual Texturing** | ❌ | ✅ RVT | ✅ MegaTexture | ⚠️ | ⚠️ |
| **Memory Efficiency** | ⚠️ Medium | ✅ Good | ✅ Excellent | ✅ Good | ✅ Good |
| **Ease of Use** | ✅ Easy | ⚠️ Complex | ❌ Complex | ⚠️ Medium | ⚠️ Medium |

---

## Storage Format Differences

### Unity
- Normalized floats (0-1)
- Stored in TerrainData asset
- Optional compression
- Per-terrain storage

### Unreal
- 16-bit integers (0-65535)
- RLE compression
- Stored in landscape asset
- Supports multiple landscapes

### Frostbite
- 16-bit with custom compression
- Chunked storage for streaming
- Optimized for large worlds
- Custom format

---

## LOD Strategies

### Unity
- **Mesh LOD**: Automatic mesh simplification
- **Billboard Trees**: Distance-based billboarding
- **Detail Culling**: Remove details at distance
- **Manual**: Requires manual setup for large worlds

### Unreal
- **HLOD**: Hierarchical LOD system
- **Nanite**: Virtualized geometry (UE5)
- **Virtual Texturing**: Texture LOD
- **Automatic**: Built-in streaming and LOD

### Frostbite
- **Advanced LOD**: Sophisticated LOD transitions
- **Streaming LOD**: LOD integrated with streaming
- **Smooth Transitions**: No popping artifacts
- **Optimized**: Highly optimized for performance

---

## Streaming Approaches

### Unity
- **Manual**: Developer must implement
- **Multiple Terrains**: Use multiple Terrain objects
- **Custom Solutions**: Third-party assets available
- **No Built-in**: No automatic streaming

### Unreal
- **World Partition**: Automatic world streaming
- **Level Streaming**: Built-in level streaming
- **Data Layers**: Organize content for streaming
- **Automatic**: Handles streaming automatically

### Frostbite
- **Granular Streaming**: Stream individual chunks
- **On-Demand**: Load only visible areas
- **Seamless**: No loading screens
- **Advanced**: State-of-the-art streaming

---

## Material/Texture Systems

### Unity Splatmaps
- 4 texture layers
- Height/slope-based blending
- Per-terrain material
- Limited to 4 layers

### Unreal Material Layers
- Unlimited material layers (in theory)
- Weight painting
- Runtime Virtual Texturing
- More flexible

### Frostbite MegaTexture
- Single large texture
- Virtual texturing
- Seamless across entire world
- Very efficient

---

## Performance Characteristics

### Unity
- **CPU**: Moderate overhead
- **GPU**: Good performance
- **Memory**: Can be high for large terrains
- **Optimization**: Requires manual optimization

### Unreal
- **CPU**: Low overhead
- **GPU**: Excellent performance
- **Memory**: Efficient with streaming
- **Optimization**: Highly optimized

### Frostbite
- **CPU**: Very low overhead
- **GPU**: Excellent performance
- **Memory**: Very efficient
- **Optimization**: Extremely optimized

---

## Use Case Recommendations

### Choose Unity If:
- Small to medium terrains
- Rapid prototyping
- Simpler requirements
- Team familiar with Unity

### Choose Unreal If:
- Large open worlds
- Need built-in streaming
- Want advanced features
- Team can handle complexity

### Frostbite (Not Available)
- AAA game development
- Massive open worlds
- Need destruction
- Have Frostbite license

---

## Exercises for Self-Learning

1. **Compare Systems**: Research and compare two engines in detail
2. **Format Analysis**: Analyze heightmap file formats from different engines
3. **Performance Test**: Compare terrain rendering performance (if possible)
4. **Feature Research**: Find examples of games using each engine's terrain system
5. **Migration Study**: Research migrating terrain between engines

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math-matrixes-manifolds.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Unity Terrain documentation
- Unreal Landscape documentation
- Frostbite technical papers
- Game engine comparison articles
