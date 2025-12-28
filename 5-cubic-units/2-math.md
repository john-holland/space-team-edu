# Cubic Units: Mathematics

## Overview
Mathematical foundations for voxel/cubic unit systems, including coordinate transformations, storage mathematics, and geometric operations.

## Learning Objectives
- Master voxel coordinate systems
- Understand storage and compression mathematics
- Learn geometric operations on voxels
- Calculate memory requirements and optimizations

---

## Voxel Coordinate Systems

### 3D Grid Coordinates
A voxel grid is a 3D array where each voxel has integer coordinates:

$$\text{Voxel} = (i, j, k) \quad \text{where } i, j, k \in \mathbb{Z}$$

### World Space to Voxel Space
Convert world coordinates to voxel coordinates:

$$i = \left\lfloor \frac{x - x_{\min}}{v_{\text{size}}} \right\rfloor$$
$$j = \left\lfloor \frac{y - y_{\min}}{v_{\text{size}}} \right\rfloor$$
$$k = \left\lfloor \frac{z - z_{\min}}{v_{\text{size}}} \right\rfloor$$

Where:
- $(x, y, z)$ = world coordinates
- $(x_{\min}, y_{\min}, z_{\min})$ = world space origin
- $v_{\text{size}}$ = voxel size (edge length)

### Voxel Space to World Space
Convert voxel coordinates to world coordinates:

$$x = i \cdot v_{\text{size}} + x_{\min} + \frac{v_{\text{size}}}{2}$$
$$y = j \cdot v_{\text{size}} + y_{\min} + \frac{v_{\text{size}}}{2}$$
$$z = k \cdot v_{\text{size}} + z_{\min} + \frac{v_{\text{size}}}{2}$$

---

## Voxel Storage Mathematics

### Dense Storage
For a voxel grid of size $w \times h \times d$:

**Memory (bytes per voxel = $b$):**
$$M_{\text{dense}} = w \times h \times d \times b$$

**Example**: 256³ grid with 1 byte per voxel:
$$M = 256^3 \times 1 = 16,777,216 \text{ bytes} = 16 \text{ MB}$$

### Sparse Storage
Only store non-empty voxels:

**Memory:**
$$M_{\text{sparse}} = n \times (b + 12)$$

Where:
- $n$ = number of non-empty voxels
- $b$ = bytes per voxel value
- $12$ = 3 integers × 4 bytes (coordinates)

**Compression Ratio:**
$$C = \frac{M_{\text{dense}}}{M_{\text{sparse}}} = \frac{w \times h \times d \times b}{n \times (b + 12)}$$

---

## Sparse Voxel Octree (SVO)

### Octree Structure
An octree subdivides 3D space into 8 octants:

**Depth $d$:**
- **Level 0**: 1 node (root)
- **Level 1**: Up to 8 nodes
- **Level 2**: Up to 64 nodes
- **Level $d$**: Up to $8^d$ nodes

### Maximum Nodes
For depth $d$:
$$N_{\max} = \sum_{i=0}^{d} 8^i = \frac{8^{d+1} - 1}{7}$$

### Memory per Node
Each node stores:
- **Child pointers**: 8 pointers = 64 bytes (64-bit)
- **Data**: Varies (1-4 bytes typical)
- **Flags**: 1 byte (leaf, empty, etc.)

**Total per node**: ~70 bytes

### Memory for SVO
$$M_{\text{SVO}} = N \times 70$$

Where $N$ is actual number of nodes (much less than $N_{\max}$ for sparse data).

---

## Voxel Operations

### Neighbor Access
6-connected neighbors (face-adjacent):
- $(i+1, j, k)$, $(i-1, j, k)$
- $(i, j+1, k)$, $(i, j-1, k)$
- $(i, j, k+1)$, $(i, j, k-1)$

26-connected neighbors (includes edges and corners):
- 6 face-adjacent
- 12 edge-adjacent
- 8 corner-adjacent

### Distance Calculations

#### Manhattan Distance
$$d_{\text{Manhattan}} = |i_2 - i_1| + |j_2 - j_1| + |k_2 - k_1|$$

#### Euclidean Distance
$$d_{\text{Euclidean}} = \sqrt{(i_2 - i_1)^2 + (j_2 - j_1)^2 + (k_2 - k_1)^2}$$

#### Chebyshev Distance
$$d_{\text{Chebyshev}} = \max(|i_2 - i_1|, |j_2 - j_1|, |k_2 - k_1|)$$

---

## Voxel Raycasting

### Ray-Voxel Intersection
Ray equation:
$$\mathbf{r}(t) = \mathbf{o} + t \mathbf{d}$$

Where:
- $\mathbf{o}$ = ray origin
- $\mathbf{d}$ = ray direction (normalized)
- $t$ = parameter

### DDA Algorithm (Digital Differential Analyzer)
Step sizes:
$$\Delta t_x = \frac{v_{\text{size}}}{d_x}, \quad \Delta t_y = \frac{v_{\text{size}}}{d_y}, \quad \Delta t_z = \frac{v_{\text{size}}}{d_z}$$

Initial $t$ values:
$$t_x = \frac{\text{next\_voxel\_x} - o_x}{d_x}$$
$$t_y = \frac{\text{next\_voxel\_y} - o_y}{d_y}$$
$$t_z = \frac{\text{next\_voxel\_z} - o_z}{d_z}$$

Step to next voxel:
- Find minimum of $t_x$, $t_y$, $t_z$
- Step in that direction
- Update corresponding $t$ value

---

## Mesh Extraction (Marching Cubes)

### Scalar Field
Voxel values define scalar field:
$$f(i, j, k) = \text{voxel value at } (i, j, k)$$

### Isosurface
Surface where $f(x, y, z) = \text{threshold}$.

### Marching Cubes
For each voxel cube (8 corners):
1. Check if each corner is above/below threshold
2. Look up configuration (256 possible)
3. Generate triangles for that configuration

**Triangles per voxel**: 0-5 triangles typical

**Total triangles**:
$$T \approx 3 \times n_{\text{surface\_voxels}}$$

---

## Compression Mathematics

### Run-Length Encoding (RLE)
Store runs of same value:

**Compression ratio:**
$$C = \frac{\text{original size}}{\text{compressed size}} = \frac{n \times b}{r \times (b + 4)}$$

Where:
- $n$ = number of voxels
- $b$ = bytes per voxel
- $r$ = number of runs

### Block Compression
Divide into blocks, compress each:

**Block size**: $B \times B \times B$

**Blocks**: $\frac{w}{B} \times \frac{h}{B} \times \frac{d}{B}$

**Memory**: Depends on compression algorithm per block.

---

## Voxel Queries

### Point Query
Check if point $(x, y, z)$ is inside voxel:

$$(i, j, k) = \text{WorldToVoxel}(x, y, z)$$
$$\text{result} = \text{voxel}[i][j][k]$$

**Complexity**: $O(1)$ for dense, $O(\log n)$ for octree

### Range Query
Find all voxels in box $[x_1, x_2] \times [y_1, y_2] \times [z_1, z_2]$:

**Voxel bounds:**
$$i_{\min} = \left\lfloor \frac{x_1 - x_{\min}}{v_{\text{size}}} \right\rfloor$$
$$i_{\max} = \left\lfloor \frac{x_2 - x_{\min}}{v_{\text{size}}} \right\rfloor$$

Similar for $j$ and $k$.

**Result count:**
$$N = (i_{\max} - i_{\min} + 1) \times (j_{\max} - j_{\min} + 1) \times (k_{\max} - k_{\min} + 1)$$

---

## Volume Calculations

### Volume of Voxel
$$V_{\text{voxel}} = v_{\text{size}}^3$$

### Total Volume
For $n$ filled voxels:
$$V_{\text{total}} = n \times v_{\text{size}}^3$$

### Surface Area
Approximate surface area from mesh:
$$A \approx \sum_{i=1}^{T} A_{\text{triangle}, i}$$

Where $T$ is number of triangles.

---

## Spatial Hashing

### Hash Function
Hash voxel coordinates to index:

$$h(i, j, k) = (i \cdot p_1 + j \cdot p_2 + k \cdot p_3) \bmod m$$

Where:
- $p_1, p_2, p_3$ = prime numbers
- $m$ = hash table size

### Collision Handling
Use chaining or open addressing.

**Load factor:**
$$\lambda = \frac{n}{m}$$

Optimal: $\lambda \approx 0.7$

---

## Exercises for Self-Learning

1. **Coordinate Conversion**: Implement world-to-voxel and voxel-to-world
2. **Memory Calculation**: Calculate memory for different voxel formats
3. **Raycasting**: Implement DDA ray-voxel intersection
4. **Neighbor Access**: Implement 6-connected and 26-connected neighbors
5. **Compression**: Compare RLE vs. sparse storage

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation-data-file-import.md)

---

## References
- Voxel algorithms and data structures
- Octree mathematics
- Ray-voxel intersection algorithms
- Compression algorithms
