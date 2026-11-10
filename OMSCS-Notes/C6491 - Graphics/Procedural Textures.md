Unlike image based textures, procedural textures generate patterns algorithmically. They are commonly used for materials like wood, marble, clouds, and terrain.
## Solid Textures
Texture mapping typically uses 2D images but solid textures define a function over 3D space. This makes it so there is no need for explicit ST/UV mapping and textures are evaluated at each surface point. 
## Simple Procedural Color Mapping
The simplest form of a procedural texture assigns one color to one side of an axis-aligned half-space and another color to the other side. This approach requires only evaluating 3D coordinates of a surface point to determine its color. For example, assigning red points to the negative x-axis and blue points to the positive x-axis.
## Wood Grain via Conical Space
A more complex procedural texture can simulate wood grain using a conical coordinate space. This approach computes the distance from the central axis and applies a sine function to create rings. Using local object coordinates simplifies procedural texture rules.![](../Images/Pasted%20image%2020261110174931.png)
## Conventional Texture Mapping vs. Solid Textures
When mapping 2D textures to 3D surfaces, issues such as seams and texture distortion can occur due to challenges in ST/UV mapping. Solid textures prevent this since they create textures procedurally using 3D coordinates. 
## Procedural Noise
Procedural noise functions generate pseudo-random patterns that simulate natural phenomena. They are widely used in computer graphics for generating textures such as clouds, fire, and terrain. They can also be layered to create complex and realistic surfaces. When applied in three dimensional space, procedural noise is sometimes referred to as **solid noise**.
### Perlin Noise
Perlin noise is a type of noise that generalizes natural looking randomness. Unlike purely random noise, Perlin noise is based on a structured grid of gradient vectors. The function smoothly interpolates between these gradients, resulting in continuous, coherent variations that are well suited for natural textures. 
### Perlin Noise 1D Pseudocode
```C
float noise1D(x) {
    int p1 = floor(x)
    int p2 = floor(x) + 1
    float t = x - p1 // 0 <= t <= 1
    seed(p1)
    slope1 = 2 * rand_int(2) - 1
    seed(p2)
    slope2 = 2 * rand_int(2) - 1
    return smooth_interp(slope1, slope2, t)
}
```
### Perlin Noise and Octaves
Perlin noise can be expanded by layering multiple instances of the noise. Octaves in Perlin noise refer to multiple layers of noise combined at different frequencies and amplitudes. Lower frequency octaves define large smooth variations, while higher frequency octaves introduce finer details.
## Perlin Noise Implementation Details
- **Octaves** -  Multiple layers of Perlin noise added together to create complexity
- **Persistence** - A multiplier that controls how much each octave contributes to the final noise
- **Lacunarity** - A scaling factor that determines how much the frequency increases at each octave