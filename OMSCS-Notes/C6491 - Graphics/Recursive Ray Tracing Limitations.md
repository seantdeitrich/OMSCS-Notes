Recursive ray tracing captures reflection, refraction, and shadows but has limitations. Complex lighting effects such as indirect illumination and caustics require more advanced techniques. Heckbert's path notation provides insight into light transport ray tracing constraints. Global illumination methods are needed for simulation of indirect lighting. 

## Heckbert's Path Notation
**Light Interactions**:
- L represents the light source (emission)
- S represents specular reflection or refraction 
	- Refraction is sometimes distinguished as transmission (T)
- D represents the diffuse reflection
- E represents the eye or observer
**Operators**:
- I* represents the Kleene Star which is a repetition operator
	- Zero or more occurences of the preceding interaction
- I+ represents the Kleene Plus which is another repitition operator
	- One or more occurences of the preceding interaction
- I^n represents n occurences of the preceding interaction
- The vertical bar represent an OR operation, allowing alternative paths

### Examples
The path LDE represents direct illumination from the light source without any additional actions
![](../Images/Pasted%20image%2020261001195324.png)

The path L(DS)E represents both diffuse and specular properties![](../Images/Pasted%20image%2020261001195339.png)

The path LDSSE represents the light undergoing two specular reflections before reaching the bowling ball. The ball then diffusely reflects the light towards the source.
![](../Images/Pasted%20image%2020261001195500.png)

Recursive Ray Tracing handles light paths involving:
- **Directly Illuminated** 
	- Diffuse Surfaces (D)
	- Specular Reflections (S)
	- For example: L(DS)E
- Reflected or refracted eye rays leading to directly illuminated surface
	- For example LDS * E
However, some paths are not possible and the ray tracer relies on direct illumination models without integrating over all incoming directions.
For example, recursive ray tracing cannot effectively handle the situation where light bounces diffusely twice to reach the light source:
![](../Images/Pasted%20image%2020261001200042.png)

Diffuse surfaces scatter light in all directions, effectively acting as secondary light sources. To compute their contribution accurately light arriving from all directions in the hemisphere above the surface must be considered. Recursive ray tracing only accounts for direct lighting from primary light sources. It ignores contributions from indirect lighting. Indirect specular illumination (that hits a diffuse surface afterwards) is also a case that recursive ray tracing does not support:
![](../Images/Pasted%20image%2020261001200610.png)

### Caustics
Caustics are the patterns of concentrated light that occur when light rays are reflected or refracted by a curved or transparent surface, such as water, glass, or polished metal, creating bright and complex patterns on nearby surface:
![](../Images/Pasted%20image%2020261001200924.png)
![](../Images/Pasted%20image%2020261001201047.png)

In indirect specular illumnination (LS+DE) light reflects or refracts off or through a sequence of one or more specular surfaces S+. It then hits a diffuse surface D and reaches the camera E. However, with recursive ray tracing at the diffuse surface D light can arrive from any direction in the hemisphere Omega above the surface. This is a challenging path to handle because it involves integrating light across all incoming directions:
![](../Images/Pasted%20image%2020261001201409.png)
Recursive ray tracing only samples direct lighting leading to an incorrect radiance at D.

## Global Illumination Ray Tracing
Recursive ray tracing is limited to idealized specular and direct lighting scenarios. Transitioning to global illumination ray tracing enables:
- Handling complex illumination effects such as indirect diffuse illumination and caustics (LS+DE)
- Utilizing advanced techniques like Monte Carlo integration for accurate light transport simulation
- Expanding on the principles of Distribution Ray Tracing to capture real world lighting phenomena

