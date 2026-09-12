The key to optimization in ray tracing is to eliminate unnecessary computations as early as possible. The goal is to determine if a ray does not intersect with geometry to save time.
**Core Strategies:**
- Skip rays that miss entire regions
- Minimize detailed intersection tests
- Leverage structured or hierarchical spatial relationships
**Challenges:**
- Complex geometry may be expensive to compute ray intersections against
- Dynamic scenes have moving objects, requiring frequent updates to acceleration structures
- Memory management is challenging in large scenes, but instancing and efficient acceleration structures mitigate this issues
- Algorithmic approximations may be used as faster methods, but balancing precision, speed, and accuracy is key

## Object Instancing
As a reminder, similarity transformations preserve shapes, angles, and proportional distances. Affine transformations preserve parallelism, collinearity, and convexity, but not angles and lengths. 

**Ray Tracing Transformed Geometry** via Object Instancing can be accomplished by:
1. Determining the desired geometry transform
2. Computing and storing the inverse transform
3. Transforming the camera to the desired pose in world coordinates
4. Computing each ray
5. Apply the inverse transform to the ray so that the transformed ray is in the original geometry's local coordinates
6. If there is a hit, transform the hit position and surface normal back to world coordinates
7. Determine the pixel color according to the ray hit

This approach allows a common object to be reused in a scene by tracking various transformations of the object in the scene. This allows for potentially better memory performance because only the inverse transform of the object needs to be stored per object instance, in addition to the one copy of the original object. Allows for simpler equations and may allow for indirect transformation of geometry equations that cannot be directly transformed.

## Transforming Surface Normals
When using object instancing, a complication arises for calculating surface normals since they cannot be transformed with the same transform that is applied to the surface. The result will not necessarily maintain orthogonality with the tangent plane of the surface. Given a surface with normal n and transformation M applied to the surface, we need to determine n prime (the new normal) such that the new normal is still orthogonal to the surface.
$$n' = (M^{-1})^Tn$$
This equation shows how to correctly transform a surface normal when transforming the related surface.

## Object Instancing Calculation
For a given ray:
$$ray = a + tb$$
- a is the ray origin
- b is the ray direction
- t is the input parameter
For a transformed ray:
$$ray'=M^{-1}a+tM^{-1}b$$
Then, the basic algorithm is:
![](../Images/Pasted%20image%2020260911182449.png)

![](../Images/Pasted%20image%2020260911182459.png)

**Note that t is the same in both local and global coordinates**. This is because the proportions are maintained after transformation. However, this is only true if the view/eye rays have not been normalized.

## Subdivision
Subdivision methods help us spatially organize and simplify geometry. There are two main types of subdivision: **Object Subdivision** and **Spatial Subdivision**.

### Object Subdivision
This method groups objects into disjoin sets, each enclosed by a bounding volume. The bounding volumes themselves can be Axis-Aligned Bounding Boxes, Oriented Bounding Boxes, or other bounding shapes.
![](../Images/Pasted%20image%2020260912102823.png)
Here you can see that boxes are placed over groups of other objects, therefore 'bounding' them.

### Spatial Subdivision
This method divides space into disjoint regions, and objects are assigned to intersecting regions. Uniform grids, kd-trees, and octrees are good examples of this.\
![](../Images/Pasted%20image%2020260912103027.png)

This provides efficient pruning of irrelevant regions during traversal. It is well suited for optimizing ray tracing by localizing ray-object tests.

#### Special Case: Decomposing Objects
This technique focuses on breaking a single object's geometry into smaller parts. One common approach is **Convex Decomposition.**
**Convex Decomposition:**
- Divides an object into convex components
- Useful for collision detection and simulation, as well as frustum/occlusion culling tests
- For example, a complex triangle mesh could be broken down into chunks

## Comparing Techniques
**Object Subdivision**:
- Groups entire objects into bounding volumes
- Flexible for varying scene distributions
**Spatial Subdivision**:
- Divides space itself into fixed or adaptive regions
- Handles dense scenes efficiently
**Convex Decomposition**:
- Special case for single objects
- Simplifies geometry for processing
The choice of technique depends on scene complexity and use case.

## Oriented Bounding Boxes (OBBs)
Oriented Bounding Boxes are bounding boxes that are rotated or skewed relative to the global axes. These differ from AABBs because they are not constrained to align with the global axes. You can define OBBs by a transform on an AABB, and store the inverse. Then you can transform the ray into the OBB's local coordinate system using the box's inverse transform matrix. If the inverse transform matrix is not already available however, direct OBB intersection algorithms may be more efficient since they avoid the computational cost of matrix inversion.

## AABB Generation
Generating AABBs the bound other geometries is common, especially for triangle meshes.
**Algorithm:**
1. Iterate over all points in the set
2. Compute $$B_x^{min}=min(x_1,x_2,...,x_n), B_x^{max}=max(x_1,x_2...,x_n)$$
3. Repeat for y and z to acquire the minimum and maximum bounds
This results in a tight fitting AABB for the given points. It works directly for triangle meshes by iterating over all vertices. 
![](../Images/Pasted%20image%2020260912104357.png)

## AABB for Parametrized Geometry
For shapes like spheres, cylinders, torii, Bezier curves, etc. you can use the known equations for that shape to compute the maximum and minimums (known extremes).
A sphere for example would use the following process:
- Given the center C (x,y,z) and radius r
- $$B_x^{min}=x-r, B_x^{max}=x+r$$
- Then follow the same process for y and z

For arbitrary parameterized geometry where it is less straightforward to calculate the extremes, there are two possible approaches. 
**Analytical Solution:**
- Use the parameterized equation of the geometry to compute exact bounds
- This often requires calculus:
	- Solve for critical points by setting partial derivatives to zero
	- Evaluate geometry at critical points and parameter bounds
	- Ensure the tightest possible bounding box
**Numerical Approximation:**
- Discretize the geometry into sample points
- Compute the min/max for x, y, and z from the sampled points
- This approach is simpler and computationally efficient but may over/under-estimate bounds
- Possibly expand the AABB by epsilon to account for error
- This approach is common in practical applications

When determining the AABB for a group of objects, we can follow a simple two step process:
1. Compute the individual AABBs
2. Merge AABBs by calculating the min and max values along each dimension across all objects to form the group's AABB

### Optimally Aligned Oriented Bounding Boxes
Bounding boxes can be transformed to minimize the bounding volume by aligning the box with the object it contains. This is useful for tightly bounding rotated or irregular shapes, but requires solving an optimization problem:
- For discrete point sets, often based on the covariance matrix of the geometry
- Eigenvectors of the matrix define optimal box orientation
- This concept is beyond the scope of this lecture so don't worry too much about it
Example Visualization:
![](../Images/Pasted%20image%2020260912105557.png)

## Application of AABBs for Ray Tracing
AABBs are used as a first pass filter to test ray intersection with geometry. If a ray does not intersect an AABB, then the ray cannot intersect anything enclosed by the AABB. If it does intersect, further intersection tests are necessary. This process reduces expensive ray-geometry intersection tests by quickly rejecting rays that miss the AABB.

AABBs also integrate seamlessly with various acceleration structures including Grids, BVHs, and kd-Trees. 
- Grids: AABBs quickly determine cell membership
- BVHs: AABBs act as bounding volumes for hierarchical subdivisions
- kd-Trees: AABBs define split regions and simplify ray traversal

## 3D Grids and DDA
A 3D grid is a spatial acceleration structure that divides the scene into voxels (volumetric pixels). They are used to optimize ray tracing by limiting intersection tests to relevant regions. Each voxel stores references to the primitives (objects) that it overlaps. 3D grid attributes are defined by the scene's bounding box and resolution (number of voxels per axis).

### Grid Construction
To construct a 3D grid take the following steps:
1. Compute the scene bounds and determine the overall bounding box of all primitives in the scene
2. Define the grid resolution by dividing the scene's bounding box into a regular grid of voxels
	1. The total number of voxels is proportional to the number of primitives
	2. A scale factor is applied to balance memory and efficiency
	3. The largest dimension of the bounding box determines the primary axis resolution, with others scaled for cubic voxel shapes
3. Assign primitives to all voxels that their AABBs overlap with
	1. AABB overlap ensures conservative assignment
	2. Avoids computationally expensive direct overlap tests with geometry primitives

Choosing the resolution is a critical step that needs special consideration. 
- Voxel size should be proportional to the average size of primitives to minimize empty space and overlap inefficiency
- A common guideline is to use a scaling factor of 3 times the cube root of the number of primitives for initial grid resolution
- Finer grids increase precision but demand more memory and computational resources, while coarser grids reduce efficiency for densely packed primitives

**Challenges with Voxelization:**
- Over Marking Voxels
	- Using AABBs may mark voxels that the primitive does not intersect, especially for thin or curved geometry
- Resolution Trade-Offs
	- Low resolution means that there are too many primitives per voxel, reducing traversal efficiency
	- High resolution is memory intensive and computationally expensive
- Numerical Precision Issues
	- Rounding errors in voxel boundary computations can cause incorrect marking
	- Parallel or grazing rays require careful handling

To optimize, Hybrid BVH Grids and Hierarchical Grids can be used.
**Hybrid BVH-Grid**:
- Combine Bounding Volume Hierarchies (BVHs) and grids for an adaptive resolution
- Dense regions use grids embedded within BVH nodes, while sparse regions are handled by hierarchical BVH partitioning
- This approach balances spatial adaptivity with efficient ray traversal
**Hierarchical Grids:**
- Dynamically adapt grid resolution based on spatial density
- Octrees are a specific case of hierarchical grids using binary subdivisions
- More flexible hierarchical grids allow arbitrary subdivisions to optimize for complex non-uniform scenes

Grids are simple and effective acceleration structures for ray tracing. They work best when voxel size is proportional to primitive size and the geometry distribution is uniform. Advanced techniques like adaptive grids or hybrid approaches can address non-uniform geometry or dynamic scenes.

## 3D Digital Differential Analyzer (3D DDA)
3D DDA is an efficient traversal algorithm on uniform 3D grids by a ray. 
- It is used in raytracing to determine ray-object intersections within a 3D grid
- Operates by stepping through cells along the ray's path
- Each cell contains a set of geometry that overlaps with it
- The ray is tested against the geometry in each cell it enters
![](../Images/Pasted%20image%2020260912112025.png)
This approach ensures that only the most relevant geometry is considered for intersection tests, which reduces computational cost significantly.

**3D DDA Algorithm**:
1. Initialize the ray and computer starting cell indeces
2. Precompute parameters - calculate step sizes and initial boundary distances
3. Traverse the grid - step through the cells along the ray's path
4. Stop traversal when the ray exits the grid or when an intersection is found that terminates the need for further traversal (like an opaque object)

**For step 1**, given the ray origin O and direction D:
$$i_x = \lfloor(O_x-B_x)/cellsize_x\rfloor$$
The same should be calculated for y and z to determine the rays starting grid coordinates. These calculations represent the distance needed on each axis to move from one cell to the next.

**For step 2**, step size and initial boundary crossings should be calculated:
$$tstep_x = cellsize_x/|D_x|$$
The same should be calculated for y and z.
Initial Boundary Crossings can be calculated with the following equations:
$$t_{next_x} = (boundary_x-O_x)/D_x$$
Then of course do the same for y and z. These represent the distance from the ray's origin to the first voxel boundary along each axis. The exact position of the nearest boundary depends on the direction of the ray. For negative directions, the boundary is located at i * cell size, and for positive directions it is (i+1) * cell size.

**For Step 3**, determine the next cell 
$$t_{min} = min(t_{next_x}, t_{next_y}, t_{next_z})$$
This makes sense because the minimum distance to the next axis would determine what cell gets stepped to next.
Then, update the relevant axis where ? is the dimension:
$$if t_{min}=t_{next_?}: i_?\pm 1, t_{next_?}=t_{next_?} + t_{step_?}$$
![](../Images/Pasted%20image%2020260912113345.png)

Note that you can't stop at the first intersection calculated in the cell. You must test all the geometry in the cell to see which is the closest hit (smallest t).

**For Step 4**, evaluate the exit conditions:
- Stop grid traversal if the ray goes off grid on any of the axes (if i<sub>?</sub> < 0 or i<sub>?</sub>>=N<sub>?</sub>)
- Check to see if the ray's extent is exhausted (rays can have a fixed range, so if it goes past that then exit)
	- $$t_{next_x},t_{next_y},t_{next_z} > t_{max}$$
- If the ray intersects geometry in the visited grid cell, test all geometry in the cell to calculate the closest hit (regardless of the initial hit).
## Bounding Volume Hierarchies
Bounding volume hierarchies efficiently accelerate ray-object intersection tests in ray tracing. The key idea is to hierarchically  partition objects into disjoint sets with bounding volumes. This is **Object Subdivision**, in contrast to Grids or kd-Trees which use **Spatial Subdivision**.
![](../Images/Pasted%20image%2020260912115128.png)

The structure of a BVH is a tree with two types of nodes: Leaf Nodes that contain primitives, and Interior Nodes that contain bounding boxes that enclose children.

BVHs are memory efficient, since each primitive appears only once in the hierarchy. The memory requirement is 2n-1 nodes for n primitives. It is also adaptive and handles irregular geometry distributions better than uniform grids.

## Constructing a BVH
Constructing a BVH involves organizing the scenes primitives into a hierarchical tree structure. This process is typically performed with a top-down greedy algorithm.

1. Partitioning
	- Recursive, top down greedy algorithm
	- Divides primitives into groups based on proximity
	- Create bounding boxes for each group
2. Split Methods
	- There are three common methods for determining how to split the primitives during construction: **Equal Count**, **Middle (Midpoint)**, and **Surface Area Heuristic (SAH)**
	- **Equal Count**: split according to count or spatially ordered nodes
	- **Middle (Midpoint)**: Split at midpoint position of bounds
	- **Surface Area Heuristic**: Split that minimizes surface area of the AABB
		- Minimizes the expected cost of ray traversal by creating AABBs with minimized surface area
		- Widely used because it produces more efficient trees at the cost of longer construction time
This method results in a balanced tree with adaptive bounding volumes.

### Basic Splitting Methods
Start with a single root node that encompasses all objects in the scene. Split from there based on spatial proximity recursively, creating bounding boxes for each group.
#### Equal Count (Median) Split Method
- Divide primitives into two groups by splitting at the middle index of the list of centroids, ordered along the selected axis
- Pick the axis with the largest spatial extent based on centroid positions
- This ensures a balanced tree structure but may result in unbalanced spatial bounds
- This is a common split method that is effective, particularly in scenarios where computational simplicity is prioritized over optimal space division
#### Middle (Midpoint) Split Method
- Divide primitives into two groups by splitting at the **spatial midpoint** of their centroids bounding box along the selected axis
- Pick the axis with the largest extend based on centroid positions
- Ensures balanced spatial bounds but may result in an unbalanced tree structure
- This method should be used when balanced spatial bounds are prioritized over a balanced tree structure

The advantages of both of these split methods is that they are simple to implement and generally have balanced trees or bounds. However, they each have a limited consideration of scene geometry: the equal count method does not consider spatial distribution while the midpoint method doesn't consider ray hit likelihood. Both methods may lead to higher total ray traversal and intersection costs compared to heuristics like SAH.

### Surface Area Heuristic
The surface area heuristic (SAH) is a widely used method in bounding volume hierarchy (BVH) construction because it considers both traversal costs and intersection likelihood. It turns out that imbalanced trees can reduce the overall cost of ray tracing by grouping high-likelihood regions more effectively. This is a key insight of SAH. 

SAH evaluates each potential split based on how it affects the number of traversal steps in an intersection test required during ray tracing. The likelihood of a ray hitting a bounding volume is proportional to the surface area of that volume. Producing smaller, bounding volumes, the SAH approach decreases the amount of unnecessary intersection tests.

SAH relies on several components to evaluate the cost of the potential splits during bounding volume hierarchy construction. One of the primary components is computing the surface area of the AABBs that enclose groups of primitives. This surface area is directly tied to the likelihood of a ray intersecting the bounding volume. Luckily computing the surface area of an AABB is simple:
$$SA = 2*(wh + hd + dw)$$
Where w, h, and d are width, height, and depth.

The second component of SAH is **intersection likelihood**. This quantifies how likely a ray is to intersect the **child** bounding volumes created during a split. To compute this, calculate the ratio of child AABB surface areas to the parent AABB surface area:
$$P_{left} = SA(A_{left})/SA(A_{parent}), P_{right} = SA(A_{right})/SA(A_{parent}), $$
These ratios calculate the probability of a ray intersecting each child volume. These can be used to evaluate the total cost of a split, which we aim to minimize. The total cost includes both the traversal cost and the intersection costs weighted by these probabilities.

### BVH Traversal
Bounding Volume Hierarchy Traversal is the process of navigating the hierarchy to identify ray object intersections during ray tracing. 
**Process:**
1. Check ray intersection with bounding volumes
2. Traverse only branches where the ray intersects the bounding volume
3. Test primitives in leaf nodes for intersections
This is efficient because it skips branches where the ray misses the bounding volume. It also minimizes redundant primitive intersection tests by pruning non-intersected branches. This approach is less sensitive to floating point precision errors compared to KD-trees, since traversal depends on bounding volume intersections rather than strict split planes.

### BVH Comparisons to Other Structures
- Grid:
	- Fixed spatial resolution
	- Simple to construct but inefficient for non-uniform distributions
	- Traversal can be expensive due to large number of grid cells
- KD-Tree:
	- Adaptive spatial partitioning using axis-aligned planes
	- High construction time due to precise plane based splitting
	- Generally faster traversal than BVHs for static scenes but less robust for dynamic ones
- BVH:
	- Groups objects hierarchically using bounding volumes
	- Intermediate build time that can be incrementally updated for dynamic scenes
	- Robust and efficient traversal, tolerating overlapping bounding volumes
	- Preferred for real time applications due to ease of refitting/rebuilding

## KD-Trees
KD-Trees are a special subdivision data structure commonly used in computer graphics. They are part of the binary space partitioning family (BSP), where space is recursively divided into smaller regions. 

KD-Trees are built by placing splitting planes that divide space into two subregions. While objects can straddle both sides of the split, the goal is to organize space to simplify queries and processing. All of the splits are axis aligned, which simplifies computations. Surface Area Heuristic can also be used to help choose split planes. 

During traversal of a KD-Tree, only a single ray box intersection test per node is needed. This makes them efficient for ray tracing as the tree structure allows rays to quickly skip irrelevant regions and focus on areas with geometry.

![](../Images/Pasted%20image%2020260912124625.png)

### Choosing Split Planes
The first thing to do is select the split axis:
- A simple method is cyclic selection, where the algorithm alternates through the axes in a fixed order such as x, y, z, and then repeats
- A more adaptive approach uses the widest dimension of the bounding box, ensuring splits better reflect the geometry's spatial distribution
Next, the split location must be determined:
- One common strategy is to split at the median of the primitive bounds, creating a balanced partition that works well for uniform distributions
- Alternatively, SAH can be used to consider the spatial extent and primitive distribution
	- This ensures that splits occur at meaningful locations relative to the geometry
Finally, numerical precision is critical for KD-Trees especially compared to BVHs, due to splitting space with planes and the need to robustly determine which side of the plane that objects belong to. Epsilon offsets from split planes might be used to decide which side of a plane geometry belongs to, which sometimes assigns geometry to both sides. Choosing effective split planes directly impacts the performance and accuracy of the KD-Tree, so these considerations are important for building an optimized data structure.

### Ray Traversal in KD-Trees
1. Split Plane Test
	- Test the ray against the splitting plane to determine traversal
	- Skip far branches if not intersected
2. Traversal Order
	- Visit the near child first
	- Process the far child only if necessary, updating t-bounds
3. Recursive Traversal
	- Focus on the near child
	- Visit the far child only if the near child lacks an intersection
4. Early Termination
	- Stop traversal if a valid intersection excludes other subregions
5. Leaf Node Intersections
	- Test geometry only in leaf nodes within the ray's subregion

### Numerical Accuracy in KD-Trees
Issues arise from comparing ray parameters and plane equations:
- Floating point rounding can cause incorrect split plane evaluation
- Precision issues when determining which side of the split the ray lies on
Mitigation strategies for the above issues inlcude:
- Careful considerations of epsilon values
	- During KD-Tree construction, offset split planes slightly away from geometry to avoid coinciding with AABB edges
	- During ray traversal, traverse both left and right child nodes when the ray is near coincident with the split plane
- Precomputing values for consistent comparisons
- Dynamic adjustment of epsilon based on node size or scene scale

### KD-Trees versus Bounding Volume Hierarchies
**KD-Trees**:
- Split space unambiguously into subregions using axis-aligned planes
- Objects may overlap multiple regions if they straddle split planes, potentially increasing memory usage and traversal steps
- Traversal is generally faster due to strict spatial partitioning
**BVHs**:
- Group objects hierarchically within bounding volumes, typically AABBs
- Bounding volumes may overlap, introducing some inefficiency during traversal
- Avoid object duplication by assigning primitives to a single node
- More numerically robust since they rely on primitive defined bounding volumes rather than strict plane-based splitting
- Often preferred for dynamic scenes due to efficient updates and rebuilds

