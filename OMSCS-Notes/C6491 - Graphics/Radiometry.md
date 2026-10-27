**Radiometry** is the measurement of light passing through space in terms of energy and power. 
## Radiometry Units and Quantities of Light
A **Photon** is a quantum of light characterized by:
- Position
- Direction of propagation
- Wavelength
The energy of a single photon is calculated as:
$$q = hc/\lambda$$
- Plank's constant h = 6.63x10<sup>-34</sup> Joules
- The speed of light c = 2.998x10<sup>8</sup> m/s
- The lecture says that wavelength is measured in nm, but for this equation to work properly the wavelength would have to be in m
## Quantities for Energy and Power of Light
- **Energy** (Q) is the total light energy measured in Joules
	- This represents the cumulative energy of photons over a period or area
- **Power** (Phi) is the rate of energy flow over time, measured in Watts
	- 1 watt = 1 Joule / second
	- This describes how much light energy is emitted or transferred per unit of time
- Examples:
	- A 100 Watt light bulb emits 100 joules of energy per second
	- Not all power is visible light (e.g. some is lost as heat)
## Tailoring Radiometry for Graphics
**Radiance** is the lights intensity in specific directions (along a ray). We can then connect radiance to measurable quantities like watts per steradian. Wavelength must also be incorporated into radiance for realistic color representation. Furthermore, surface angle relative to the light (Lambert's Cosine Law) must be accounted for, as well as distinguishing between incoming and outgoing radiance for surface calculations. Finally, the material properties and how they modify light direction and intensity must be considered as light interacts with the surface. 
In summary:
- Radiance is a fundamental that should be calculated considering the following:
	- Wavelength
	- Surface Angle
	- Surface Material
## Spectral Quantities
Spectral quantities allow us to analyze the distribution and power across different wavelengths of the spectrum in nanometer units. 
- **Spectral Energy** (Q) is energy (J) per wavelength (in nanometers), making the unit Joules per nanometer.
- **Spectral Power** (Phi) is power (W) per wavelength (nm) interval, making the units Watters per nanometer. This is used to describe light sources and their distributions.
	- Example: Total power = 100W over 400nm to 800nm
		- Spectral power = 100W/400nm (the range) W/nm
## Light Direction with Polar Coordinates
To describe light distributed across all possible directions from a single point, we use a polar or spherical coordinate system. 
- **Azimuth** (phi) has the range 0 to 2pi and is the angle around the horizontal plane
- **Polar Angle** (theta) has a range of 0 to pi and is the angle from the vertical axis
- **Radius** (r) is the distance to the edge of the sensor sphere
![](../Images/Pasted%20image%2020260919123516.png)
## Solid Angles and Steradians
A solid angle (Omega) measures how large a patch of directions appears from a point. A full sphere has a total solid angle of four pi steradians. 
$$\Delta\Omega=\Delta S/r^2 $$
- Delta S is the surface area of the spherical patch
- r is the radius of the sphere
Dividing by r^2 normalizes to direction only, giving the solid angle
This normalization standardizes directional measurements offset from the point.
The unit of solid angles is **steradians**. A sphere subtends a total solid angle of 4pi sr, which is the area of a sphere with a radius of 1 unit.
## Radiant Intensity
Radiant intensity describes the power of light emitted in a specific direction per unit solid angle:
$$I = \Delta\phi/\Delta\Omega$$
- Delta phi is radiant power in watts
- Delta Omega is the solid angle in steradians
Radiant intensity is directional, focuses on the distribution of light power in different directions. 
The units for radiant intensity are Watts per steradian.
## Isotropic Light Sources
An isotropic light source emits power equally in all directions. This uniform distribution simplifies the calculation of radiant intensity to:
$$I = \phi / 4\pi$$
## Projected Power
Projected power accounts for the cosine effect on radiant power due to the projected area:
$$\Delta\phi_{projected} = \Delta\phi cost\theta$$
- Delta Phi is the total radiant power emitted or reflected
- Theta is the angel between the surface normal and measurement direction
## Radiance
Radiance (L) is a fundamental quantity of radiometry that combines several key concepts into a single measurement. It represents the radiant flux or power emitted, reflected, transmitted, or received by a surface, per unit solid angle, per unit projected area of the surface.
The formula for radiance is:
$$L = \frac{\Delta\phi}{\Delta Acos\theta \Delta\Omega}$$
- Delta phi is the radiant flux (spectral power)
- Delta A cos theta is the projected surface area (onto an imaginary plane perpendicular to the radiance direction)
- Delta Omega is the differential solid angle
The unit for radiance is Watters per (meter squared meridian nanometer). Radiance is fundamental to graphics and ray tracing as it does not vary a long.
## Radiometry Irradiance
**Iiradiance (H)** is the total radiant flux (phi) incident on a surface per unit area. It is the integral of incoming radiance over the hemisphere of directions above the surface:
$$H = \int_{\Omega}L_{in}cos\theta d\Omega$$
- Lin is the incoming radiance
- cos theta is the projection factor for the incident angle
- d Omega is the differential solid angle
The unit for irradiance is waters per meter squared.
## Reflectance
Reflectance (rho) is the ratio of outgoing to incoming radiance at a surface:
$$\rho = \frac{L_{out}}{L_{in}}, \space \rho \le 1$$
## Bidirectional Reflectance Distribution Function (BRDF)
This function describes how light is reflected at a surface. It relates incoming radiance from direction k<sub>i</sub> to outgoing radiance in direction k<sub>o</sub>:
$$\rho(k_i,k_o) = \frac{L_{out}(k_o)}{L_{in}(k_i)}$$
- L<sub>out</sub>(k<sub>o</sub>) is the radiance in the outgoing direction
- L<sub>in</sub>(k<sub>i</sub>) is the radiance in the incoming direction, sometimes represented with an H instead of L
- Both directions (k's) are expressed in polar coordinates
The resulting unit of the BRDF is inverse steradians.

Practically, the BRDF is used to calculate outgoing radiance for a given incoming light source based on surface properties. This can be computer with:
$${L_{out}(k_o)} = \int_{\Omega}\rho(k_i,k_o)L_{in}(k_i)cos\theta_i d\Omega_i$$
### Discrete vs. Continuous Evaluation
The BRDF can be evaluated in two distinct ways depending on the lighting scenario.
**Discrete Evaluation of BRDF (Direct Illumination):**
- Evaluates light contributions from specific point light sources
- No integration is needed, BRDF is applied per light source
- Example: Phong shading uses discrete directions for k<sub>i</sub>
**Continuous Evaluation of BRDF (Global Illumination):**
- Integrates light contributions from all incoming directions in the hemisphere
- Captures indirect lighting and reflections
- Requires numerical methods like Monte Carlo sampling for approximation of integrals
The discrete evaluation of BRDF simplifies lighting for efficiency. The continuous evaluation method generalizes this to model realistic light transport in global illumination.

### Properties of BRDF
**Energy Conservation**:
- Outgoing energy cannot exceed incoming energy
**Reciprocity**:
- For any pair of directions ki and ko
- $$\rho(k_i,k_o)=\rho(k_o,k_i)$$
- Reflects the physical principle of reversible light paths

### Material Types and BRDF
**Diffuse (Lambertian)** materials reflect light uniformly across the hemisphere, independent of viewing angle. Radiance is constant for all view angles for a fixed incoming light direction.
$$\rho() = \rho_{diffuse}/\pi$$
**Glossy or Specular** materials reflect light preferentially towards directions near the specular reflection angle, producing visible highlights. BRDF depends on polar angles and relative azimuth between incoming light and outgoing measurement direction. 
$$(\theta_i, \theta_o, \phi_o-\phi_i)$$This is often paired with diffuse reflectance, such as in Phong shading to model materials with both shiny and matte properties.

![](../Images/Pasted%20image%2020260919143733.png)

**Isotropic Materials** reflect light identically in all directions, for example perfectly smooth uniform materials. **Anisotropic Materials** reflect light differently depending on direction, for example brushed metal or fabrics, which need all four degrees of freedom in BRDF.

## Radiosity
Radiosity is a rendering method for computing global illumination. Based on radiometric principles, it calculates the transfer of light energy between surfaces. 
Features:
- Models indirect lighting, including diffuse interreflections
- Assumes all surfaces reflect light diffusely (Lambertian surfaces)
Applications:
- Previously used in ray tracers like POV-Ray to precompute lighting
- Utilized in rasterization based renderers for static shadow maps or baked lighting

