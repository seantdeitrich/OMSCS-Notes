Texture mapping is a a technique used to apply two dimensional images onto three dimensional surfaces. **h** are 2D images can be used to store surface details like color, patterns, or normals. **Texels** are a texture element, representing a single sampled value from the texture map. 
## Texture Mapping Process
To map a texture onto a 3D surface, we first need to transform the surface's coordinates into texture space. This process assigns each point on the 3D model a corresponding location in the 2D texture using coordinates typically labeled as *s* and *t* or *u* and *v*. These texture coordinates define how the texture wraps around the model. In practice, texture coordinates are assigned at specific fixed points on the 3D geometry such as vertices of triangles in a mesh. For all other points on the surface, texture coordinates are interpolated to ensure smooth mapping across the model. 
## Texture Coordinate Mapping
Texture coordinates *(s,t)* are assigned to each vertex of a triangle. Each triangle vertex stores a texture coordinate that maps to a texture image. To determine the texture coordinate at any point P inside the triangle:
- Use barycentric interpolation to compute the texture coordinate at point P
- Can map texture to other surfaces so long as interpolation of texture coordinates is possible
## Texture Sampling
Once the texture coordinate is determined, a texel must be sampled. Common methods for this include:
- Nearest Neighbor - selects the closest texel to the requested coordinate 
- Bilinear Interpolation - samples the four nearest texels and blends their values based on distances to the coordinate
Aliasing can occur with either of these methods, particularly when a texture is minified. Additionally, when a texture is viewed at a smaller scale than its original resolution, multiple texels may contribute to a single pixel leading to flickering or visual artifacts. More advanced filtering techniques such as mipmapping as isotropic filtering are needed to ensure a smoother and more accurate representation of the texture across different viewing distances.
## Magnification
### Nearest Neighbor Approach
When a texture is applied at a larger resolution than its native size, you can select the texel closest to the requested coordinate. However, this produces blocky, pixelated visuals and sharp transitions between texels.

![](../Images/Pasted%20image%2020261110161626.png)
Nearest neighbor is computationally efficient but not ideal for high quality magnification. It can be used in lower resolution or blocky / pixelated games.
### Bilinear Interpolation Approach
This approach improves magnification by using the four surrounding texels to interpolate color. This blurs sharp transitions and reduces pixelation, but doesn't eliminate blurring and may make final details appear soft. Bilinear interpolation works by performing two linear interpolations in one direction and a final interpolation in the other. 

Interpolation formula:
- $C_0 = C_{0,0} + \alpha(C_{1,0} - C_{0,0})$
- $C_1 = C_{0,1} + \alpha(C_{1,1} - C_{0,1})$    
- $C = C_0 + \beta(C_1 - C_0)$
- where $\alpha, \beta$ are fractional offsets in the texel space.
![](../Images/Pasted%20image%2020261110162112.png)
## Minification
Minification is when a texture appears smaller on the screen than its original resolution. This creates challenges where fine details in the texture may no longer be accurately represented. Nearest neighbor sampling is highly prone to aliasing in this case. Patterns become distorted and small details may disappear or flicker as the viewpoint changes. Bilinear Interpolation improves this by averaging a two-by-two group of texels reducing sharp pixelization. However, it does not fully prevent aliasing at smaller scales. To address these problems, more advanced solutions like mipmapping are needed. 

**Mipmapping** precomputes lower resolution versions of textures. This allows the renderer to select a version that better matches the display scale, reducing aliasing. **Isotropic Filtering** further improves texture quality by adapting sampling based on the angle of the surface relative to the viewer.

### Nearest Neighbor and Minification
Since nearest neighbor does not interpolate, it can skip over regions and lead to a loss of detail. The following picture shows sampling in four regions where the black squares are completely ignored:
![](../Images/Pasted%20image%2020261110172620.png)

### Bilinear Interpolation and Minification
Bilinear Interpolation can still skip over texels, leading to aliasing:
![](../Images/Pasted%20image%2020261110172829.png)
All samples in this image align with the middle of white squares, resulting in interpolation of only white textures. The result is a blurred, incorrect representation of the original pattern. While bilinear interpolation smooths transitions, it does not solve aliasing for minification.
## Image Pyramid
An image pyramid is a technique used to create multiple levels of an image at progressively lower resolutions. Each level is generated by applying a smooth downsampling process, preserving important details and avoiding texel skipping. This provides a multi-scale representation of the texture and helps in handling different viewing distances and reducing aliasing. The process to create an image pyramid typically includes a blurring or smoothing filter before downsampling occurs. In texture mapping, this structured approach is known as mipmapping. 
## Mipmapping
Mipmapping is a solution to minification that precomputes multiple texture levels at reduced resolutions. Each level is half the resolution of the previous, and a lookup function selects the mipmap level closest to the screen footprint. This approach reduces aliasing and improves performance.
## Trilinear Filtering
An improvement over bilinear filtering that addresses the problem of abrupt transitions between mipmap levels by introducing even smoother blending. With bilinear filtering, textures are smoothly interpolated within a single mipmap level. The level selected is the one that best matches the screen space footprint of the texture. However, when switching between different mipmap levels, abrupt changes in texture detail can still occur, leading to noticeable seams or flickering. In trilinear filtering, a bilinear interpolation is applied separately to both the lower resolution and higher resolution mip levels. Then these two results are linearly blended based on how closely the texture footprint falls between the two mip levels. This interpolation is controlled by a blending factor Gamma, which acts as a weight to determine how much influence each mip level has in the final result
$$C = C_{\text{low}} + \gamma (C_{\text{high}} - C_{\text{low}})$$
![](../Images/Pasted%20image%2020261110173956.png)
Selecting the appropriate mip level is important for ensuring that textures appear correctly at different viewing distances. The goal is to choose a level where the texel size closely matches the pixel size in screen space. Note that mip level selection is always an approximation.