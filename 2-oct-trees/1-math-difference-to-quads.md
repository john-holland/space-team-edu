# Oct Trees: Mathematical Differences from Quad Trees

## Overview
Oct trees extend quad tree concepts to 3D space. This document focuses on the mathematical differences and extensions needed for three-dimensional spatial partitioning.

## Learning Objectives
- Understand how 3D subdivision differs from 2D
- Learn octant mathematics and coordinate calculations
- Master 3D query operations (point, range, nearest neighbor)
- Calculate 3D bounding volumes and intersections

---

## Dimensional Extension: 2D → 3D

### Quad Tree (2D)
- **Space**: 2D plane $(x, y)$
- **Subdivision**: 4 quadrants
- **Bounding Volume**: Rectangle (2D)

### Oct Tree (3D)
- **Space**: 3D volume $(x, y, z)$
- **Subdivision**: 8 octants
- **Bounding Volume**: Axis-Aligned Bounding Box (AABB) (3D)

---

## 3D Bounding Box Mathematics

### AABB Representation
An oct tree node represents a 3D axis-aligned bounding box:

$$\text{Node} = \{(x, y, z) : x_0 \leq x \leq x_1, y_0 \leq y \leq y_1, z_0 \leq z \leq z_1\}$$

Where:
- $(x_0, y_0, z_0)$ = minimum corner (back-bottom-left)
- $(x_1, y_1, z_1)$ = maximum corner (front-top-right)
- Width: $w = x_1 - x_0$
- Height: $h = y_1 - y_0$
- Depth: $d = z_1 - z_0$

### Center Point Calculation
The center of a node's bounding box:

$$c_x = \frac{x_0 + x_1}{2}, \quad c_y = \frac{y_0 + y_1}{2}, \quad c_z = \frac{z_0 + z_1}{2}$$

### Volume Calculation
Volume of a bounding box:

$$V = w \times h \times d = (x_1 - x_0)(y_1 - y_0)(z_1 - z_0)$$

---

## Octant Subdivision

### Eight Octants
A node is divided into eight equal octants:

**Octant 0 (Back-Bottom-Left):**
$$O_0 = \{(x, y, z) : x_0 \leq x < c_x, y_0 \leq y < c_y, z_0 \leq z < c_z\}$$

**Octant 1 (Back-Bottom-Right):**
$$O_1 = \{(x, y, z) : c_x \leq x \leq x_1, y_0 \leq y < c_y, z_0 \leq z < c_z\}$$

**Octant 2 (Back-Top-Left):**
$$O_2 = \{(x, y, z) : x_0 \leq x < c_x, c_y \leq y \leq y_1, z_0 \leq z < c_z\}$$

**Octant 3 (Back-Top-Right):**
$$O_3 = \{(x, y, z) : c_x \leq x \leq x_1, c_y \leq y \leq y_1, z_0 \leq z < c_z\}$$

**Octant 4 (Front-Bottom-Left):**
$$O_4 = \{(x, y, z) : x_0 \leq x < c_x, y_0 \leq y < c_y, c_z \leq z \leq z_1\}$$

**Octant 5 (Front-Bottom-Right):**
$$O_5 = \{(x, y, z) : c_x \leq x \leq x_1, y_0 \leq y < c_y, c_z \leq z \leq z_1\}$$

**Octant 6 (Front-Top-Left):**
$$O_6 = \{(x, y, z) : x_0 \leq x < c_x, c_y \leq y \leq y_1, c_z \leq z \leq z_1\}$$

**Octant 7 (Front-Top-Right):**
$$O_7 = \{(x, y, z) : c_x \leq x \leq x_1, c_y \leq y \leq y_1, c_z \leq z \leq z_1\}$$

### Octant Size After Subdivision
After $d$ levels of subdivision, each leaf node has size:

$$w_d = \frac{w_0}{2^d}, \quad h_d = \frac{h_0}{2^d}, \quad d_d = \frac{d_0}{2^d}$$

Where $w_0, h_0, d_0$ are the root node dimensions.

---

## Octant Selection Mathematics

### Point-to-Octant Mapping
Determine which octant contains point $(p_x, p_y, p_z)$:

$$\text{octant}(p_x, p_y, p_z) = \begin{cases}
0 & \text{if } p_x < c_x \land p_y < c_y \land p_z < c_z \\
1 & \text{if } p_x \geq c_x \land p_y < c_y \land p_z < c_z \\
2 & \text{if } p_x < c_x \land p_y \geq c_y \land p_z < c_z \\
3 & \text{if } p_x \geq c_x \land p_y \geq c_y \land p_z < c_z \\
4 & \text{if } p_x < c_x \land p_y < c_y \land p_z \geq c_z \\
5 & \text{if } p_x \geq c_x \land p_y < c_y \land p_z \geq c_z \\
6 & \text{if } p_x < c_x \land p_y \geq c_y \land p_z \geq c_z \\
7 & \text{if } p_x \geq c_x \land p_y \geq c_y \land p_z \geq c_z
\end{cases}$$

### Binary Encoding
Octant can be encoded as 3-bit value:
- Bit 0: $x$ axis ($1$ if $p_x \geq c_x$)
- Bit 1: $y$ axis ($1$ if $p_y \geq c_y$)
- Bit 2: $z$ axis ($1$ if $p_z \geq c_z$)

$$\text{octant} = (p_x \geq c_x) + 2(p_y \geq c_y) + 4(p_z \geq c_z)$$

---

## 3D Query Operations

### Point-in-Box Test
Check if point $(p_x, p_y, p_z)$ is within node bounds:

$$\text{contains}(p_x, p_y, p_z) = (x_0 \leq p_x \leq x_1) \land (y_0 \leq p_y \leq y_1) \land (z_0 \leq p_z \leq z_1)$$

### AABB Intersection
Check if query AABB $B_q$ intersects node AABB $B_n$:

$$\text{intersects}(B_q, B_n) = \neg \begin{pmatrix}
(B_q.x_1 < B_n.x_0) \lor (B_q.x_0 > B_n.x_1) \lor \\
(B_q.y_1 < B_n.y_0) \lor (B_q.y_0 > B_n.y_1) \lor \\
(B_q.z_1 < B_n.z_0) \lor (B_q.z_0 > B_n.z_1)
\end{pmatrix}$$

### AABB Containment
Check if query AABB contains node:

$$\text{contains}(B_q, B_n) = \begin{pmatrix}
(B_q.x_0 \leq B_n.x_0) \land (B_q.x_1 \geq B_n.x_1) \land \\
(B_q.y_0 \leq B_n.y_0) \land (B_q.y_1 \geq B_n.y_1) \land \\
(B_q.z_0 \leq B_n.z_0) \land (B_q.z_1 \geq B_n.z_1)
\end{pmatrix}$$

---

## 3D Distance Calculations

### Euclidean Distance (3D)
Distance from point $(p_x, p_y, p_z)$ to point $(q_x, q_y, q_z)$:

$$d = \sqrt{(p_x - q_x)^2 + (p_y - q_y)^2 + (p_z - q_z)^2}$$

### Distance to AABB
Minimum distance from point to node's bounding box:

$$d_{\min} = \begin{cases}
0 & \text{if point inside box} \\
\sqrt{d_x^2 + d_y^2 + d_z^2} & \text{if point outside}
\end{cases}$$

Where:
- $d_x = \max(0, x_0 - p_x, p_x - x_1)$
- $d_y = \max(0, y_0 - p_y, p_y - y_1)$
- $d_z = \max(0, z_0 - p_z, p_z - z_1)$

### Sphere-AABB Intersection
Check if sphere with center $(c_x, c_y, c_z)$ and radius $r$ intersects AABB:

$$d_{\min} \leq r$$

Where $d_{\min}$ is distance from sphere center to AABB.

---

## Complexity Analysis (3D vs 2D)

### Tree Height
For 3D space, average depth increases:

**Quad Tree (2D):**
$$d_{\text{avg}} \approx \log_4(n) = \frac{\log_2(n)}{2}$$

**Oct Tree (3D):**
$$d_{\text{avg}} \approx \log_8(n) = \frac{\log_2(n)}{3}$$

### Node Count
**Quad Tree**: Up to $4^d$ nodes at depth $d$
**Oct Tree**: Up to $8^d$ nodes at depth $d$

### Memory Per Node
**Quad Tree**: 4 child pointers
**Oct Tree**: 8 child pointers (2× memory overhead)

### Query Complexity
Both maintain $O(\log n + k)$ for range queries, but:
- **Oct Tree**: More nodes to traverse (8 children vs 4)
- **Oct Tree**: More complex intersection tests (3D vs 2D)

---

## Volume-Based Subdivision Criteria

### Volume Threshold
Stop subdividing when volume becomes too small:

$$V = w \times h \times d < V_{\min}$$

### Density-Based Subdivision
Subdivide based on object density:

$$\text{density} = \frac{|\text{objects}|}{V} > \rho_{\max}$$

Where $\rho_{\max}$ is maximum allowed density.

### Combined 3D Criteria
$$\text{subdivide} = (d < d_{\max}) \land (|\text{objects}| > n_{\max}) \land (V \geq V_{\min}) \land (\text{density} > \rho_{\max})$$

---

## Spatial Relationships in 3D

### Adjacent Octants
Two octants are adjacent if they share a face:
- **X-adjacent**: Different in $x$ coordinate only
- **Y-adjacent**: Different in $y$ coordinate only
- **Z-adjacent**: Different in $z$ coordinate only

### Octant Neighbors
For octant $o$ with binary encoding $(b_x, b_y, b_z)$:
- **X-neighbor**: Flip $b_x$ bit
- **Y-neighbor**: Flip $b_y$ bit
- **Z-neighbor**: Flip $b_z$ bit

### Face-Sharing Octants
Octants share a face if exactly one bit differs in their encoding.

---

## Performance Considerations

### Memory Overhead
Oct trees use approximately **2× more memory** than quad trees:
- 8 children vs 4 children
- 3D coordinates vs 2D coordinates
- Larger bounding box structures

### Traversal Cost
Each level requires checking **8 children** instead of 4:
- **Quad Tree**: $O(4^d)$ worst-case nodes to check
- **Oct Tree**: $O(8^d)$ worst-case nodes to check

### Cache Performance
Larger node structures may have worse cache locality:
- More data per node
- Larger memory footprint
- More cache misses during traversal

---

## Mathematical Formulations Summary

### Key Differences Table

| Aspect | Quad Tree (2D) | Oct Tree (3D) |
|--------|----------------|---------------|
| **Dimensions** | $(x, y)$ | $(x, y, z)$ |
| **Children** | 4 quadrants | 8 octants |
| **Bounding Volume** | Rectangle | AABB |
| **Subdivision** | $2^2 = 4$ | $2^3 = 8$ |
| **Average Depth** | $\log_4(n)$ | $\log_8(n)$ |
| **Max Nodes at Depth $d$** | $4^d$ | $8^d$ |
| **Memory per Node** | $O(1)$ | $O(1)$ but 2× larger |
| **Distance Formula** | $\sqrt{dx^2 + dy^2}$ | $\sqrt{dx^2 + dy^2 + dz^2}$ |

---

## Exercises for Self-Learning

1. **Octant Calculation**: Given a point $(5, 3, 7)$ and center $(4, 4, 6)$, calculate the octant
2. **Volume Calculation**: Calculate volume of AABB from $(0,0,0)$ to $(10, 20, 30)$
3. **Distance to AABB**: Find minimum distance from point $(15, 15, 15)$ to AABB $(0,0,0)$ to $(10,10,10)$
4. **Complexity Comparison**: Compare memory usage of quad tree vs oct tree for 1000 objects
5. **Subdivision Math**: Calculate node sizes at depth 3 for a 100×100×100 root

---

## Next Steps
- **Usage**: Learn practical applications (2-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- 3D Computational Geometry
- Spatial Data Structures (3D extensions)
- AABB intersection algorithms
