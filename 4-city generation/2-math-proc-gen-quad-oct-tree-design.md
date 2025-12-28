# City Generation: Mathematics of Procedural Generation and Quad/Oct Tree Design

## Overview
Mathematical foundations for procedural city generation, including spatial data structures (quad/oct trees), L-systems, agent-based simulation, and geometric algorithms.

## Learning Objectives
- Understand spatial partitioning for city generation
- Learn L-system mathematics for road networks
- Master agent-based simulation math
- Calculate building placement and zoning
- Understand geometric algorithms for city layout

---

## Spatial Data Structures for Cities

### Quad Tree for 2D City Layout
City can be represented in 2D (top-down view):

$$\text{City} = \{(x, z) : \text{contains roads, buildings, zones}\}$$

Quad tree subdivides city space:
- **Level 0**: Entire city
- **Level 1**: 4 districts
- **Level 2**: 16 neighborhoods
- **Level n**: $4^n$ cells

### Oct Tree for 3D City
For full 3D city representation:

$$\text{City3D} = \{(x, y, z) : \text{contains 3D structures}\}$$

Oct tree subdivides 3D space:
- **Level 0**: Entire city volume
- **Level 1**: 8 octants
- **Level 2**: 64 cells
- **Level n**: $8^n$ cells

### Subdivision Criteria for Cities
Subdivide based on:
- **Building density**: $\rho = \frac{|\text{buildings}|}{V} > \rho_{\max}$
- **Complexity**: Subdivide complex areas more
- **LOD**: Different detail levels

---

## L-Systems for Road Networks

### L-System Definition
An L-system is a formal grammar:

$$G = (V, \omega, P)$$

Where:
- $V$ = alphabet (symbols)
- $\omega$ = axiom (initial string)
- $P$ = production rules

### Road Network Grammar
Example grammar for roads:

**Alphabet:**
- `F` = move forward (draw road)
- `+` = turn right (angle $\delta$)
- `-` = turn left (angle $\delta$)
- `[` = push state (branch start)
- `]` = pop state (branch end)

**Axiom:** `F`

**Rules:**
- $F \rightarrow F[+F]F[-F]F$ (branching road)

### Iteration
After $n$ iterations:

$$\omega_n = P^n(\omega_0)$$

Where $P^n$ means apply rules $n$ times.

### Road Length Scaling
Road length decreases with iteration:

$$L_n = L_0 \cdot r^n$$

Where $r \in (0, 1)$ is reduction factor.

### Angle Calculation
Turn angles:

$$\delta_n = \delta_0 \cdot \alpha^n$$

Where $\alpha$ controls angle reduction.

---

## Agent-Based Road Generation

### Agent Movement
Agent position at time $t$:

$$\mathbf{p}(t) = \mathbf{p}_0 + \int_0^t \mathbf{v}(\tau) \, d\tau$$

Where $\mathbf{v}(t)$ is velocity.

### Attraction/Repulsion Forces
Agent is attracted to certain points, repelled from others:

$$\mathbf{F}_{\text{attract}} = k_a \frac{\mathbf{p}_{\text{target}} - \mathbf{p}}{|\mathbf{p}_{\text{target}} - \mathbf{p}|^2}$$

$$\mathbf{F}_{\text{repel}} = -k_r \frac{\mathbf{p} - \mathbf{p}_{\text{obstacle}}}{|\mathbf{p} - \mathbf{p}_{\text{obstacle}}|^2}$$

### Total Force
$$\mathbf{F}_{\text{total}} = \sum_i \mathbf{F}_{\text{attract},i} + \sum_j \mathbf{F}_{\text{repel},j}$$

### Velocity Update
$$\mathbf{v}(t+\Delta t) = \mathbf{v}(t) + \frac{\mathbf{F}_{\text{total}}}{m} \Delta t$$

Where $m$ is agent mass.

### Road Placement
When agent moves, place road segment:

$$\text{Road} = \{\mathbf{p}(t) : t \in [t_0, t_1]\}$$

---

## Voronoi Diagrams for Zoning

### Voronoi Cell
Given seed points $\{s_1, s_2, \ldots, s_n\}$, Voronoi cell:

$$V_i = \{\mathbf{p} : d(\mathbf{p}, s_i) \leq d(\mathbf{p}, s_j) \forall j \neq i\}$$

Where $d$ is Euclidean distance.

### Distance Function
$$d(\mathbf{p}, s_i) = \sqrt{(p_x - s_{i,x})^2 + (p_z - s_{i,z})^2}$$

### Weighted Voronoi
With weights $w_i$:

$$V_i = \{\mathbf{p} : \frac{d(\mathbf{p}, s_i)}{w_i} \leq \frac{d(\mathbf{p}, s_j)}{w_j} \forall j \neq i\}$$

### Zoning Assignment
Assign zone type to each Voronoi cell:

$$\text{Zone}(V_i) = f(s_i, \text{context})$$

Where $f$ is zoning function.

---

## Building Placement Mathematics

### Grid-Based Placement
Place buildings on grid:

$$\text{Grid} = \{(i \cdot \Delta x, j \cdot \Delta z) : i, j \in \mathbb{Z}\}$$

### Poisson Disk Sampling
Place buildings with minimum distance $r$:

For each candidate point $\mathbf{p}$:
$$\min_{i} d(\mathbf{p}, \mathbf{p}_i) \geq r$$

Where $\mathbf{p}_i$ are existing buildings.

### Building Size Distribution
Building size follows distribution:

$$P(\text{size} = s) = f(s)$$

Common distributions:
- **Uniform**: $f(s) = \frac{1}{s_{\max} - s_{\min}}$
- **Normal**: $f(s) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(s-\mu)^2}{2\sigma^2}}$
- **Exponential**: $f(s) = \lambda e^{-\lambda s}$

### Building Height
Height based on zone and density:

$$h = h_{\text{base}} + h_{\text{zone}} \cdot \text{density} + \text{noise}$$

---

## Road Network Mathematics

### Graph Representation
Road network as graph:

$$G = (V, E)$$

Where:
- $V$ = intersections/road endpoints
- $E$ = road segments

### Road Hierarchy
Assign hierarchy level $l$ to each road:

$$l = f(\text{traffic}, \text{length}, \text{connections})$$

Higher $l$ = more important road.

### Intersection Types
- **T-junction**: 3 roads meet
- **Crossroads**: 4 roads meet
- **Roundabout**: Circular intersection

### Road Width
Width based on hierarchy:

$$w = w_{\min} + (w_{\max} - w_{\min}) \cdot \frac{l}{l_{\max}}$$

---

## Geometric Algorithms

### Polygon Triangulation
Triangulate city blocks for rendering:

Given polygon vertices $\{\mathbf{v}_1, \mathbf{v}_2, \ldots, \mathbf{v}_n\}$:
- Triangulate into triangles
- Ear clipping algorithm: $O(n^2)$
- Delaunay triangulation: $O(n \log n)$

### Convex Hull
Find city boundaries:

$$\text{Hull} = \text{ConvexHull}(\{\text{all city points}\})$$

Graham scan: $O(n \log n)$

### Point-in-Polygon
Check if building is in zone:

$$\text{Inside} = \text{PointInPolygon}(\mathbf{p}, \text{zone polygon})$$

Ray casting algorithm.

---

## Density Calculations

### Population Density
Density at point $\mathbf{p}$:

$$\rho(\mathbf{p}) = \sum_i w_i \cdot K(d(\mathbf{p}, \mathbf{p}_i))$$

Where:
- $w_i$ = weight at point $\mathbf{p}_i$
- $K$ = kernel function (e.g., Gaussian)

### Gaussian Kernel
$$K(d) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{d^2}{2\sigma^2}}$$

### Building Density
Number of buildings per unit area:

$$\rho_{\text{buildings}} = \frac{|\text{buildings in region}|}{\text{area of region}}$$

---

## Procedural Generation Parameters

### City Size
Total city area:

$$A_{\text{city}} = w \times h$$

### Road Density
Road length per unit area:

$$\rho_{\text{roads}} = \frac{\text{total road length}}{A_{\text{city}}}$$

### Building Coverage
Fraction of area covered by buildings:

$$C = \frac{A_{\text{buildings}}}{A_{\text{city}}}$$

### Green Space Ratio
Fraction of parks/green space:

$$G = \frac{A_{\text{parks}}}{A_{\text{city}}}$$

---

## Noise Functions for Variation

### Perlin Noise
Generate natural variation:

$$n(x, z) = \sum_{i=0}^{k} \text{PerlinNoise}(x \cdot 2^i, z \cdot 2^i) \cdot \frac{1}{2^i}$$

### Fractal Noise
$$n_{\text{fractal}}(x, z) = \sum_{i=0}^{octaves} \text{noise}(x \cdot \text{freq}^i, z \cdot \text{freq}^i) \cdot \text{amp}^i$$

Where:
- $\text{freq}$ = frequency multiplier
- $\text{amp}$ = amplitude multiplier

---

## Optimization Mathematics

### Spatial Hashing
Hash function for spatial lookup:

$$h(x, z) = \left\lfloor \frac{x}{\text{cell size}} \right\rfloor + \left\lfloor \frac{z}{\text{cell size}} \right\rfloor \cdot \text{grid width}$$

### Culling
Frustum culling test:

$$\text{visible} = \text{TestPlanesAABB}(\text{frustum planes}, \text{building AABB})$$

### LOD Selection
Choose LOD level based on distance:

$$LOD = \begin{cases}
0 & \text{if } d < d_0 \\
1 & \text{if } d_0 \leq d < d_1 \\
2 & \text{if } d_1 \leq d < d_2 \\
\vdots
\end{cases}$$

---

## Exercises for Self-Learning

1. **L-System**: Implement simple L-system for road generation
2. **Voronoi**: Generate Voronoi diagram for city zones
3. **Poisson Sampling**: Implement Poisson disk sampling for buildings
4. **Density Calculation**: Calculate and visualize population density
5. **Graph Analysis**: Analyze road network as graph (find shortest paths)

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- L-systems and formal grammars
- Computational geometry algorithms
- Agent-based modeling
- Voronoi diagrams
- Procedural generation papers
