# Hierarchical Pathfinding: Mathematics

## Overview
Mathematical foundations for hierarchical pathfinding, including graph abstraction, multi-level pathfinding algorithms, and complexity analysis.

## Learning Objectives
- Master graph abstraction mathematics
- Understand hierarchical pathfinding algorithms
- Learn complexity analysis for hierarchical systems
- Calculate pathfinding costs and optimizations

---

## Graph Abstraction

### Base Graph
Original detailed graph:
$$G = (V, E)$$

Where:
- $V$ = set of vertices (nodes)
- $E$ = set of edges (connections)
- $|V| = n$ = number of vertices
- $|E| = m$ = number of edges

### Abstract Graph
High-level abstract graph:
$$G_A = (V_A, E_A)$$

Where:
- $V_A$ = abstract vertices (regions/clusters)
- $E_A$ = abstract edges (connections between regions)
- $|V_A| = n_A \ll n$
- $|E_A| = m_A \ll m$

### Abstraction Ratio
$$r = \frac{n}{n_A}$$

Typical values: $r = 10$ to $100$ or more.

---

## Region-Based Hierarchical Pathfinding

### Region Division
Divide space into $k$ regions:
$$R = \{R_1, R_2, \ldots, R_k\}$$

Each region contains subset of vertices:
$$V_i \subseteq V, \quad \bigcup_{i=1}^{k} V_i = V, \quad V_i \cap V_j = \emptyset \text{ for } i \neq j$$

### Region Graph
Create graph of regions:
$$G_R = (R, E_R)$$

Edge $(R_i, R_j) \in E_R$ if regions are adjacent or connected.

### Pathfinding Process
1. **High-level**: Find path in region graph
   $$P_A = \{R_{s}, R_{i_1}, R_{i_2}, \ldots, R_{t}\}$$

2. **Low-level**: Find paths within each region
   $$P_i = \text{path within } R_i$$

3. **Connect**: Connect paths at region boundaries

---

## Cluster-Based Hierarchical Pathfinding

### Clustering
Group vertices into clusters:
$$C = \{C_1, C_2, \ldots, C_k\}$$

Each cluster:
$$C_i = \{v_{i,1}, v_{i,2}, \ldots, v_{i,|C_i|}\}$$

### Cluster Graph
Create graph of clusters:
$$G_C = (C, E_C)$$

Edge $(C_i, C_j) \in E_C$ if clusters are connected.

### Hierarchical Levels
Multiple levels of abstraction:
- **Level 0**: Original graph $G$
- **Level 1**: Clusters $C^1$
- **Level 2**: Clusters of clusters $C^2$
- **Level $L$**: Top-level clusters

---

## A* Pathfinding Mathematics

### A* Cost Function
$$f(n) = g(n) + h(n)$$

Where:
- $g(n)$ = cost from start to node $n$
- $h(n)$ = heuristic estimate from $n$ to goal
- $f(n)$ = estimated total cost

### Heuristic Requirements
**Admissible**: $h(n) \leq h^*(n)$ (never overestimate)

**Consistent**: $h(n) \leq c(n, n') + h(n')$ for all neighbors $n'$

Where $h^*(n)$ is true cost and $c(n, n')$ is edge cost.

### Euclidean Heuristic
$$h(n) = \sqrt{(x_g - x_n)^2 + (y_g - y_n)^2 + (z_g - z_n)^2}$$

### Manhattan Heuristic
$$h(n) = |x_g - x_n| + |y_g - y_n| + |z_g - z_n|$$

---

## Hierarchical A* Complexity

### Single-Level A*
**Time Complexity**: $O(b^d)$ worst case
- $b$ = branching factor
- $d$ = depth of solution

**Space Complexity**: $O(b^d)$

### Two-Level Hierarchical
**High-Level**: $O(b_A^{d_A})$
- $b_A$ = branching in abstract graph
- $d_A$ = depth in abstract graph

**Low-Level**: $O(b^{d_{\text{local}}})$ per region
- $d_{\text{local}}$ = depth within region

**Total**: $O(b_A^{d_A} + k \cdot b^{d_{\text{local}}})$

Where $k$ is number of regions in path.

### Improvement
If $b_A^{d_A} + k \cdot b^{d_{\text{local}}} \ll b^d$, hierarchical is faster.

---

## Path Cost Calculation

### Edge Cost
Cost of edge $(u, v)$:
$$c(u, v) = w(u, v)$$

Where $w$ is weight function (distance, time, etc.).

### Path Cost
Cost of path $P = \{v_0, v_1, \ldots, v_k\}$:
$$C(P) = \sum_{i=0}^{k-1} c(v_i, v_{i+1})$$

### Hierarchical Path Cost
High-level cost:
$$C_A(P_A) = \sum_{i=0}^{k-1} c_A(R_i, R_{i+1})$$

Where $c_A(R_i, R_{i+1})$ is cost between regions.

Low-level cost within region:
$$C_i(P_i) = \sum_{j} c(v_{i,j}, v_{i,j+1})$$

Total cost:
$$C_{\text{total}} = C_A(P_A) + \sum_{i} C_i(P_i)$$

---

## Region Boundary Costs

### Entry/Exit Points
For region $R_i$:
- **Entry point**: $e_i$ (where path enters)
- **Exit point**: $x_i$ (where path exits)

### Boundary Cost
Cost to cross boundary:
$$c_{\text{boundary}}(R_i, R_j) = \min_{e_i \in \partial R_i, e_j \in \partial R_j} d(e_i, e_j)$$

Where $\partial R_i$ is boundary of region $R_i$.

### Precomputed Costs
Precompute costs between boundary points:
$$c_{\text{precomputed}}(e_i, e_j) = \text{shortest path cost}$$

Store in lookup table for fast access.

---

## Multi-Resolution Pathfinding

### Resolution Levels
Multiple resolutions of same space:
- **Level 0**: Full resolution (1m per node)
- **Level 1**: Coarse resolution (10m per node)
- **Level 2**: Very coarse (100m per node)

### Resolution Ratio
$$r_i = \frac{\text{resolution at level } i}{\text{resolution at level } 0}$$

Example: $r_1 = 10$, $r_2 = 100$.

### Node Count
At level $i$:
$$n_i = \frac{n_0}{r_i^d}$$

Where $d$ is dimensionality (2D or 3D).

---

## Dynamic Updates

### Local Updates
When edge cost changes:
- **High-level**: May or may not need update
- **Low-level**: Update affected region only

### Update Cost
If change affects region $R_i$:
- **Region update**: $O(|V_i| \log |V_i|)$
- **Boundary update**: $O(|\partial R_i|^2)$

Much cheaper than full graph update: $O(n \log n)$.

---

## Optimality Guarantees

### Hierarchical Optimality
Hierarchical path may not be globally optimal:
$$C_{\text{hierarchical}} \geq C_{\text{optimal}}$$

### Bounded Suboptimality
If hierarchical path is within factor $\epsilon$:
$$C_{\text{hierarchical}} \leq (1 + \epsilon) C_{\text{optimal}}$$

### Refinement
Refine hierarchical path to improve:
1. Find hierarchical path
2. Refine at boundaries
3. Check for shortcuts

---

## Memory Requirements

### Graph Storage
**Base graph**: $O(n + m)$

**Abstract graph**: $O(n_A + m_A)$

**Boundary data**: $O(|\partial R|^2)$ per region

**Total**: $O(n + m + n_A + m_A + k \cdot |\partial R|^2)$

### Precomputed Costs
Store all-pairs shortest paths between boundary points:
$$M_{\text{precomputed}} = O(k \cdot |\partial R|^2)$$

---

## Exercises for Self-Learning

1. **Graph Abstraction**: Create abstract graph from detailed graph
2. **Hierarchical A***: Implement two-level hierarchical pathfinding
3. **Cost Calculation**: Calculate path costs at different levels
4. **Complexity Analysis**: Compare single-level vs. hierarchical complexity
5. **Refinement**: Implement path refinement algorithm

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Graph theory and algorithms
- A* pathfinding optimization
- Hierarchical pathfinding papers
- Multi-resolution algorithms
