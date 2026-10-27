Our current method of ray tracing is good for simple light paths, but is unable to account for shadows, refractions, etc. Rendering more realistic scenes requires accounting for additional light paths including:
- **Occlusion Effects** - How objects block light and cast shadows
- **Indirect Illumination** - How light interacts with other surfaces before reaching the viewer (e.g. reflections and refractions)

The goal is to extend our framework to handle **Secondary Rays**, which will enable us to:
- Trace shadows
- Simulate reflections
- Handle refractions

## Shadow Rays
Shadow rays determine whether a point on a surface is illuminated or in shadow. Once a ray has an intersection, a shadow ray can be drawn from the surface to the light source. If the shadow ray intersects another object before reaching the light, the surface is in shadow for that light source. 

Shadow rays are cast from the intersection point toward the light source.
The mathematical representation is:
$$r(t)=p+tl, t\in[\epsilon, t_{max}]$$
- Epsilon is a small offset to avoid self intersection, and t<sub>max</sub> ensures the ray stops at the light source.
![](../Images/Pasted%20image%2020260916185930.png)

## Direct Self Shadow
Direct self shadow can quickly be determined before spawning a shadow ray. This occurs when a surface normal (n) of a surface faces away from a light source direction (l). Self shadow occurs when:
$$n\cdot l < 0$$
If the above is true, the surface faces **away** from the light.
This eliminates unnecessary shadow ray computations for back-facing surfaces. In other words, if we already know the surface is facing away from the light that means it will be in shadow, so don't bother casting a shadow ray.

## Shading Adjustment with Shadows

$$c = c_r \cdot c_a + \sum_{\text{lights}} \left[
\text{if not in shadow: }
c_r \cdot c_{l_i} \cdot \max(0, \mathbf{n} \cdot \mathbf{l}_i)
+ c_p \cdot c_{l_i} \cdot \max(0, \mathbf{n} \cdot \mathbf{h}_i)^p
\right]$$
For points in shadow, we exclude the diffuse and specular contributions from the light source. This ensures the light does not incorrectly illuminate shadowed areas. Ambient lighting is always included to prevent total darkness. This prevents areas in shadow from appearing completely dark.

## Self Intersection
Self intersection occurs when the intersection point p is incorrectly considered occluded by the object itself, causing unwanted shadow artifacts. Offsetting the shadow ray's origin slightly along the ray or surface normal helps alleviate this issue. The offset (epsilon) must be chosen carefully to balance precision and artifacts.
![](../Images/Pasted%20image%2020260916191645.png)
Additionally, the object ID (or a flag) can be used to ignore the particularly geometry that we're casting the shadow ray from. However, this approach is only reliable for convex objects.

## Hard Shadows
Hard shadows from point light sources typically appear unrealistic in games or simulations. Soft shadows with area lights are more realistic, and can be achieved by casting multiple shadow rays to sample an area light source. This will be revisited with **Distribution Ray Tracing**.
![](../Images/Pasted%20image%2020260916192045.png)

## Reflection
Reflection (specifically ideal specular reflection) occurs when light bounces off a surface, changing direction according to the **angle of incidence**. The angle of incidence equals the angle of reflection measured relative to the surface normal. Reflection rays are traced from the surface point in the reflected direction to determine what the surface 'sees' in the environment.

To calculate the reflection vector:
$$R = V - 2(V\cdot N)N$$
- R is the reflection vector
- V is the incoming ray direction (toward the surface)
- N is the surface normal at the point of intersection
![](../Images/Pasted%20image%2020260916192423.png)

### Implementing Reflection
1. Compute the reflection vector using:
	- $$r = d-2(d\cdot n)n$$
	- d is the direction of the incoming ray
	- n is the surface normal
2. Offset the reflection ray origin to avoid self intersection:
	- $$r_{origin}=p+\epsilon n, or \space r_{origin}=p+\epsilon r$$
	- Epsilon is the small offset along the direction of the surface normal or reflected ray
3. Make a recursive call to the ray tracing function to compute the color of the reflected ray:
	- $$c = c + k_m\cdot raycolor(r_{origin},\epsilon,\infty)$$
	- k<sub>m</sub> is the specular reflection coefficient (RGB)
	- raycolor is the function that recursively traces rays
4. Limit recursion depth to avoid infinite loops in closed environment

## Practical Considerations in Ray Tracing
**Energy Loss:**
- Real world surfaces lose energy during reflection
- The reflection may favor certain colors such as gold reflecting yellow more than blue
- The above means that different materials have different color biases, and reflections should account for material properties to achieve realistic results
**Efficiency:**
- Generate reflection rays only if k<sub>m</sub> (the specular coefficient) does not equal 0 (it's non black)
- If k<sub>m</sub> is black, the surface does not reflect light and no additional rays are needed
**Recursion Depth:**
- Set a maximum recursion depth to terminate reflections in complex environments, such as closed rooms
- Avoid infinite loops or very long computation

## Refraction
Refraction is the bending of light as it passes from one medium into another with a different refractive index. This phenomenon occurs because the speed of light changes depending on optical density of the medium it is travelling through. 

A **Dielectric** is a type of non-conductive material that does not have free electrons to support electrical conductions. **Transparency** arises because dielectric materials have a bandgap too large for visible light photons to excite electrons. Most visible light passes through the material without absorption, undergoing bending (refraction) as it enters the material, and partial reflection at the surface. This is important to consider when simulating glass, water, and crystals in ray tracing.

**Index of Refraction (IOR)** is the ratio of the speed of light in a vacuum to the speed in a given medium or material.
$$IOR = c_{vacuum} / c_{material}$$
For example, in a vacuum the IOR is 1, and in water the IOR is 1.33.

### Snell's Law and Refracted Rays
$$nsin(\theta) = n_tsin(\phi)$$
Snell's law describes the relationship between angles of incidence (theta) and transmission (phi) at a boundary between to media. It depends on the indices of refraction (IORs, represented here as n and n<sub>t</sub>).

### Formula for Refracted Rays:
$$

\mathbf{t}
=
\frac{{n}}{n_t}
\left(
\mathbf{d} - \mathbf{n}(\mathbf{d}\cdot\mathbf{n})
\right)
-
\mathbf{n}
\sqrt{
1 -
\frac{n^2}{n_t^2}
\left(
1-(\mathbf{d}\cdot\mathbf{n})^2
\right)
}

$$
- d is the incoming ray direction
- **n** is the surface normal
- t is the transmitted (refracted) ray direction
- *n,n<sub>t</sub>* are the indices of refraction for the current and transmitted media
**Don't get mixed up by the two/three different n's.**

When implementing this formula **ensure consistent vertex winding and use the dot product with the surface normal to determine the ray is entering or exiting the medium.** 

### Total Internal Reflection (TIR)
Total internal reflection occurs when no refracted ray is possible because the angle of incidence exceeds the critical angle:
$$sin(\theta_c)={n_t}/{n}, \theta_c=arcsin({n_t}/{n})$$
In this case, all light is reflected internally, but refracted rays may still escape after subsequent bounces within the medium. 
![](../Images/Pasted%20image%2020260916194805.png)

Total Internal Reflection is identified when the radicand in the refracted ray equation becomes negative. The formula for the radicand is:
$$1 - \frac{n^2}{n_t^2}(1-(d\cdot n)^2$$
To handle this in ray tracing, we can skip generating refracted rays for total internal reflection and focus on processing the reflection ray. For materials like diamonds, consider continuing refracted rays within the medium to simulate brilliance and light escaping through other surfaces.

### Implementing Refraction with Snell's Law
1. Compute cos(theta) using:
	- $$cos(\theta) = d\cdot n$$
2. Derive the ray refraction equation radicand (cos^2 phi):
	- $$cos^2(\phi) = 1 -\frac{n^2}{n_t^2}(1-cos^2(\theta))$$
3. Precheck for Total Internal Reflection (TIR)
	- If the radicand in the refracted ray equation is negative, TIR occurs and possibly no reflected ray is computed. Either terminate or compute the reflection ray
4. Compute the refracted ray direction:
	- $$t = \frac{n}{n_t}(d-n(d\cdot n)) - n\sqrt(1 - \frac{n^2}{n_t^2}(1-(d\cdot n)^2)$$
5. Adjust normals and indices of refraction:
	- Flip normals if the ray is exiting the medium
	- Swap n and n<sub>t</sub> as needed

### Numerical Precision
When n and n<sub>t</sub> are close to being equal, or when **dn** is close to zero, floating point errors can occur. 

## Fresnel Effect
The **Fresnel Effect** describes how the proportion of reflected vs. refracted light changes depending on the viewing angle at the interface between two materials. At shallow viewing angles, reflections dominate as the refracted component diminishes. This effect is useful for rendering materials like water, glass, and polished metals, and realistically captures how reflections change naturally with viewing angles.
![](../Images/Pasted%20image%2020260916200102.png)

### Fresnel Effect Equations
For light incident at an interface between two materials:
$$R_s = \left( \frac{n \cos\theta_i - n_t \cos\theta_t}{n \cos\theta_i + n_t \cos\theta_t} \right)^2$$

$$R_p = \left( \frac{n_t \cos\theta_i - n \cos\theta_t}{n_t \cos\theta_i + n \cos\theta_t} \right)^2$$

$$R(\theta) = \frac{R_s + R_p}{2}$$
- R<sub>s</sub> is the reflectance for **s-polarized** light
- R<sub>p</sub> is the reflectence for **p-polarized** light
- Theta<sub>i</sub> is the angle of incidence
- Theta<sub>t</sub> is the angle of refraction, calculated using Snell's Law

R(Theta) represents the proportion of light reflected at a surface as determined by the Fresnel effect. This reflectance depends on the angle of incidence between the incoming ray and the surface normal. 

Typically we don't calculate this directly because of the cost of trigonometric functions like cos / sin.

### Schlick Approximation
$$R(\theta) = R_0 + (1 - R_0)(1 - \cos\theta)^5$$

$$R_0 = \left( \frac{n_t - n}{n_t + n} \right)^2$$
- R<sub>0</sub> is the reflectance at normal incidence and is calculated using the refractive indices of the two media at the surface boundary
These formulas provide a more efficient way to estimate the Fresnel reflectance. In Blinn-Phong shading, this can be used to modify specular reflection to account for angle dependent reflectants. It is also instrumental in recursive ray tracing where it calculates the ratio of reflectant to transmittance at each interaction, determining how much light is reflected vs refracted.
$$Reflectance: R(\theta)$$
$$Transmittance: 1-R(\theta)$$