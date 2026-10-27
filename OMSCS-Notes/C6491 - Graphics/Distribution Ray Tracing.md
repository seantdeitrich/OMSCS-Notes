Distribution Ray Tracing was created to address the unrealistic look of traditional ray tracing. The purpose is to replicate physical phenomena such as soft shadows, depth of field, and motion blur which are absent in traditional methods. 

Distribution Ray Tracing is an extension of traditional, where multiple rays are shot out with small variations. The visual impact of these rays is then averaged to determine the final color.

## Aliasing 
Aliasing is the distortion or artifacts that occur when a signal is sampled at too low a resolution to accurately represent  its details. The **Nyquist-Shannon Sampling Theorem** states that to avoid aliasing, the sampling rate must be at least twice the highest frequency in the signal. 

## Super Sampling 
Super sampling reduces aliasing by sampling multiple points within each pixel. The final pixel color is the average of the sampled contributions. 

**Stratified or Grid Sampling** involves dividing each pixel into an n x n grid and taking the average of those pixels. This is straightforward to implement and works effectively for smooth scenes with minimal detail. However, it can introduce moiré patterns due to alignment with scene geometry.

**Random Sampling** instead places sample points randomly within each pixel. This more closely mimics real world phenomena and produces better results. However, it can appear noisy unless a high number of samples are used. This approach is suitable for complex scenes with high frequency details or irregular patterns.

**Jittered Sampling** combines these two approaches by using a sub pixel grid and then placing samples randomly in each cell. This balances the strengths and weaknesses of the other approaches but is slightly more complex to implement.

The same approach can be used for soft shadows, depth of field, glossy reflections, and motion blur.
- For **soft shadows**, distribute samples across the area light to simulate penumbra
- For **depth of field**, distribute samples across the lens aperture for focus effects
- For **glossy reflections**, perturb reflection vectors within a region to simulate blur
- For **motion blur**, distribute samples over time intervals to simulate motion


## Disk Rejection Sampling
Disk rejection sampling is used to sample random points within a circular area light. Candidate points are generated in a bounding square around the disk, and points are accepted if they pass a squared distance test compared to the center and radius of the circle. This method helps with soft shadows.
![](../Images/Pasted%20image%2020261001184009.png)

For spherical light fixtures, we can orient the disk perpendicular to the direction from the sphere center to the surface point (aka use the front half of the sphere).

## Depth of Field
Depth of field is the range of distances within a scene that appear acceptably sharp in an image. It is determined by:
- Aperture size
- Focal length
- Distance to subject
Objects outside the sharp range appear blurred. The quality of the blur is called **bokeh** and is influenced by the aperture's shape and characteristics. With distribution sampling, depth of field has two components:
- Focus Plane - the distance where objects appear sharp. The conventional image plane is collocated with the focus plane.
- Aperture Square - this controls the blur strength with configurable size. It is located at the camera origin to simulate aperture sampling.

### DoF Rendering Process
1. Position the focus plane and aperture square
	1. Place the focus plane at the desired view distance where objects should appear sharp
	2. Position the aperture square at the camera origin
2. Configure the Image Plane
	1. Collocate the image plane with the focus plane
	2. Adjust the size to achieve the desired field of view
3. Size Aperture Square for desired Depth of Field effect
	1. Smaller apertures lead to sharper focus, larger aperture leads to increased blur
4. Randomize Ray Origins
	1. Generate random offsets within the aperture square at the camera origin
5. Aim rays toward the image plane
	1. Direct each perturbed ray origin toward the corresponding point on the image plane
6. Trace rays through the scene
	1. Evaluate intersections and accumulate contributions for shading
7. Average the contributions of the rays

![](../Images/Pasted%20image%2020261001185924.png)

## Glossy Reflections
Glossy reflection is where light is scattered in a controlled manner around the ideal reflection direction, resulting in highlights that appear soft or blurred. This most often occurs in coarsely polished metal, frosted glass, and satin paint.
Glossy reflections are simulated by perturbing rays around the ideal reflection vector. A square plane perpindicular to the reflection vector is used to distribute rays. The squares size (a) or its distance from the ray origin is adjusted based on surface roughness or scattering properties. This provides a simple approximation of blurred or hazy reflections. 
![](../Images/Pasted%20image%2020261001190451.png)

### Generating the Sampling Plane
- First an orthonormal basis is constructed for the plane
	- Set w to r (the ideal reflection direction)
	- Set u and v perpendicular to w and each other
- Sample random offsets within the square and use them to perturb the reflection direction
- The square's size or distance controls the spread of the reflection rays
	- For surfaces with high roughness, the square is larger or closer, leading to broader scattering and more diffuse reflections
	- For surfaces with lower roughness, the square is smaller or further from the ray origin resulting in tighter scattering and glossier reflections,
- This is less expensive than generating a random rotational transform for each perturbed ray
- Rays perturbed below the surface are physically invalid, so we discard those rays or any returning zero color

## Translucency
Translucency is modeled similarly to glossy reflection, but rays are perturbed around the refraction vector instead of the reflection vector. This simulates the scattering of light as it passes through a rough or translucent surface.
![](../Images/Pasted%20image%2020261001191646.png)

## Motion Blur
Motion blur is implemented with the following approach:
1. Assign a random time to each ray
	1. Randomly sample time within the defined exposure range
	2. The number of time samples can influence the quality of the motion blur (more samples leads to a smoother blur)
2. Interpolate object positions
	1. Use the rays sampled time to compute the object's position and orientation at that moment
	2. Interpolate based on the objects velocity, acceleration, etc.
3. Trace rays against objects 
	1. Evaluate ray intersections using the interpolated positions of the objects at the sampled time
	2. Combine the results of all rays over the exposure interval to produce the motion blurred effect
![](../Images/Pasted%20image%2020261001192314.png)

## Simultaneous Distribution Effects 
All of the above effects can be implemented simultaneously. Each technique requires sampling from specific distributions such as time, aperture, area light, or material roughness. Combining these effects enhances realism in rendered images. However, naive sampling strategies can lead to exponential growth in ray counts which is not computationally feasible. Instead of independent sampling, a set number of samples are shared across all effects. For example, 50-100 rays per pixel may suffice by distributing samples across each effect. This approach balances quality and computational cost. Additionally, some rays can help contribute to multiple effects at the same time, reducing redundancy in sampling. 