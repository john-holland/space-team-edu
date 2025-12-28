# Heightmaps: Mathematics, Matrices, and Manifolds

## Overview
Understanding the mathematical foundations of heightmaps, including matrix operations for transformations, manifold theory for smooth surfaces, and interpolation methods.

## Learning Objectives
- Master heightmap coordinate systems and transformations
- Understand interpolation mathematics (bilinear, bicubic)
- Learn gradient and normal calculations
- Explore manifold concepts for smooth terrain
- Calculate slopes, curvature, and other terrain properties

---

## Heightmap Coordinate Systems

### 2D Grid Coordinates
A heightmap is a 2D array $H$ where:
- $H[i][j]$ = height value at grid position $(i, j)$
- $i \in [0, width-1]$, $j \in [0, height-1]$
- Usually stored as $H[y][x]$ or $H[row][col]$

### World Space Coordinates
Convert grid coordinates to world space:

$$x_{\text{world}} = x_{\text{grid}} \cdot \text{scale}_x + \text{offset}_x$$
$$z_{\text{world}} = z_{\text{grid}} \cdot \text{scale}_z + \text{offset}_z$$
$$y_{\text{world}} = H[x_{\text{grid}}][z_{\text{grid}}] \cdot \text{scale}_y + \text{offset}_y$$

Where:
- $\text{scale}_x, \text{scale}_z$ = horizontal spacing (e.g., meters per pixel)
- $\text{scale}_y$ = vertical scaling factor
- $\text{offset}$ = world space origin

### Inverse Transformation
Convert world coordinates to grid coordinates:

$$x_{\text{grid}} = \frac{x_{\text{world}} - \text{offset}_x}{\text{scale}_x}$$
$$z_{\text{grid}} = \frac{z_{\text{world}} - \text{offset}_z}{\text{scale}_z}$$

---

## Matrix Transformations

### Heightmap as Matrix
Heightmap can be represented as matrix:

$$\mathbf{H} = \begin{pmatrix}
h_{0,0} & h_{0,1} & \cdots & h_{0,w-1} \\
h_{1,0} & h_{1,1} & \cdots & h_{1,w-1} \\
\vdots & \vdots & \ddots & \vdots \\
h_{h-1,0} & h_{h-1,1} & \cdots & h_{h-1,w-1}
\end{pmatrix}$$

Where $w$ = width, $h$ = height.

### Translation Matrix
Translate heightmap in world space:

$$\mathbf{T} = \begin{pmatrix}
1 & 0 & 0 & t_x \\
0 & 1 & 0 & t_y \\
0 & 0 & 1 & t_z \\
0 & 0 & 0 & 1
\end{pmatrix}$$

### Scaling Matrix
Scale heightmap:

$$\mathbf{S} = \begin{pmatrix}
s_x & 0 & 0 & 0 \\
0 & s_y & 0 & 0 \\
0 & 0 & s_z & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}$$

### Rotation Matrix (around Y-axis)
Rotate terrain:

$$\mathbf{R}_y(\theta) = \begin{pmatrix}
\cos\theta & 0 & \sin\theta & 0 \\
0 & 1 & 0 & 0 \\
-\sin\theta & 0 & \cos\theta & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}$$

---

## Interpolation Methods

### Nearest Neighbor
Use height of nearest grid point:

$$H(x, z) = H[\lfloor x \rfloor][\lfloor z \rfloor]$$

### Bilinear Interpolation
Interpolate between 4 neighboring points:

Given point $(x, z)$ with fractional parts:
- $fx = x - \lfloor x \rfloor$
- $fz = z - \lfloor z \rfloor$

Get 4 corner heights:
- $h_{00} = H[i][j]$
- $h_{10} = H[i+1][j]$
- $h_{01} = H[i][j+1]$
- $h_{11} = H[i+1][j+1]$

Bilinear interpolation:
$$H(x,z) = (1-fx)(1-fz)h_{00} + fx(1-fz)h_{10} + (1-fx)fz h_{01} + fx \cdot fz \cdot h_{11}$$

### Bicubic Interpolation
Smoother interpolation using 16 neighboring points:

$$H(x,z) = \sum_{i=0}^{3}\sum_{j=0}^{3} a_{ij} x^i z^j$$

Where coefficients $a_{ij}$ are determined by:
- 16 neighboring height values
- Continuity constraints (value and derivatives)

More computationally expensive but produces smoother results.

---

## Gradient Calculations

### Discrete Gradient (Forward Difference)
Gradient in X direction:

$$\frac{\partial H}{\partial x}[i][j] = \frac{H[i+1][j] - H[i][j]}{\Delta x}$$

Gradient in Z direction:

$$\frac{\partial H}{\partial z}[i][j] = \frac{H[i][j+1] - H[i][j]}{\Delta z}$$

### Central Difference (More Accurate)
Gradient in X direction:

$$\frac{\partial H}{\partial x}[i][j] = \frac{H[i+1][j] - H[i-1][j]}{2\Delta x}$$

Gradient in Z direction:

$$\frac{\partial H}{\partial z}[i][j] = \frac{H[i][j+1] - H[i][j-1]}{2\Delta z}$$

### Gradient Vector
$$\nabla H = \left(\frac{\partial H}{\partial x}, \frac{\partial H}{\partial z}\right)$$

---

## Normal Vector Calculation

### Surface Normal
For a heightmap surface $S(x,z) = (x, H(x,z), z)$, the normal vector is:

$$\mathbf{n} = \frac{\partial S}{\partial x} \times \frac{\partial S}{\partial z}$$

Where:
- $\frac{\partial S}{\partial x} = \left(1, \frac{\partial H}{\partial x}, 0\right)$
- $\frac{\partial S}{\partial z} = \left(0, \frac{\partial H}{\partial z}, 1\right)$

Cross product:
$$\mathbf{n} = \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
1 & \frac{\partial H}{\partial x} & 0 \\
0 & \frac{\partial H}{\partial z} & 1
\end{vmatrix} = \left(-\frac{\partial H}{\partial x}, 1, -\frac{\partial H}{\partial z}\right)$$

Normalized:
$$\hat{\mathbf{n}} = \frac{\mathbf{n}}{|\mathbf{n}|} = \frac{\left(-\frac{\partial H}{\partial x}, 1, -\frac{\partial H}{\partial z}\right)}{\sqrt{1 + \left(\frac{\partial H}{\partial x}\right)^2 + \left(\frac{\partial H}{\partial z}\right)^2}}$$

---

## Slope and Angle Calculations

### Slope
Slope magnitude:

$$\text{slope} = |\nabla H| = \sqrt{\left(\frac{\partial H}{\partial x}\right)^2 + \left(\frac{\partial H}{\partial z}\right)^2}$$

### Slope Angle
Angle from horizontal:

$$\theta = \arctan(|\nabla H|) = \arctan\left(\sqrt{\left(\frac{\partial H}{\partial x}\right)^2 + \left(\frac{\partial H}{\partial z}\right)^2}\right)$$

### Aspect (Direction of Slope)
Direction of steepest descent:

$$\phi = \arctan2\left(\frac{\partial H}{\partial z}, \frac{\partial H}{\partial x}\right)$$

---

## Curvature Calculations

### Mean Curvature
Second derivative approximation:

$$H_{\text{mean}} = \frac{H_{xx}(1 + H_z^2) - 2H_{xz}H_xH_z + H_{zz}(1 + H_x^2)}{2(1 + H_x^2 + H_z^2)^{3/2}}$$

Where:
- $H_{xx} = \frac{\partial^2 H}{\partial x^2}$
- $H_{zz} = \frac{\partial^2 H}{\partial z^2}$
- $H_{xz} = \frac{\partial^2 H}{\partial x \partial z}$

### Gaussian Curvature
$$K = \frac{H_{xx}H_{zz} - H_{xz}^2}{(1 + H_x^2 + H_z^2)^2}$$

### Discrete Second Derivatives
$$H_{xx}[i][j] = \frac{H[i+1][j] - 2H[i][j] + H[i-1][j]}{(\Delta x)^2}$$

$$H_{zz}[i][j] = \frac{H[i][j+1] - 2H[i][j] + H[i][j-1]}{(\Delta z)^2}$$

$$H_{xz}[i][j] = \frac{H[i+1][j+1] - H[i+1][j-1] - H[i-1][j+1] + H[i-1][j-1]}{4\Delta x \Delta z}$$

---

## Manifold Theory for Terrain

### Heightmap as 2D Manifold
A heightmap defines a 2D manifold embedded in 3D space:

$$M = \{(x, H(x,z), z) : (x,z) \in \mathbb{R}^2\}$$

### Parametric Representation
$$S(u,v) = (u, H(u,v), v)$$

Where $(u,v)$ are parameters in $[0,1] \times [0,1]$.

### Tangent Vectors
$$\mathbf{t}_u = \frac{\partial S}{\partial u} = \left(1, \frac{\partial H}{\partial u}, 0\right)$$
$$\mathbf{t}_v = \frac{\partial S}{\partial v} = \left(0, \frac{\partial H}{\partial v}, 1\right)$$

### Metric Tensor (First Fundamental Form)
$$g_{ij} = \begin{pmatrix}
1 + H_u^2 & H_u H_v \\
H_u H_v & 1 + H_v^2
\end{pmatrix}$$

### Area Element
$$dA = \sqrt{\det(g)} \, du \, dv = \sqrt{1 + H_u^2 + H_v^2} \, du \, dv$$

Total area:
$$A = \iint \sqrt{1 + H_u^2 + H_v^2} \, du \, dv$$

---

## Filtering and Smoothing Operations

### Gaussian Blur
Convolve heightmap with Gaussian kernel:

$$G(x,z) = \frac{1}{2\pi\sigma^2} e^{-\frac{x^2 + z^2}{2\sigma^2}}$$

$$H_{\text{smoothed}}[i][j] = \sum_{k,l} H[i-k][j-l] \cdot G(k,l)$$

### Laplacian Smoothing
$$H_{\text{new}}[i][j] = H[i][j] + \lambda \nabla^2 H[i][j]$$

Where Laplacian:
$$\nabla^2 H = H_{xx} + H_{zz}$$

### Median Filter
Replace height with median of neighborhood (removes spikes).

---

## Heightmap Operations

### Addition/Subtraction
Add two heightmaps:

$$H_3 = H_1 + H_2$$

### Multiplication (Scaling)
Scale heights:

$$H_{\text{scaled}} = \alpha \cdot H$$

### Blending
Blend two heightmaps:

$$H_{\text{blend}} = \alpha H_1 + (1-\alpha) H_2$$

Where $\alpha \in [0,1]$ is blend factor.

---

## Distance Calculations on Heightmap

### Geodesic Distance
Distance along surface (not straight-line):

For small distances, approximate using metric tensor:
$$ds^2 = g_{11} du^2 + 2g_{12} du \, dv + g_{22} dv^2$$

### Straight-Line Distance
3D Euclidean distance:

$$d = \sqrt{(x_2-x_1)^2 + (H(x_2,z_2) - H(x_1,z_1))^2 + (z_2-z_1)^2}$$

---

## Exercises for Self-Learning

1. **Interpolation**: Implement bilinear interpolation for height lookup
2. **Normal Calculation**: Calculate surface normals from heightmap gradients
3. **Slope Analysis**: Compute slope map from heightmap
4. **Curvature**: Calculate mean and Gaussian curvature
5. **Transformation**: Apply rotation and scaling matrices to heightmap coordinates

---

## Next Steps
- **Usage**: Review practical applications (1-usage.md)
- **Implementation**: See Unity code with these formulas (3-unity-implementation.md)

---

## References
- Differential Geometry (manifolds)
- Numerical Methods (interpolation, derivatives)
- Computer Graphics (surface representation)
- Terrain Analysis algorithms
