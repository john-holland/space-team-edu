# Standard Galactic Units: Data File Format Specification

## Overview
Complete specification for the Standard Galactic Units (SGU) file format, including binary structure, encoding details, and implementation guidelines.

## Learning Objectives
- Understand SGU file format structure
- Learn binary encoding details
- Master format versioning and compatibility
- Implement SGU reader/writer

---

## File Format Overview

### File Structure
```
[SGU File]
├── Header (64 bytes)
├── Metadata Section (variable length)
└── Voxel Data Section (variable length)
```

### File Extension
- **Primary**: `.sgu`
- **Alternative**: `.sgv` (SGU Voxel)

### Magic Number
- **Value**: `0x20554753` (ASCII: "SGU " with space)
- **Endianness**: Little-endian by default
- **Location**: Bytes 0-3

---

## Header Structure

### Header Layout (64 bytes total)

| Offset | Size | Field Name | Type | Description |
|--------|------|------------|------|-------------|
| 0x00 | 4 | magic | uint32 | Magic number: `0x20554753` |
| 0x04 | 2 | version | uint16 | Format version (current: 1) |
| 0x06 | 2 | flags | uint16 | Format flags (see below) |
| 0x08 | 4 | width | uint32 | Voxel grid width |
| 0x0C | 4 | height | uint32 | Voxel grid height |
| 0x10 | 4 | depth | uint32 | Voxel grid depth |
| 0x14 | 4 | voxelSize | uint32 | Bytes per voxel (1, 2, or 4) |
| 0x18 | 4 | dataOffset | uint32 | Byte offset to voxel data |
| 0x1C | 4 | metadataSize | uint32 | Size of metadata section |
| 0x20 | 4 | checksum | uint32 | CRC32 of header (bytes 0-0x1F) |
| 0x24 | 28 | reserved | bytes | Reserved for future use |

### Format Flags (uint16)

| Bit | Name | Description |
|-----|------|-------------|
| 0 | compressed | Data is compressed |
| 1 | sparse | Sparse encoding (not dense) |
| 2 | bigEndian | Big-endian byte order |
| 3 | hasMetadata | Metadata section present |
| 4-15 | reserved | Reserved for future use |

### Compression Types (if compressed flag set)

Stored in first byte of compressed data:
- `0x00`: No compression (shouldn't have compressed flag)
- `0x01`: RLE (Run-Length Encoding)
- `0x02`: LZ4
- `0x03`: Zlib/DEFLATE
- `0x04-0xFF`: Reserved

---

## Voxel Data Encoding

### Dense Encoding

Voxels stored sequentially in row-major order:

```
For each z from 0 to depth-1:
    For each y from 0 to height-1:
        For each x from 0 to width-1:
            Write voxel[x][y][z] (voxelSize bytes)
```

**Index Calculation**:
```
index = x + y * width + z * width * height
byteOffset = index * voxelSize
```

### Sparse Encoding

Only non-empty voxels stored:

**Entry Format**:
```
[Coordinate] (12 bytes)
    uint32 x
    uint32 y
    uint32 z
[Value] (voxelSize bytes)
    voxel value
```

**Structure**:
```
[Entry Count] (4 bytes, uint32)
[Entry 1]
[Entry 2]
...
[Entry N]
```

### Run-Length Encoding (RLE)

For compressed dense data:

**Run Format**:
```
[Value] (voxelSize bytes)
[Count] (4 bytes, uint32)
```

**Structure**:
```
[Compression Type] (1 byte, 0x01)
[Run 1]
[Run 2]
...
[Run N]
```

---

## Metadata Section

### Metadata Format

**Structure**:
```
[Metadata Size] (4 bytes, uint32)
[Metadata Entries]
```

### Metadata Entry Format

**Entry Header**:
```
[Key Length] (2 bytes, uint16)
[Value Type] (1 byte)
[Value Length] (4 bytes, uint32)
```

**Value Types**:
- `0x01`: String (UTF-8)
- `0x02`: Integer (int32)
- `0x03`: Float (float32)
- `0x04`: Binary blob
- `0x05`: Boolean (1 byte)

**Entry Data**:
```
[Key] (Key Length bytes, UTF-8 string, null-terminated)
[Value] (Value Length bytes)
```

### Standard Metadata Keys

- `"author"`: String - Author name
- `"created"`: String - ISO 8601 timestamp
- `"tool"`: String - Tool that created file
- `"toolVersion"`: String - Tool version
- `"description"`: String - File description
- `"units"`: String - Unit system ("meters", "voxels", etc.)
- `"voxelSize"`: Float - Physical size of voxel in world units

---

## Checksum Calculation

### CRC32 Algorithm

**Polynomial**: `0xEDB88320` (reversed) or `0x04C11DB7` (normal)

**Implementation**:
```csharp
uint32 crc32_table[256];

void init_crc32_table() {
    for (uint32 i = 0; i < 256; i++) {
        uint32 crc = i;
        for (int j = 0; j < 8; j++) {
            crc = (crc >> 1) ^ ((crc & 1) ? 0xEDB88320 : 0);
        }
        crc32_table[i] = crc;
    }
}

uint32 calculate_crc32(byte* data, size_t length) {
    uint32 crc = 0xFFFFFFFF;
    for (size_t i = 0; i < length; i++) {
        crc = (crc >> 8) ^ crc32_table[(crc ^ data[i]) & 0xFF];
    }
    return crc ^ 0xFFFFFFFF;
}
```

### Checksum Usage

- **Header Checksum**: Calculated over bytes 0-0x1F (excluding checksum field itself)
- **Data Checksum**: Optional, stored in metadata as `"dataChecksum"`

---

## Version History

### Version 1 (Current)

**Features**:
- Dense and sparse encoding
- RLE, LZ4, Zlib compression
- Metadata support
- CRC32 checksums

**Limitations**:
- Maximum dimensions: 2^32 - 1
- Single voxel type per file

### Future Versions

**Version 2 (Planned)**:
- Multi-voxel types
- Extended metadata
- Chunked storage
- Streaming support

---

## Endianness Handling

### Default Endianness
- **Little-endian** by default
- Big-endian indicated by flag bit 2

### Byte Order Conversion

For big-endian files on little-endian systems:
```csharp
uint32 ReadUInt32BE(byte[] data, int offset)
{
    return (uint32)(data[offset] << 24) |
                   (data[offset + 1] << 16) |
                   (data[offset + 2] << 8) |
                   data[offset + 3];
}
```

---

## File Validation

### Validation Steps

1. **Magic Number Check**: Verify `magic == 0x20554753`
2. **Version Check**: Verify version is supported
3. **Dimension Check**: Verify width, height, depth > 0
4. **Size Check**: Verify file size >= header + metadata + data
5. **Checksum Check**: Verify header checksum
6. **Data Validation**: Verify data section is valid

### Error Codes

- `0x01`: Invalid magic number
- `0x02`: Unsupported version
- `0x03`: Invalid dimensions
- `0x04`: File too small
- `0x05`: Checksum mismatch
- `0x06`: Invalid compression
- `0x07`: Corrupted data

---

## Example File Structure

### Small Dense File (8×8×8, uncompressed)

```
[Header: 64 bytes]
    magic: 0x20554753
    version: 1
    flags: 0x0000 (no compression, dense, little-endian)
    width: 8
    height: 8
    depth: 8
    voxelSize: 1
    dataOffset: 64
    metadataSize: 0
    checksum: [CRC32 of header]
    reserved: [28 zero bytes]

[Voxel Data: 512 bytes]
    512 voxel values (1 byte each)
```

### Sparse File Example

```
[Header: 64 bytes]
    flags: 0x0002 (sparse encoding)
    ...
    
[Voxel Data]
    [Entry Count: 4 bytes] = 100
    [Entry 1: 13 bytes]
        x: 5, y: 3, z: 7
        value: 255
    [Entry 2: 13 bytes]
        ...
    [100 entries total]
```

---

## Implementation Guidelines

### Reading SGU Files

1. Read and validate header
2. Check version compatibility
3. Read metadata (if present)
4. Read voxel data based on encoding
5. Decompress if needed
6. Validate checksums

### Writing SGU Files

1. Prepare voxel data
2. Choose encoding (dense/sparse)
3. Optionally compress
4. Calculate checksums
5. Write header
6. Write metadata
7. Write voxel data

### Performance Considerations

- **Streaming**: Read/write in chunks for large files
- **Memory**: Use sparse encoding for mostly-empty data
- **Compression**: LZ4 for speed, Zlib for size
- **Caching**: Cache decompressed data if frequently accessed

---

## Compatibility Notes

### Cross-Platform

- **Windows**: Little-endian (default)
- **Linux**: Little-endian (default)
- **macOS**: Little-endian (default)
- **Big-endian systems**: Use big-endian flag

### Tool Compatibility

- **Version 1**: All tools should support
- **Future versions**: Tools should handle gracefully (skip unknown features)

---

## Exercises for Self-Learning

1. **File Parser**: Implement SGU file reader
2. **File Writer**: Implement SGU file writer
3. **Validation**: Create file validation tool
4. **Conversion**: Convert between dense/sparse formats
5. **Compression**: Compare compression algorithms

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand format mathematics (2-math.md)

---

## References
- Binary file format design
- CRC32 algorithm
- Compression algorithms
- File format specifications
