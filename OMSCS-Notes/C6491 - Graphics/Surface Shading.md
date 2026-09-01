Surface shading transforms flat geometry into visually compelling scenes. It enhances the perception of depth and material properties. We will focus on two core models: **Diffuse Shading** and **Phong Shading**, which simulate how light interacts with objects.

### Surface Normals
Surface normals are vectors that point perpendicularly out from a surface at a given point. It defines the orientation of the surface for shading and reflection calculations.

For **Sphere Normal Calculation**:
- A sphere is defined by its center C and radius R
- For a point P on a sphere
$$N = (P-C)/||P-C||$$
- Then normalize N to ensure it is a unit vector

For **Triangle Normal Calculation**:
- Triangles are defined by 3 vertices, V1, V2, and V3
- Compute two edge vectors relative to a common vertex:
$$E_1=V_2-V_1, \space E_2 = V_3 - V_1$$
- Then take the cross product of these edge vectors: N = E1 X E2
- Normalize N to ensure it is a unit vector
- The direction of N in this case depends on the vertex winding order (counter-clockwise means an outward-facing normal)
- **Ensure consistent winding order** to guarantee the normal has the correct orientation

For **AABB Normal Calculation**:
- If there was a hit detected, solve the ray equation with t0 to determine ray intersection point
- To determine the surface normal, whichever axis caused t0 to be updated identifies the surface plane
- Evaluate the sign of ray direction of identified axis to determine the surface normal:
$$N = -Sign(D[axis])\cdot \hat{e}_{axis} $$
- E hat axis is the unit vector corresponding to the axis
- For example:
$$D = [1,2,-3]^T, \space Axis:y, N = -Sign(D[1])\cdot [0,1,0]^T = [0,-1,0]^T$$
## Illumination
**Direct Illumination** is light that reaches a surface directly from a light source.
**Indirect Illumination** is light that includes contributions from reflections or scattering events before reaching a surface.
**Global Illumination** refers to the comprehensive simulation of light in a scene, including both direct and indirect lighting.

### Diffuse Shading Basics
Modeled using the **Lambertian Model**. This model represents matte surfaces like paper and unfinished wood. It follows **Lambert's Cosine Law**:
$$c = c_r \cdot c_l \cdot max(0,n \cdot l)$$
- cr is the surface reflectance
- cl is the light intensity
- n is the surface normal (unit vector)
- l is the direction towards the light (unit vector)

### Ambient Illumination
Ambient illumination adds a baseline illumination (c<sub>a</sub>) to simulate indirect light and crudely approximate global lighting effects. It often supplements diffuse direct lighting and keeps shadows soft. 
![](../../Pasted%20image%2020260901185257.png)
This method is common and computationally efficient, but doesn't capture reflections properly.

### Specular Reflections
Specular reflections occur when light reflects off a surface in a single dominant direction, like a mirror. It produces sharp specular highlights on smooth surfaces, that is visible when the view direction aligns with the angle of reflection of the light source off the surface.
![](../../Pasted%20image%2020260901185459.png)

### Implementing Diffuse and Specular Shading
- All vectors should be calculated in world space
- Diffuse shading relies on the surface normal *n* light direction *l*
	- back facing normals can be handled by using max(0, n * l)
- An ambient term is also usually added
$$c = c_r \cdot (c_a + c_l \cdot max(0,n\cdot l))$$
- cr is diffuse reflecance
- ca ambient intensity
- cl is the incoming light color
This improves realism by simulating global illumination effects, but does not enforce conservation of energy.

## Phong Shading
Phong shading models highlights on shiny surfaces like polished metal and tiles. It is a **direct** illumination method that incorporates reflection and viewer direction to simulate specular highlights.

**Phong Lighting Equation:**
$$c = c_r \cdot (c_a+c_l\cdot max(0,n\cdot l)+ c_p\cdot c_l \cdot max(0,e\cdot r)^p$$
- cp is the specular reflectance
- p is the phong exponent which controls the sharpness of the highlight
- e is the unit vector toward the viewer (likely from eye ray so e = - |D|
- r is the reflection vector

### Reflection Vector Calculation
$$r = -l+2(l\cdot n)n$$
This is derived geometrically from the light direction (l) and surface normal (n). It is used to determine alignment with the view direction (e dot r). 
Specular highlights occur when the viewer aligns with the reflection direction.
![](../../Pasted%20image%2020260901190627.png)

To derive the reflection vector r, we first need to decompose l, the light vector:
$$l=l_\parallel+l_\perp$$
- *l* parallel is the component parallel to n, computed via projection:
$$l_\parallel = (l\cdot n)n$$
- *l* perpendicular is computed as:
$$l_\perp=l-l_\parallel$$
From here, we can flip the perpendicular component and keep the parallel one unchanged:
![](../../Pasted%20image%2020260901191042.png)
This leads to the following equation:
$$r = -l+2(l\cdot n)n$$

### Blinn Phong Shading
This approach further optimizes Phong shading by replacing the reflection vector with the **halfway vector** (h), efficiently approximating the specular component.
$$h = (l+e)/||l+e||$$
$$c = c_r \cdot (c_a+c_l\cdot max(0,n\cdot l)+ c_p\cdot c_l \cdot max(0,n\cdot h)^p$$
This avoids the need to compute the reflection vector. Visually, the specular highlight is softer than normal Phong shading which is sometimes desirable.

## Multiple Light Sources
The superposition principle easily describes that the total light contribution is the sum of the effects from all individual light sources. This allows us to extend the shading model to handle N point light sources. Applied to Blinn Phong shading we get the following:
$$c = c_r \cdot c_a+\Sigma^N_{i=1}[max(0,n\cdot l_i)+c_p\cdot c_{l_i}\cdot max(0,n\cdot h_i)^p )]$$
## Distance Based Light Attenuation
Real world light sources lose intensity with distance. Without attenuation, distant lights appear unrealistically strong. Attenuation simulates spreading and absorption of light by the environment.

**Linear Attenuation** is good for simple artificial light sources like lamps:
$$A = 1/d$$
**Quadratic Attenuation** is good for physically realistic artificial lights. It is often used in real-time engines like OpenGL and Unity. The coefficients a, b, and c are chosen to control falloff.
$$A = 1/(a+bd+cd^2)$$
To include this in the Blinn Phong model, we modify each lights contribution by the attenuation factor Ai:
$$c = c_r \cdot c_a+\Sigma^N_{i=1}A_i[max(0,n\cdot l_i)+c_p\cdot c_{l_i}\cdot max(0,n\cdot h_i)^p )]$$
In practice we measure the distance to each light and use that to calculate attenuation. Choose proper coefficients (a,b,c) to avoid sharp cutoffs. Some engines clamp minimum light or use smoother falloff curves. Artistic control often outweighs physical realism.

## Direct Light Shading in Ray Tracing
Both models (diffuse and phong shading) are evaluated at ray-object intersection points.
- Diffuse shading requires surface normal and light direction at the intersection. 
- Phong shading adds computation for the reflection vector and view direction
By combining these models ray tracing captures a realistic light model, representing both diffuse and specular interactions between light and surfaces.

## Diffuse vs Phong Shading
- Diffuse Shading:
	- Simple and computationally efficient
	- Suitable for matte surfaces
- Phong Shading:
	- Extends diffuse shading
	- Adds realism with specular highlights
	- Higher computational cost due to reflection vector and exponentiation, but this can be improved with the Blinn-Phong variant