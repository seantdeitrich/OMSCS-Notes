# Ray Tracing Introduction
## Rendering
There are two fundamental approaches to rendering:
- Object-Order Rendering 
	- Considers each object in a scene individually
- Image-Order Rendering
	- Considers which objects in a scene contribute to each pixel in the image
Rendering must consider direct and indirect lighting, physical properties of objects, and then consider how that information renders to an specific viewport.

### Object Order Rendering
In Object Order Rendering we consider the contribution of each object to the view of the scene from the perspective of the observer. We project 3D features of the environment onto the camera / eye.
- Once we know where the object maps to on an image, we can then paint the appropriate image considering where the object is in relation to light sources
- In order to properly do object order rendering, you must consider which objects or parts of objects are in front of others relative to the camera
	- This can be done by comparing distances of objects to the cameras
	- **Painters Algorithm** and Z-Buffer Rendering are the most common solutions to this

**Painter's Algorithm** sorts all objects in the scene by the depth from the camera and paints in order from the farthest to the nearest. Objects further away are painted first, and closer objects are painted on top of them.

**Z-Buffer** maintains a depth buffer that records the depth of the closest objects at each pixel. As each object is processed its depth is compared to the current value in the buffer, and the new image is rendered only if the new value is closer.

### Image-Order Rendering
Analyzes each point in a cameras image plane, and then determining which objects contribute to each point. The camera view is split up into a raster (grid of pixels), and we can determine the path that light would take from the scene to that pixel. Whichever object surface is closest to the camera along the path will be the one used.
- All objects in the scene must be considered for each pixel, which leads to problems with efficiency and scaling
- More computationally expensive, but is better at accurate and fine grained light rendering
- Generates images that are potentially more accurate than Object-Order Rendering
- Ray Tracing is an example of Image-Order Rendering
- Known for being computationally expensive

## Ray Tracing Concepts
### Rays
Rays are situation vectors that originate from the camera or eye. They are referred to as viewing rays, primary rays, or camera rays. They are directed according to path from pixel sensor of camera or eye to the scene geometry. 

### Process
1. Ray Generation
2. Ray Intersection
3. Shading

**Ray Generation** determines the ray based on camera view per pixel. 
**Ray Intersection** determines which rays intersect geometry in the scene. Intersection position and surface normal are also calculated.
**Shading** assigns a pixel value according to the surface properties of the object and illumination.
```pseudocode
for each pixel do
	compute viewing ray
	find first object hit by ray and its surface normal n
	set pixel color to value computerd from hit point, light, and n
```

Real world cameras require a pinhole or lens to be between the sensor and the scene, which inverts the image. A virtual camera can be simplified conceptually. The mathematics are also simplified since we do not need to worry about inverting the image.

### Ray Generation
A view ray can be described with a vector -
- Assume the camera / eye position is known
- We know a point on the image plane that corresponds with a pixel
- This view ray is a relative vector from the eye to the pixel, however this only gives us a vector, and we need a situated vector (a vector at a certain location in space)

#### Math for Ray Generation
- u<sub>i</sub> is the center coordinate of the x pixel, and v<sub>j</sub> is the center coordinate of the y pixel
- $$u_i = l + (r-l)(i+0.5)/n_x$$
- $$v_j = b + (t-b)(j+0.5)/n_u$$
- Solve for the ray by computing:
	- Ui and Vj
	- The Ray Vector: $$-dw+u_iu+v_jv$$
	- The Ray Origin: e
- $$S_i,_j = e - dw + u_iu+v_jv$$
### Parametric Line Equation for Rays
- E is for Eye
- S is for Screen
$$ P(t) = E + t(S-E)$$
$$P(0) = E $$
 $$P(1) = S$$
 - Interpolation occurs when 0<=t<1, and extrapolation occurs when t > 1 or t<0
 - If 0 < t<sub>1</sub> < t<sub>2</sub> then P(t<sub>1</sub>) is closer to the eye than P(t<<sub>2</sub>)
 - P(t<sub>i</sub>) = l (intersection with some geometry)

### From Camera Pose and Field of View
- Assume a Left Handed Coordinate System (LHCS)
- Camera is positioned at e, looking in direction w
- The camera frame uses:
	- **u**: right vector
	- **v**: up vector
	- **w**: forward vector
	- Note that these axes are different than the scene or world axes, since we're considering the way the camera is facing
- ![](../Images/Pasted%20image%2020260831195949.png)
- The **View Plane** is divided into a grid of **W**x**H** pixels
- To map from pixel coordinates to camera space, we first center the pixel grid around the middle of the screen. This is done by subtracting half the width from i (the horizontal pixel coordinate) and half the height from j (the vertical pixel coordinate)\
- For example, for a 1920x1080 image the center is (960,540)
	- A pixel at (1200,300) becomes:
	- i - W/2 = 1200-960 = 240
	- j - H/2 = 300-540 = -240
	- So that pixel at (960,540) on the screen is (240, -240) which is 240 pixels to the right and 240 pixels above the screens center
- The important distinction here is the u, v, and w are the camera's 3D direction vectors while i and j are 2D pixel coordinates used to determine where a particular pixel lies on the view plane.
#### Scaling to World Space
- Now that we've centered the pixel coordinates, we scale them to match the size of the view plane in world space.
- Place the view plane 1 unit in front of the camera along w, the forward vector of the camera
- The vertical field of view (FoV) is theta
	- k is the vertical half height of the view plane
	- $$k = tan(\theta/2)$$
- If the screen is square, the view plane dimensions are: Height = 2k, Width = 2k
- ![](../../Pasted%20image%2020260831202037.png)
- Now map pixel coordinates to world space:
- $$u = (i - W/2) \cdot 2k/W $$
- $$v = (j - H/2) \cdot 2k/H $$
- The result of this is that each pixel is assigned a u and v coordinate that we can use to compute its corresponding ray direction.
- Aspect ratio is needed here because most screens are not square
- **Aspect Ratio** = W/H
	- If A > 1 then the image is wider than it is tall (widescreen)
	- If A < 1 then the image is taller than it is wide (portrait mode)
- We want to keep vertical FoV fixed and scale horizontally to match
- Recall that vertical FoV theta gives vertical slice k = tan (theta/2)
- View plane height = 2k, and we multiply by A = W/H
- View plane width: 2k*A
- This way the view frustum expands horizontally on wider screens, while keeping vertical coverage the same
#### Mapping to World Space with Aspect Ratio
$$u = (i-W/2)\cdot2k/W$$
$$v=(j-H/2)\cdot2k/H$$
However, this approach assumes we're firing rays from the corners of pixels. For better accuracy we shift i and j each to the pixel center.
$$u = (2(i+0.5)/W-1)\cdot A \cdot k$$
$$v = (2(j+0.5)/W-1)\cdot k$$
Here, the direction of the v coordinate needs to flip because in world space we want v to increase upward, but most screen coordinates measure downwards as the positive direction.
Refactoring that equation leaves us with:
$$v = (1-(2(j+0.5)/H)$$
Ray direction is then calculated with:
$$d = u\cdot \mathbf u + v \cdot \mathbf v + d \cdot \mathbf w$$
- It's common not to normalize this vector, since it's a performance hit and has no impact on ray intersection
- Note that *d* in the above equation is just 1, since it is the distance from the eye to the camera

#### Calculating Horizontal Field of View
Typically, vertical FoV is fixed since monitors vary more in width than in height.
To calculate the angle of the horizontal field of view, use the following equation:
$$\theta_h = 2 \arctan(tan(\theta_v/2)\cdot(W/H)$$
Software ray tracers must choose a primary FoV axis
- If the vertical FoV is chosen, do not scale v-axis by the aspect ratio
- Apply aspect ratio to the u (horizontal) axis
- This preserves the intended vertical field of view

### Ray Intersection with Spheres
Two solutions are possible, but the one closer to the eye (smaller t) should be used for rendering. One solution and no solutions are also possible.
- Using the radicand portion of the quadratic equation (b<sup>2</sup> - 4ac) we can determine how many solutions there are
	- No solution if the discriminant is less than 1
	- One solution if the discriminant equals 0
	- Two solutions if the discriminant is greater than 0
Solving for t gives us the following equation:
$$t = (-D\cdot(E-C)\pm \sqrt(D-(E-C))^2 - (D\cdot D)((E-C)\cdot(E-C)-r^2))/(D\cdot D)$$

### Ray Intersection with Planes
Unlike spheres, there is either a single solution or none.
$$t=(P_1-E)\cdot N/D\cdot N$$
- Note that it doesn't matter if N (the normal vector of the plane) is normalized or not
- Watch out for division by 0 - If D * N is 0, then consider the ray to miss the plane
- A negative t also misses the plane
- Solve for P, where the intersection occurs
- Ignore back face collisions, where D * N > 0
	- The scene likely only has shading models for front facing surfaces

### Ray Intersection with 3D Triangles
**Ray Plane + Point Half Plane Test**
1. Given a triangle, determine the equation of the plane that contains it
	1. Use the three vertices of the triangle
	2. Determine the plane normal from triangle vertices
2. Compute the ray intersection with the plane
	1. If a solution is found, this guarantees that the point of intersection is in the same plane as the triangle
3. Determine if the point of intersection is inside the triangle
	1. Iterate through each triangle edge testing Left() predicate for point of intersection

Ray Plane + Barycentric Coordinates (possibly with Cramer's Rule) and Moller Trumbore algorithms can potentially give better performance.
### Ray Intersection with Boxes
Axis Aligned Bounding Boxes (AABB) are defined by their minimum and maximum corners. A point P is inside the box if for each dimension Bmin <= P <= Bmax. These can also be thought of as the intersection between three slabs.

A slab is the space between two parallel planes aligned with a coordinate axis.

For each slab, compute the intersecting ray's t-values for the two bounding planes:
$$t_{near} = (B_{min}-O)/D,\space t_{far}=(B_{max}-O)/D$$
- Ensure tnear <= tfar, and swap if needed
- Start with t0,t1 = 0,**∞**
- For each slab, update t0 to be the max of (t0,tnear), and t1 to the min of (t1,tfar)
- If t0 > t1, the ray misses the box
- The algorithm is applied to each dimension for x, y, and z
- Test if each axis interval tnear, tfar overlaps with the others
	- If so, the ray intersects, otherwise it does not
If there is a hit, we can solve the ray equation with t0 to determine the intersection point.
To determine the surface normal, whichever axis caused t0 to be updated identifies the surface plane.
Evaluate sign of ray direction of identified axis to determine the surface normal:
$$N = -Sign(D[axis])\cdot \hat{e}_{axis} $$
- Note that eaxis is the unit vector corresponding to the axis
#### Computing t-Values for Slabs
- For each axis aligned slab, calculate where the ray intersects the two bounding planes, ensuring that the slab normals are the same direction.
- The general plane equation is given by: ax + by + cz + d = 0
- (a,b,c) is the normal vector of the plane, and d is the plane constant
$$T_x = (B_x  - O_x) / D_x$$
Bx is the slab boundary determined from plane constant d as:
$$d = -B_x \space from \space d = -D_{plane}|N|$$

Similar equations for the y and z slabs.

#### Special Cases and Numerical Robustness in AABB Intersections
**Parallel Rays** are the special case where D = 0. If the rays origin is not within the slab bounds, then the ray misses the box. 
Parallel Rays can also introduce Bx - Ox = 0 cases leading to 0/0 producing NaN. This case must be explicitly handled, where all comparisons to NaN resolve to false.
**Ray Origin Inside the Box** is another special case to consider, where t0 is 0.
**Swapping tnear and tfar**: If D<0, swap near and far values to ensure tnear <= tfar

### Ray-Box Intersection Advantages
Ray-Box intersections are advantageous due to their:
- Efficiency with few arithmetic operations per axis
	- Faster than most ray-geometry intersection tests, except for spheres
- Simplicity with simple parametric ray equations
- Extensibility with easy integrations into acceleration structures like grids of BVHs
- Better Bounding Volume which is ideal for bounding many geometries due to their tight fit and simplicity



