Implicit surfaces define shapes as solutions to f(x,y,z)=0. For example, a sphere is defined as:
$$ x^2 + y^2 + z^2 - r^2 = 0$$
These equations can be expanded from ray tracing, where they are used for hit detection, into a powerful modeling framework.
## Implicit Surfaces as Distance Thresholds
We can define implicit surfaces using distance from some geometry. Instead of a fixed radius, we can use a distance threshold:
$$f(p) = t - d(p, G)$$
- *d(p,G)* is the shortest distance from point p to geometry G
- *t* is the threshold defining the surface
Alternatively, we can use squared distance:
$$f(p) = t^2 - d^2(p, G)$$
This defines an implicit surface based on:
$$f(p) = \begin{cases} > 0, & \text{inside the surface} \\ = 0, & \text{on the surface} \\ < 0, & \text{outside the surface} \end{cases}$$
Using square distance avoids computing square roots and is often more efficient. This approach extends to lines curves and other geometric primitives.
## Skeletal Primitives
Any geometric shape can serve as a skeletal primitive for an implicit surface. the only requirement is that we can define a distance function from any point to the geometry. Common skeletal primitives include points, line segments, basic 2D shapes, and basic 3D shapes. Choosing different skeletal primitives allows for flexible shape modeling while maintaining smooth blending and continuity.
## Line Segments and Skeletal Primitives
Line segments act as extended skeletal elements. Each point *p* in space is influenced by the closest point on the line segment. Distance calculation determines the implicit field contribution.
### Distance from a Point to a Line Segment
Given a line segment defined by endpoints A and B, the closest point on the segment to p is found via projection:
$$t = \frac{(p - A) \cdot (B - A)}{\|B - A\|^2}$$
The closest point is:
$$P_{\text{closest}} = A + \text{clamp}(t, 0, 1) \cdot (B - A)$$
The distance to the segment is:
$$d(p) = \| p - P_{\text{closest}} \|$$
This distance replaces *d(x,,y,z)* in implicit function definitions. Note that this can be simplified to a standardized axis-aligned unit line when used in conjunction with the inverse transform of test points, allowing it to define arbitrary line segments efficiently.

## Distance Fields and Implicit Surfaces
A distance field assigns each point in space a value representing its shortest distance to a given geometry. For every point *p* we compute *d(p,G)*. The distance field allows us to evaluate any point in space relative to the geometry. We can define an implicit surface by thresholding the distance field:
$$f(p) = t-d(p,G)$$
The surface exists where:
$$f(p) = 0  => d(p,G)=t$$
Points inside have f(p) > 0 and points outside are less than 0.
## Iso-Surface and Iso-Value Terminology
An iso-surface is a three dimensional surface that represents all points in space where a scalar field takes on a constant value. The iso-value is the specific constant that defines the surface, determining where the transition occurs. In different contexts, the iso-value is also referred to as:
- Threshold (t, T, or thresh) - This is common in implicit modeling and volume rendering
- Offset Radius (r) - Used in distance fields and expanded geometry
- Level set - Used in numerical simulations and physics-based modeling
- Contour level - Used in visualization, such as topographical maps
Typically iso-surfaces are extracted from continuous fields like elevation, distance, or density. Adjusting iso-value / threshold shifts the surface, revealing different layers or structural details in the field.
### Water Level as a Contour Line
For a conceptual analogy, imagine a terrain where the elevation is modeled as a scalar field. A water level defines a contour line at:
$$ f(x,y) = w$$ Where *w* is the water level. Changing the water level alters the visible landforms. Higher water levels submerge more terrain, and lower water levels reveal more. Similar to how a water level selects a contour line, we can visualize iso-values in other scalar fields, such as distance fields or density fields.
![](../Images/Pasted%20image%2020261027165450.png)
## Implicit Surfaces from Multiple Skeletal Primitives
So far we have considered distance fields from a single geometry. When multiple geometric elements are present, we need a way to combine their influence. There are two main approaches:
 - Union of Distance Influence
 - Combining Distances for Compound Shapes
### Union of Distance Influence
The simplest approach is to determine which individual geometry has the closest influence. This is done by selecting the minimum distance before applying a threshold:
$$f(p) = t - \min(d_1(p), d_2(p), ..., d_n(p))$$
This results in a shape that is the union of the individual surfaces.
### Combining Distances for Compound Shapes
Instead of taking the minimum distance, we can combine distances with a gradual transition to form new implicit shapes. This is less straightforward than the union approach because different distance functions may require different blending strategies, and the combination method affects smoothness and continuity.
## Summation Blending
A raw distance field assigns values based on the shortest distance to the nearest shape. Two issues arise:
- Local Influence - nearby shapes could contribute more strongly than distance ones
- Blending - When multiples shapes influence a region, we need smooth transitions
These problems motivate transforming the raw distance field into a basis-mapped distance field that is additive.
## Inverting Distance for Local Influence
Distance fields treat large values as further away, but we need larger values to mean stronger influence. A simple approach is to invert the distance, but it creates issues when the distance is zero. Influence also drops too rapidly near the geometry, causing poor blending. Instead of direct inversion, we use a controlled non-linear mapping to ensure a smooth and gradual decay of influence.
## Nonlinear Drop-Off for Smooth Blending
Instead of a sharp inversion, we can define a basis-mapped distance field:
$$F(p) = g(d,(p,G))$$
Where *g(d)* is a smooth function (or fall off filter) that:
- Enhances local influence, ensuring nearby primitives contribute more significantly
- Decays smoothly with distance, preventing abrupt falloff and ensuring controlled influence
- Facilitates smooth blending by shaping how distances from multiple primitives contribute to the final surface
The choice of *g(d)* directly affects the final shape blending, and a well chosen function ensures C1 continuity. This prevents artifacts and maintains smooth normals.
## Common Threshold for Implicit Evaluation
Once we transform the distance field, we define the common threshold:
$$F(p) = T$$
This threshold determines the implicit surface:
$$F(p) = \begin{cases} > T, & \text{inside the surface} \\ = T, & \text{on the surface} \\ < T, & \text{outside the surface} \end{cases}$$
## Blobby Implicit Surfaces and Summation Blending
Blobby Implicit Surfaces define smooth, organic shapes by summing influence functions centered on skeletal primitives. Instead of using explicit geometry, these surfaces emerge where a field function meets a threshold. This technique is commonly found in soft-body modeling, fluid simulation, and procedural shape generation.
![](../Images/Pasted%20image%2020261027172038.png)
The surface is defined implicitly by the sum of falloff functions, ensuring smooth blending between overlapping regions.
## Defining a Blobby Implicit Surface
A blobby implicit surface is defined using a threshold T:
$$F(p) = \sum_{i=1}^{n} g_i(d_i(p)) - T$$
The surface exists where *f(p)* = 0:
$$F(p) = \begin{cases} > 0, & \text{inside the surface} \\ = 0, & \text{on the surface} \\ < 0, & \text{outside the surface} \end{cases}$$
Basis functions *g_i(d_i(p))* are summed over all skeletal primitives. These functions are designed to be non-linear fall off filters to ensure smooth blending between elements that are summed. 
## Basis Functions
Many implicit surfaces are defined by summing basis functions centered at skeletal elements. Basis functions control the influence of each primitive over space. Common basis functions include:
- Blinn's Gaussian basis function
- *g(d) = e<sup>-ad<sup>2</sup></sup> 
- Wyvill's basis function:
$$g(d) = (1 - (d/r)^2)^3, \quad d < r \text{ ($r$ is often set to $1$)}$$
These functions are non-linear and generally continuously differentiable to support perceptively smooth blending between primitives.
## Basis Function Terminology
Basis functions are also referred to as:
- Fall-off functions or fall-off filters - emphasizes how influence decreases with distance
- Potential functions - common in physics based modeling and soft objects
- Radial basis functions - used in interpolation and implicit surface blending
- Kernels - frequently used in signal processing, machine learning, and density estimation
These terms are interchangeable depending on the context, but they all can describe functions that define influence based on distance from a skeletal primitive.
## Blinn vs Wyvill Basis Functions
![](../Images/Pasted%20image%2020261027173255.png)
Blinn's Gaussian function extends indefinitely, meaning it never fully reaches zero. Wyvill's basis function has a cutoff beyond which all values are zero. This difference plays a key role in how each function influences the shape and smoothness of the implicit surface.
## Compact vs. Global Support
Wyvill's basis function has compact support, meaning it has local influence. It only affects points within a finite region. In contrast, Blinn's Gaussian basis function has global support, meaning it never fully reaches zero and contributes even at large distances.

Advantages of compact support:
- We only need to sum contributions where fields overlap, reducing computational cost
- Blinn's Gaussian requires summing all skeletal elements, even those far away, making it computationally expensive for large numbers of primitives

Downsides of compact support:
- The hard cutoff can introduce slight discontinuities in blending, potentially affecting smoothness
- Blending between multiple primitives may require additional tuning to avoid visible artifacts
## Transforming Implicit Surfaces
Implicit surfaces are defined by a function *F(p)= 0*. When transforming an implicit surface, we apply the inverse transformation to the point *p*. This preserves the implicit function without needing to modify its definition. Scaling, rotation, and translation can all be handled using inverse transformations.
## Inverse Transform of a Point
Given a transformation T, instead of modifying the function, we modify the point before evaluation:
$$F(p)=g(d(p))$$
When transforming an implicit surface, we apply the inverse transformation to the point:
$$F(T^{-1}(p)) = g(d(T^{-1}(p)))$$
This approach keeps the function unchanged and ensures transforms affect the evaluation point, not the field definition. It works for affine transformations such as scaling, rotation, and translation.
## Rendering Implicit Surfaces
Two common approaches for rendering implicit surface are:
- Ray Tracing
	- Uses ray marching (sphere tracing) to iteratively refine surface intersection
	- This works well for smooth implicit surfaces but can be computationally expensive
- Polygonization
	- Converts the implicit surface into a polygonal mesh for rasterization
	- Involves Iso-Surface Extraction, which converts volumetric data into explicit geometry
	- Marching Cubes is a widely used Iso-Surface Extraction Method
## Marching Squares
Marching squares is the 2D equivalent of marching cubes. The goal is to extract contours from a scalar field. Each square cell is assigned a binary configuration:
- A vertex is inside if *f(p)* >= 0
- A vertex is outside if *f(p)* < 0
There are 16 possible cases, but rotational symmetry reduces them to 4 unique cases. The method interpolates along edges to construct the contour.
![](../Images/Pasted%20image%2020261027175149.png)
![](../../Pasted%20image%2020261027175306.png)
In the above image we see 2D cells with values at their corners. Interpolated points are found where the scalar field changes sign at each cell edge. These points are where the scalar value is zero. The interpolated points are then connected to form polygons, which create the contour around the positive values in the scalar field.
## Marching Cubes
Marching Cubes extends the squares to three dimensions. Each cube in the grid is classified based on which of its eight vertices are inside or outside. There are 256 possible combinations, but symmetry and transforms reduce these cases to 15 unique configurations. ![](../../Pasted%20image%2020261027175728.png)
Lookup tables store the polygonal patches for each case allowing for efficient extraction. Marching cubes is widely used in medical imaging such as MRI and CT scans, as well as in scientific visualization. 
## Dual Contouring
Dual Contouring improves edge preservation in marching cubes. It places vertices inside each grid cell instead of only interpolating along edges. Hermite data (surface normals) are used to detect and preserve sharp edges. This allows for more designer control, allowing selection of which edges remain sharp or smooth. Dual Contouring is well suited for CAD models, mechanical parts, and structured geometry. It improves representation of hard surface models in implicit surfaces.
## Constructive Solid Geometry (CSG)
Constructive Solid Geometry uses boolean operations on implicit surfaces:
**Union**: $$f_A \lor f_B$$
$$F_{\cup}(p) = \max(F_A(p), F_B(p))$$
**Intersection**: $$f_A \land f_B$$
$$F_{\cap}(p) = \min(F_A(p), F_B(p))$$
**Difference**: $$f_A \setminus f_B$$
$$F_{-}(p) = \min(F_A(p), 2 \cdot \text{thresh} - F_B(p))$$
## CSG For Multiple Surfaces
**Union**:
Take the maximum field value across all contributing shapes
$$F_{\cup}(p) = \max_{j}(F_j(p))$$
**Intersection**: 
Take the minimum field value across all contributing shapes
$$F_{\cap}(p) = \min_{j}(F_j(p))$$
**Intersection**: 
One shape has multiple others subtracted - 
$$F_{-}(p) = \min\left(F_0(p), 2 \cdot \text{thresh} - \max_{j>0}(F_j(p))\right)$$
## Warping and Deformation Techniques
Warping modifies shape by transforming space. Common warping operations include:
- Twist - rotates a shape progressively along an axis
- Taper - scales a shape along an axis based on position
- Bend - curves an object by modifying its spatial embedding
These can be used for procedural animation, organic modeling, and shape control.
### Twist Deformation
Twist applies a rotational transformation along an axis. A point *(x,y,z)* is transformed as:
$$x' = x\cos(\theta(z)) - y\sin(\theta(z))$$
$$y' = x\sin(\theta(z)) + y\cos(\theta(z))$$
$$z' = z$$
Where *theta(z)* is a function of position along the twist axis. This produces torsional effects seen in rubber-like materials.
### Taper
Tapering scales an object's size along an axis based on position. It is defined by a scaling function *k(z)*:
$$x' = k(z)x, \quad y' = k(z)y, \quad z' = z$$
Where *k(z)* is a linear or nonlinear function controlling the taper effect.
### Bend
Bending curves an object by modifying its spatial embedding. Exact formulation depends on bending constraints and coordinate transformations. This often involves mapping straight line coordinates into curves space, and using trigonometric functions to define curvature. Bending is more complex than twist and taper, and requires additional parameters.