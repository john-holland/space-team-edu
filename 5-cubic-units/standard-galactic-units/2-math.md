# Standard Galactic Units: Mathematics

## Overview
Mathematical foundations for the Standard Galactic Units (SGU) file format, including data encoding, compression mathematics, checksums, and format validation.

## Learning Objectives
- Master SGU file format structure
- Understand encoding and compression mathematics
- Learn checksum and validation calculations
- Calculate file sizes and performance metrics

---

## SGU File Structure

### File Header
SGU file starts with header:

**Header Size**: Fixed 64 bytes

**Header Fields**:
- **Magic Number**: 4 bytes (`'SGU '`)
- **Version**: 2 bytes (format version)
- **Flags**: 2 bytes (compression, endianness, etc.)
- **Width**: 4 bytes (voxel grid width)
- **Height**: 4 bytes (voxel grid height)
- **Depth**: 4 bytes (voxel grid depth)
- **Voxel Size**: 4 bytes (size of each voxel in bytes)
- **Data Offset**: 4 bytes (offset to voxel data)
- **Metadata Size**: 4 bytes (size of metadata section)
- **Checksum**: 4 bytes (CRC32 of header)
- **Reserved**: 28 bytes (for future use)

### Header Validation
Check magic number:
$$\text{magic} = \text{read\_uint32}(\text{bytes}[0:4])$$
$$\text{valid} = (\text{magic} == \text{'SGU '})$$

---

## Voxel Data Encoding

### Dense Encoding
Store all voxels sequentially:

**Data Size**:
$$S_{\text{dense}} = w \times h \times d \times b$$

Where:
- $w, h, d$ = dimensions
- $b$ = bytes per voxel

**Index Calculation**:
For voxel at $(i, j, k)$:
$$\text{index} = i + j \times w + k \times w \times h$$

**Memory Address**:
$$\text{address} = \text{data\_offset} + \text{index} \times b$$

### Sparse Encoding
Store only non-empty voxels:

**Entry Format**:
- **Coordinates**: 3 × 4 bytes = 12 bytes
- **Value**: $b$ bytes

**Data Size**:
$$S_{\text{sparse}} = n \times (12 + b)$$

Where $n$ = number of non-empty voxels.

**Compression Ratio**:
$$C = \frac{S_{\text{dense}}}{S_{\text{sparse}}} = \frac{w \times h \times d \times b}{n \times (12 + b)}$$

---

## Compression Mathematics

### Run-Length Encoding (RLE)

#### Encoding
Store runs of same value:
$$(v, \text{count})$$

Where:
- $v$ = voxel value
- $\text{count}$ = number of consecutive voxels

#### Compression Ratio
For $r$ runs:
$$S_{\text{RLE}} = r \times (b + 4)$$

Where 4 bytes for count.

**Efficiency**:
$$E = \frac{S_{\text{original}}}{S_{\text{RLE}}} = \frac{w \times h \times d \times b}{r \times (b + 4)}$$

Best case: $E = w \times h \times d$ (all same value)
Worst case: $E = \frac{b}{b + 4}$ (no runs)

### LZ4 Compression
LZ4 uses block-based compression:

**Block Size**: Typically 64 KB

**Compression Ratio**: Typically 50-70% of original

**Mathematical Model**:
$$S_{\text{LZ4}} \approx S_{\text{original}} \times C_{\text{ratio}}$$

Where $C_{\text{ratio}} \approx 0.5$ to $0.7$.

### Zlib Compression
Zlib uses DEFLATE algorithm:

**Compression Ratio**: Typically 30-50% of original

**Mathematical Model**:
$$S_{\text{zlib}} \approx S_{\text{original}} \times C_{\text{ratio}}$$

Where $C_{\text{ratio}} \approx 0.3$ to $0.5$.

---

## Checksum Calculation

### CRC32 Checksum
Cyclic Redundancy Check for error detection:

**Polynomial**: $x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1$

**Calculation**:
$$\text{CRC32}(data) = \text{remainder}(data \cdot x^{32} / \text{polynomial})$$

**Validation**:
$$\text{valid} = (\text{CRC32}(\text{header}) == \text{stored\_checksum})$$

### Data Integrity
Calculate checksum of voxel data:
$$\text{data\_checksum} = \text{CRC32}(\text{voxel\_data})$$

Store in metadata for validation.

---

## Chunked Storage

### Chunk Division
Divide large voxel grid into chunks:

**Chunk Size**: $C_w \times C_h \times C_d$

**Number of Chunks**:
$$N_{\text{chunks}} = \left\lceil \frac{w}{C_w} \right\rceil \times \left\lceil \frac{h}{C_h} \right\rceil \times \left\lceil \frac{d}{C_d} \right\rceil$$

### Chunk Index
For chunk at $(c_i, c_j, c_k)$:
$$\text{chunk\_index} = c_i + c_j \times \left\lceil \frac{w}{C_w} \right\rceil + c_k \times \left\lceil \frac{w}{C_w} \right\rceil \times \left\lceil \frac{h}{C_h} \right\rceil$$

### Chunk Offset
Offset to chunk data:
$$\text{chunk\_offset} = \text{data\_offset} + \text{chunk\_index} \times S_{\text{chunk}}$$

Where $S_{\text{chunk}}$ is size of chunk (may vary if compressed).

---

## Metadata Encoding

### Metadata Structure
Variable-length metadata section:

**Format**: Key-value pairs or structured data

**Size**: Stored in header (`metadata_size`)

### Common Metadata Fields
- **Author**: String
- **Creation Date**: Timestamp
- **Tool Version**: String
- **Custom Data**: Binary blob

### Metadata Parsing
Parse metadata based on format:
- **JSON**: Human-readable, larger size
- **Binary**: Compact, faster parsing
- **Custom**: Application-specific

---

## Endianness Handling

### Byte Order
SGU supports both little-endian and big-endian:

**Flag Bit**: Stored in header flags

**Conversion**:
If endianness mismatch:
$$\text{value} = \text{swap\_bytes}(\text{read\_value})$$

### Multi-byte Values
For 32-bit integer:
- **Little-endian**: LSB first
- **Big-endian**: MSB first

**Detection**:
Check magic number byte order.

---

## File Size Calculations

### Uncompressed Dense
$$S_{\text{file}} = S_{\text{header}} + S_{\text{metadata}} + S_{\text{dense}}$$

Where:
- $S_{\text{header}} = 64$ bytes
- $S_{\text{metadata}} = \text{metadata\_size}$
- $S_{\text{dense}} = w \times h \times d \times b$

### Compressed Dense
$$S_{\text{file}} = S_{\text{header}} + S_{\text{metadata}} + S_{\text{compressed}}$$

Where:
$$S_{\text{compressed}} = S_{\text{dense}} \times C_{\text{ratio}}$$

### Sparse
$$S_{\text{file}} = S_{\text{header}} + S_{\text{metadata}} + S_{\text{sparse}}$$

Where:
$$S_{\text{sparse}} = n \times (12 + b)$$

---

## Performance Metrics

### Loading Time
**Uncompressed**:
$$T_{\text{load}} = \frac{S_{\text{file}}}{B_{\text{read}}}$$

Where $B_{\text{read}}$ is read bandwidth.

**Compressed**:
$$T_{\text{load}} = \frac{S_{\text{file}}}{B_{\text{read}}} + \frac{S_{\text{decompressed}}}{B_{\text{decompress}}}$$

Where $B_{\text{decompress}}$ is decompression bandwidth.

### Memory Usage
**During Load**:
$$M_{\text{load}} = S_{\text{file}} + S_{\text{decompressed}}$$

**After Load**:
$$M_{\text{final}} = \text{size of target format}$$

---

## Validation Mathematics

### Dimension Validation
Check dimensions are valid:
$$w > 0 \land h > 0 \land d > 0$$
$$w \leq w_{\max} \land h \leq h_{\max} \land d \leq d_{\max}$$

### Size Validation
Check file size matches header:
$$S_{\text{actual}} \geq S_{\text{header}} + S_{\text{metadata}} + S_{\text{data}}$$

### Checksum Validation
Verify data integrity:
$$\text{CRC32}(\text{data}) == \text{stored\_checksum}$$

---

## Exercises for Self-Learning

1. **Header Parsing**: Parse SGU header and validate
2. **Data Encoding**: Implement dense and sparse encoding
3. **Compression**: Compare RLE, LZ4, zlib compression ratios
4. **Checksum**: Calculate and verify CRC32 checksums
5. **Chunking**: Implement chunked storage and loading

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Format**: See format specification (3-standard-data-files-format.md)

---

## References
- CRC32 algorithm
- Compression algorithms (RLE, LZ4, zlib)
- File format design
- Binary data encoding
