## Barycentric Coordinates of a Triangle
Barycentric coordinates interpolate a point inside a 2D triangle based on a weighted influence from each vertex. These weights are determined by the proportional areas of the sub-triangles formed when the point divides the original triangle.
![](../../Images/Pasted%20image%2020261110141847.png)
Note that each coordinate is influenced by the area of the triangle opposite to the corresponding vertex. For example alpha in the image above is influenced by the sub-triangle PBC, or Aa. Additionally, the sum of barycentric coordinates always sum to exactly one. The point is insi
## Triangle Area via Cross Product
The signed area of triangle given three points is computed using the cross product:
$$A_{\text{total}} = \frac{1}{2} \vert{}(B - A) \times (C - A)\vert{}$$
You can also calculate the area of the sub-triangles:  
- Triangle $\Delta PBC$ (opposite to $A$):
$$A_A = \frac{1}{2} \vert{}(P - B) \times (P - C)\vert{}$$
- Triangle $\Delta PCA$ (opposite to $B$):
$$A_B = \frac{1}{2} \vert{}(P - C) \times (P - A)\vert{}$$
- Triangle $\Delta PAB$ (opposite to $C$):    
$$A_C = \frac{1}{2} \vert{}(P - A) \times (P - B)\vert{}$$
The total area of the triangle can be calculated as the sum of these three parts. This is useful since we are already computing the sub-triangle areas to determine barycentric coordinates. This means no additional calculation is needed for the total area of the triangle.
## Normalization to Define Weights
Barycentric coordinates are defined as:

$$\alpha = \frac{A_A}{A_{\text{total}}}, \quad \beta = \frac{A_B}{A_{\text{total}}}, \quad \gamma = \frac{A_C}{A_{\text{total}}}$$
These express how much each triangle contributes to the total, forming a partition of unity.
## P as a Weighted Average
The point P is expressed as a weighted sum of the triangles vertices:
$$P = \alpha A + \beta B + \gamma C$$
## Ray-Triangle Intersection Algorithm
1. Compute the Intersection Point
2. Compute Barycentric Weights
3. Test the coordinates are between 0 and 1
4. Use the coordinates for interpolation
## Barycentric Interpolation of Attributes
Barycentric coordinates allow for interpolation of any per-vertex attribute such as color, texture coordinates, and normal vectors. ![](../../Images/Pasted%20image%2020261110160050.png)
