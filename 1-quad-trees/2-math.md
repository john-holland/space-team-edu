# Quad Trees: Mathematical Foundations

## Overview
Understanding the mathematical principles behind quad trees is essential for effective implementation and optimization.

## Learning Objectives
- Master the coordinate system and spatial partitioning mathematics
- Understand subdivision algorithms and termination conditions
- Learn query mathematics (point, range, nearest neighbor)
- Calculate performance bounds and complexity

---

## Coordinate Systems and Bounding Boxes

### 2D Coordinate System
A quad tree operates in a 2D Cartesian coordinate system:

$$\text{Space} = \{(x, y) : x_{\min} \leq x \leq x_{\max}, y_{\min} \leq y \leq y_{\max}\}$$

### Bounding Box Representation
Each node represents a rectangular region:

$$\text{Node} = \{(x, y) : x_0 \leq x \leq x_1, y_0 \leq y \leq y_1\}$$

Where:
- $(x_0, y_0)$ = bottom-left corner
- $(x_1, y_1)$ = top-right corner
- Width: $w = x_1 - x_0$
- Height: $h = y_1 - y_0$

### Center Point Calculation
The center of a node's bounding box:

$$c_x = \frac{x_0 + x_1}{2}, \quad c_y = \frac{y_0 + y_1}{2}$$

---

## Subdivision Mathematics

### Quadrant Division
A node is divided into four equal quadrants:

**Quadrant 0 (Bottom-Left):**
$$Q_0 = \{(x, y) : x_0 \leq x < c_x, y_0 \leq y < c_y\}$$

**Quadrant 1 (Bottom-Right):**
$$Q_1 = \{(x, y) : c_x \leq x \leq x_1, y_0 \leq y < c_y\}$$

**Quadrant 2 (Top-Left):**
$$Q_2 = \{(x, y) : x_0 \leq x < c_x, c_y \leq y \leq y_1\}$$

**Quadrant 3 (Top-Right):**
$$Q_3 = \{(x, y) : c_x \leq x \leq x_1, c_y \leq y \leq y_1\}$$

### Subdivision Size
After $d$ levels of subdivision, each leaf node has size:

$$w_d = \frac{w_0}{2^d}, \quad h_d = \frac{h_0}{2^d}$$

Where $w_0, h_0$ are the root node dimensions.

---

## Subdivision Criteria

### Maximum Depth Limit
Stop subdividing when depth reaches maximum:

$$\text{subdivide} = \begin{cases}
\text{true} & \text{if } d < d_{\max} \\
\text{false} & \text{if } d \geq d_{\max}
\end{cases}$$

### Maximum Objects Per Node
Stop when object count is below threshold:

$$\text{subdivide} = \begin{cases}
\text{true} & \text{if } |\text{objects}| > n_{\max} \\
\text{false} & \text{if } |\text{objects}| \leq n_{\max}
\end{cases}$$

### Minimum Node Size
Stop when node becomes too small:

$$\text{subdivide} = \begin{cases}
\text{true} & \text{if } w \geq w_{\min} \text{ and } h \geq h_{\min} \\
\text{false} & \text{if } w < w_{\min} \text{ or } h < h_{\min}
\end{cases}$$

### Combined Criteria
Most implementations use multiple criteria:

$$\text{subdivide} = (d < d_{\max}) \land (|\text{objects}| > n_{\max}) \land (w \geq w_{\min}) \land (h \geq h_{\min})$$

---

## Point Queries

### Point-in-Node Test
Check if point $(p_x, p_y)$ is within node bounds:

$$\text{contains}(p_x, p_y) = (x_0 \leq p_x \leq x_1) \land (y_0 \leq p_y \leq y_1)$$

### Quadrant Selection
Determine which quadrant contains a point:

$$\text{quadrant}(p_x, p_y) = \begin{cases}
0 & \text{if } p_x < c_x \land p_y < c_y \\
1 & \text{if } p_x \geq c_x \land p_y < c_y \\
2 & \text{if } p_x < c_x \land p_y \geq c_y \\
3 & \text{if } p_x \geq c_x \land p_y \geq c_y
\end{cases}$$

### Point Query Algorithm Complexity
- **Best case**: $O(1)$ - point in root node
- **Average case**: $O(\log n)$ - balanced tree
- **Worst case**: $O(n)$ - degenerate tree (all objects in one path)

---

## Range Queries

### Rectangle Intersection
Check if query rectangle $R_q$ intersects node rectangle $R_n$:

$$\text{intersects}(R_q, R_n) = \neg((R_q.x_1 < R_n.x_0) \lor (R_q.x_0 > R_n.x_1) \lor (R_q.y_1 < R_n.y_0) \lor (R_q.y_0 > R_n.y_1))$$

### Rectangle Containment
Check if query rectangle contains node:

$$\text{contains}(R_q, R_n) = (R_q.x_0 \leq R_n.x_0) \land (R_q.x_1 \geq R_n.x_1) \land (R_q.y_0 \leq R_n.y_0) \land (R_q.y_1 \geq R_n.y_1)$$

### Range Query Complexity
For a range query returning $k$ results:
- **Time**: $O(\log n + k)$
- **Space**: $O(\log n)$ for recursion stack

---

## Nearest Neighbor Queries

### Euclidean Distance
Distance from point $(p_x, p_y)$ to point $(q_x, q_y)$:

$$d = \sqrt{(p_x - q_x)^2 + (p_y - q_y)^2}$$

### Distance to Bounding Box
Minimum distance from point to node's bounding box:

$$d_{\min} = \begin{cases}
0 & \text{if point inside box} \\
\min(d_x, d_y) & \text{if point outside}
\end{cases}$$

Where:
- $d_x = \max(0, x_0 - p_x, p_x - x_1)$
- $d_y = \max(0, y_0 - p_y, p_y - y_1)$

### Nearest Neighbor Algorithm
1. Start with best distance = $\infty$
2. Traverse tree, pruning nodes where $d_{\min} > \text{best distance}$
3. Update best distance when finding closer objects
4. Return closest object

**Complexity**: $O(\log n)$ average case

---

## Tree Height and Depth Analysis

### Maximum Depth
For a space of size $w \times h$ with minimum node size $s_{\min}$:

$$d_{\max} = \left\lceil \log_2\left(\frac{\max(w, h)}{s_{\min}}\right) \right\rceil$$

### Average Depth
For $n$ uniformly distributed points:

$$d_{\text{avg}} \approx \log_4(n) = \frac{\log_2(n)}{2}$$

### Tree Height
Height of a balanced quad tree with $n$ objects:

$$h = O(\log n)$$

---

## Space Complexity

### Node Count
In worst case (degenerate tree):
- **Maximum nodes**: $O(n \cdot d_{\max})$

In best case (balanced tree):
- **Minimum nodes**: $O(n)$

### Memory Per Node
Each node stores:
- Bounding box: 4 floats = 16 bytes
- Object references: $O(k)$ where $k$ is objects per node
- Child pointers: 4 pointers = 32 bytes (64-bit)

**Total per node**: $O(1)$ + object storage

---

## Performance Analysis

### Insertion Complexity
- **Best case**: $O(1)$ - insert into root
- **Average case**: $O(\log n)$ - balanced tree
- **Worst case**: $O(n)$ - all objects in one path

### Query Complexity
- **Point query**: $O(\log n)$ average
- **Range query**: $O(\log n + k)$ where $k$ is results
- **Nearest neighbor**: $O(\log n)$ average

### Rebalancing
If tree becomes unbalanced, rebuild cost:
- **Time**: $O(n \log n)$
- **Space**: $O(n)$ temporary

---

## Optimization Mathematics

### Load Balancing
Optimal objects per node to minimize query time:

$$n_{\text{opt}} = \sqrt{\frac{C_{\text{query}}}{C_{\text{traverse}}}}$$

Where:
- $C_{\text{query}}$ = cost to query objects in node
- $C_{\text{traverse}}$ = cost to traverse to child

### Memory vs. Performance Trade-off
Memory usage vs. query performance:

$$M = O(n \cdot (1 + \alpha \cdot d_{\max}))$$

$$T_{\text{query}} = O(\log_4(n) + \beta \cdot n_{\text{per\_node}})$$

Where $\alpha, \beta$ are tuning parameters.

---

## Exercises for Self-Learning

1. **Calculate Subdivision**: Given a 1000×1000 space, calculate node sizes at depth 5
2. **Range Query Math**: Derive the intersection formula for two rectangles
3. **Distance Calculation**: Implement distance-to-bounding-box calculation
4. **Complexity Analysis**: Prove O(log n) average case for point queries
5. **Optimization**: Find optimal $n_{\max}$ for your use case

---

## Next Steps
- **Usage**: Review practical applications (1-usage.md)
- **Implementation**: See Unity code with these formulas (3-unity-implementation.md)

---

## References
- Computational Geometry algorithms
- Spatial Data Structures textbooks
- Algorithm complexity analysis resources
