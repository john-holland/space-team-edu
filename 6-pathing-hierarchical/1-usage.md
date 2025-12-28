# Hierarchical Pathing: Usage and Applications

## Overview
Hierarchical pathfinding uses multiple levels of abstraction to find paths efficiently in large, complex environments. This document covers use cases, applications, and when to use hierarchical pathfinding.

## Learning Objectives
- Understand hierarchical pathfinding concepts
- Identify use cases and applications
- Learn different hierarchical approaches
- Recognize performance benefits

---

## What is Hierarchical Pathfinding?

### Definition
Hierarchical pathfinding breaks pathfinding into multiple levels:
- **High Level**: Navigate between regions/areas
- **Low Level**: Navigate within regions
- **Abstraction**: Simplify complex spaces into manageable graphs

### Basic Concept
- **Multi-Level**: Pathfinding at different scales
- **Abstraction**: Complex space → simple graph
- **Efficiency**: Much faster than single-level pathfinding
- **Scalability**: Handles very large environments

---

## Common Use Cases

### 1. Large Open Worlds
**Problem**: Pathfinding across kilometers of terrain
- **Use Case**: Open-world games, MMOs
- **Benefit**: Find paths quickly across huge areas
- **Example**: Navigate from city to city, then within city

### 2. RTS Games
**Problem**: Pathfinding for many units
- **Use Case**: Strategy games, large armies
- **Benefit**: Efficient pathfinding for hundreds of units
- **Example**: Move army across map, then navigate around obstacles

### 3. AI Navigation
**Problem**: NPCs need to navigate complex environments
- **Use Case**: NPCs in games, autonomous agents
- **Benefit**: Fast pathfinding for many agents
- **Example**: NPCs navigate city, then navigate building

### 4. Multi-Scale Environments
**Problem**: Navigate at different scales (space → planet → city)
- **Use Case**: Space games, multi-scale simulations
- **Benefit**: Handle vastly different scales
- **Example**: Navigate solar system, then land on planet, then navigate city

### 5. Dynamic Environments
**Problem**: Pathfinding when environment changes
- **Use Case**: Destructible environments, dynamic obstacles
- **Benefit**: Update only affected levels
- **Example**: Building destroyed, update local graph, keep high-level same

---

## When to Use Hierarchical Pathfinding

### ✅ Good Use Cases
- **Large environments** (kilometers or more)
- **Many agents** (hundreds or thousands)
- **Multi-scale** (different levels of detail)
- **Complex spaces** (many obstacles, complex geometry)
- **Dynamic updates** (environment changes)

### ❌ Poor Use Cases
- **Small environments** (overhead not worth it)
- **Simple spaces** (simple A* sufficient)
- **Few agents** (simple pathfinding enough)
- **Static environments** (precomputed paths might be better)

---

## Hierarchical Approaches

### 1. Region-Based
- **Concept**: Divide space into regions, pathfind between regions
- **High Level**: Graph of regions
- **Low Level**: Pathfinding within regions
- **Use**: Open worlds, large maps

### 2. Cluster-Based
- **Concept**: Cluster nodes into groups
- **High Level**: Graph of clusters
- **Low Level**: Detailed graph within clusters
- **Use**: Complex graphs, many nodes

### 3. Abstraction-Based
- **Concept**: Create abstract representation
- **High Level**: Simplified graph
- **Low Level**: Detailed graph
- **Use**: Navigation meshes, waypoint systems

### 4. Multi-Resolution
- **Concept**: Multiple resolutions of same space
- **High Level**: Low resolution
- **Low Level**: High resolution
- **Use**: Terrain pathfinding, heightmaps

---

## Performance Characteristics

### Time Complexity
- **Single-Level A***: $O(b^d)$ where $b$ is branching, $d$ is depth
- **Hierarchical**: $O(\log n)$ high-level + $O(k)$ low-level
- **Benefit**: Much faster for large spaces

### Memory Usage
- **High-Level Graph**: Small, abstract
- **Low-Level Graphs**: One per region
- **Total**: Usually less than single detailed graph

### Scalability
- **Single-Level**: Doesn't scale well
- **Hierarchical**: Scales to very large environments
- **Benefit**: Handles kilometers of terrain

---

## Practical Examples

### Example 1: Open World Game
```
Scenario: 50km² game world
High Level: Navigate between cities/regions
Low Level: Navigate within city streets
Result: Fast pathfinding across entire world
```

### Example 2: RTS Game
```
Scenario: 1000 units need paths
High Level: Move to target region
Low Level: Avoid obstacles within region
Result: Efficient pathfinding for many units
```

### Example 3: Space Game
```
Scenario: Navigate solar system to city
Level 1: Solar system navigation
Level 2: Planet surface navigation  
Level 3: City street navigation
Result: Handle vastly different scales
```

---

## Comparison with Single-Level Pathfinding

### Single-Level A*
- **Pros**: Simple, works for small spaces
- **Cons**: Slow for large spaces, doesn't scale
- **Use**: Small maps, simple environments

### Hierarchical Pathfinding
- **Pros**: Fast, scalable, handles large spaces
- **Cons**: More complex, setup overhead
- **Use**: Large maps, complex environments

---

## Exercises for Self-Learning

1. **Use Case Analysis**: Identify games using hierarchical pathfinding
2. **Design Decision**: Choose hierarchical approach for specific game
3. **Performance Analysis**: Compare hierarchical vs. single-level
4. **Multi-Scale Design**: Design pathfinding for multi-scale game
5. **Implementation Research**: Research hierarchical pathfinding algorithms

---

## Next Steps
- **Math**: Learn hierarchical pathfinding mathematics (2-math.md)
- **Implementation**: See Unity code examples (3-unity-implementation.md)
- **Voxel Pathing**: Learn pathfinding with voxels (4-pathing-with-dual-encoded-voxels.md)

---

## References and Further Reading
- Hierarchical pathfinding algorithms
- A* pathfinding optimization
- Navigation mesh systems
- Multi-scale pathfinding papers
