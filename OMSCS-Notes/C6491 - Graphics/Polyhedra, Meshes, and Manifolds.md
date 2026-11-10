## Polyhedra 
Polyhedra are three dimensional solids composed of flat polygonal faces, for example tetrahedrons and cubes. 
## Meshes
Meshes are used to represent geometry in computer graphics by approximating the surfaces of three-dimensional objects. A mesh consists of vertices, edges, and faces arranged in a network. Although meshes directly represent Polyhedra, they can also effectively approximate curved surfaces like spheres or cylinders. Triangle meshes are the most common in computer graphics because triangles are simple and efficient to process. 
## Mesh Approximation and Curved Surfaces
The geometry of a smooth curved surface can be effectively approximated using a mesh composed of discrete vertices, edges, and faces. By sampling points on the surface of the desired shape, the mesh provides a piecewise linear approximation. Although each mesh face is flat, smooth shading is achieved through interpolation of vertex normals. However, if the goal is to render explicitly faceted surfaces, normals are computed per polygon face without interpolation, clearly displaying each individual facet. 
## Estimating the Gradient
To approximate a smooth curve surface, we can compute the gradient of the implicit surface function using finite differences at each vertex. The **gradient** is a vector composed of the partial derivatives of the function with respect to x, y, and z. Each partial derivative can be approximated by evaluating the function at small delta offset positions. The resulting gradient provides an approximation of the surface normal direction at the vertex. ![](../../Pasted%20image%2020261110182551.png)
## Computing Vertex Normals
For each vertex, compute the gradient using finite differences. The normal at the vertex is then:
$$\mathbf{n} = \frac{\nabla f}{\Vert{}\nabla f\Vert{}}$$
Which is simply the normalized version of the gradient. This process yields vertex normals that accurately reflect the local curvature of the underlying smooth surface. 
## Barycentric Interpolation of Normals
Per-vertex normals n1, n2, and n3 are interpolated across the triangle with their respective barycentric coordinates:
$$\mathbf{n}(p) = \alpha \mathbf{n}_1 + \beta \mathbf{n}_2 + \gamma \mathbf{n}_3$$
Normalizing n(p) yields a smooth normal field across the surface. This interpolation can be used with Phong Shading, giving a smooth appearance.
## Phong Shading and Mesh Rendering
Phong shading uses per-pixel lighting calculations rather than per-vertex. Although meshes consist of flat triangles, smooth shading effects are achieved by interpolating vertex normals across each triangle. These interpolated normals allow lighting calculations, including diffuse and specular reflections to simulate the appearance of smoothed curved surfaces. 
## Manifolds
A manifold is a space in which every point has a neighborhood topologically equivalent (homeomorphic) to an open region of Euclidean space of a specific dimension n. Informally, this means that every point is surrounded by a region that looks like flat Euclidean space. In computer graphics, a manifold typically refers to a surface representing the boundary or shape of a three-dimensional object. The surface looks everywhere like a piece of a 2D plane (possibly curved). 

**Stopped Watching at About 4:30**

