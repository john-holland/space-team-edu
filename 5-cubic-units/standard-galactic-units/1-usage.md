# Standard Galactic Units: Usage and Applications

## Overview
Standard Galactic Units (SGU) is a standardized data format for representing voxel/cubic unit data across different systems and applications. This document covers use cases, applications, and when to use SGU format.

## Learning Objectives
- Understand what Standard Galactic Units format is
- Identify use cases and applications
- Learn when to use SGU vs. other formats
- Recognize benefits of standardization

---

## What is Standard Galactic Units?

### Definition
Standard Galactic Units (SGU) is a standardized file format for storing and exchanging voxel/cubic unit data, designed for interoperability across different game engines, tools, and applications.

### Basic Concept
- **Standardized**: Consistent format across platforms
- **Efficient**: Optimized for storage and loading
- **Extensible**: Can be extended with custom data
- **Universal**: Works with any voxel-based system

---

## Common Use Cases

### 1. Cross-Platform Voxel Data
**Problem**: Share voxel data between different engines/tools
- **Use Case**: Export from editor, import to game engine
- **Benefit**: Single format works everywhere
- **Example**: Create voxel model in tool A, use in Unity/Unreal

### 2. Procedural Generation Storage
**Problem**: Store procedurally generated voxel worlds
- **Use Case**: Save generated worlds, load later
- **Benefit**: Efficient storage format
- **Example**: Save Minecraft-like chunks in SGU format

### 3. Asset Pipeline
**Problem**: Convert between different voxel formats
- **Use Case**: Import/export pipeline
- **Benefit**: SGU as intermediate format
- **Example**: Convert from format A → SGU → format B

### 4. Network Transmission
**Problem**: Send voxel data over network
- **Use Case**: Multiplayer games, streaming
- **Benefit**: Compact, efficient format
- **Example**: Stream voxel chunks to clients

### 5. Version Control
**Problem**: Track changes to voxel data
- **Use Case**: Git, version control systems
- **Benefit**: Binary format with good diff properties
- **Example**: Version control voxel world edits

### 6. Archival Storage
**Problem**: Long-term storage of voxel data
- **Use Case**: Backup, archival
- **Benefit**: Standard format ensures future compatibility
- **Example**: Archive voxel worlds for later use

---

## When to Use SGU Format

### ✅ Good Use Cases
- **Cross-platform** data exchange
- **Standardization** needed across tools
- **Efficient storage** requirements
- **Network transmission** of voxel data
- **Long-term compatibility** needed
- **Asset pipeline** integration

### ❌ Poor Use Cases
- **Single-platform** only (proprietary format may be better)
- **Very simple** data (overhead not worth it)
- **Temporary** data (no need for standardization)
- **Proprietary** systems (may prefer custom format)

---

## SGU Format Features

### Core Features
- **Header**: Metadata, version, dimensions
- **Voxel Data**: Efficient storage of voxel values
- **Compression**: Optional compression support
- **Metadata**: Custom metadata support
- **Chunking**: Support for chunked/large worlds

### Data Types Supported
- **Material IDs**: Integer material identifiers
- **Density Values**: Float density values
- **Color Data**: RGB/RGBA color per voxel
- **Custom Data**: Extensible custom data fields

### Compression Options
- **None**: Uncompressed (fastest loading)
- **RLE**: Run-length encoding (good for uniform regions)
- **LZ4**: Fast compression (good balance)
- **Zlib**: Better compression (smaller files)

---

## Comparison with Other Formats

### SGU vs. Raw Binary
| Aspect | SGU | Raw Binary |
|--------|-----|------------|
| **Standardization** | ✅ Yes | ❌ No |
| **Metadata** | ✅ Yes | ❌ No |
| **Compression** | ✅ Yes | ❌ No |
| **Compatibility** | ✅ Cross-platform | ⚠️ Platform-specific |
| **Simplicity** | ⚠️ More complex | ✅ Simple |

### SGU vs. Proprietary Formats
| Aspect | SGU | Proprietary |
|--------|-----|-------------|
| **Compatibility** | ✅ Universal | ❌ Limited |
| **Optimization** | ⚠️ General | ✅ Optimized |
| **Features** | ⚠️ Standard | ✅ Custom |
| **Future-proof** | ✅ Yes | ⚠️ Depends |

---

## Practical Examples

### Example 1: Cross-Engine Workflow
```
Tool: Voxel editor exports to SGU
Unity: Imports SGU, converts to Unity format
Unreal: Imports same SGU, converts to Unreal format
Result: Single source of truth, works everywhere
```

### Example 2: Procedural World Storage
```
Generation: Procedurally generate voxel world
Storage: Save chunks in SGU format
Loading: Load SGU chunks on demand
Result: Efficient storage, fast loading
```

### Example 3: Network Streaming
```
Server: Generates/loads voxel chunks
Format: Converts to SGU for transmission
Client: Receives SGU, converts to local format
Result: Efficient network usage
```

---

## Integration Considerations

### Import Pipeline
1. **Load SGU file**: Read header and data
2. **Decompress**: If compressed, decompress
3. **Validate**: Check format version, dimensions
4. **Convert**: Convert to engine-specific format
5. **Create**: Create voxel representation

### Export Pipeline
1. **Extract**: Extract voxel data from engine
2. **Validate**: Ensure data is valid
3. **Compress**: Optionally compress
4. **Write**: Write SGU file with metadata

### Version Handling
- **Version Check**: Verify SGU version compatibility
- **Migration**: Convert older versions if needed
- **Fallback**: Handle unsupported versions gracefully

---

## Performance Characteristics

### File Size
- **Uncompressed**: $w \times h \times d \times b$ bytes
- **RLE Compressed**: Varies (good for uniform data)
- **LZ4 Compressed**: ~50-70% of original
- **Zlib Compressed**: ~30-50% of original

### Loading Performance
- **Uncompressed**: Fastest (direct read)
- **RLE**: Fast (simple decompression)
- **LZ4**: Very fast (optimized algorithm)
- **Zlib**: Slower (better compression)

### Memory Usage
- **During Load**: Temporary buffer for decompression
- **After Load**: Depends on target format
- **Streaming**: Can load chunks incrementally

---

## Exercises for Self-Learning

1. **Format Research**: Study SGU format specification
2. **Import/Export**: Implement SGU import and export
3. **Compression**: Compare different compression methods
4. **Cross-Platform**: Test SGU files across different tools
5. **Performance**: Benchmark loading performance

---

## Next Steps
- **Math**: Learn SGU format mathematics (2-math.md)
- **Format**: See format specification (3-standard-data-files-format.md)

---

## References and Further Reading
- SGU format specification
- Voxel file format standards
- Compression algorithms
- Cross-platform data formats
