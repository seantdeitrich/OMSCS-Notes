Photon mapping is a two-pass global illumination technique where photon maps are created at surfaces, and those maps are queried at ray hits. Rays are cast from light sources in this process which differs from conventional ray tracing. It is efficient for complex lighting scenarios, handles caustics and indirect diffuse illumination, and accurately simulates light.
## Two Pass Algorithm
**Pass 1**: 
- Photons are traced from light sources (not the camera)
- As these photons interact with surfaces, their positions, energy, and directions are stored in a kd-tree
	- There are two types of photon maps, caustic and indirect illumination,
**Pass 2:**
- Rays are traced from the eye or camera
- The photon map is queried to estimate illumination density
- When a ray hits a surface the photon map is queried to estimate 
- The final illumination is calculated using a combination of traditional ray tracing methods, caustics from the caustic map, and indirect illumination from the global (indirect) photon map
## Caustics with Photon Mapping
Caustics occur when light is concentrated by specular interactions such as reflection off shiny surfaces or refraction through transparent objects. The caustic photon map only contributes or cares about photons that contribute to caustics, with a light path of LS+DE. Other paths do not result in caustic photon mapping. 
## Photon Storage
Each photon stores its position, energy (flux), and incoming direction. KD-Trees are used because they enable fast nearest neighbor lookup, support quick photon density estimation, and are an efficient data structure for global illumination queries. 

Photon map KD-Trees split the space on the median photon along the target axis, which balances the tree. Photons are stored in the leaves, with leaves usually containing multiple photons to improve performance. In practice 'bending' photons in each leaf helps improve performance. 
## K Nearest Neighbor Photon Search
The goal here is to find the k-nearest photons to a surface point efficiently. These neighbors are not necessarily in the same subtree, and backtracking is required to ensure the search is exhaustive. Additionally, the search must only terminate when all possible candidates are checked.
### Algorithm
1. Start at the root node, compare the query point with the splitting plane, and recursively move into the closer child node first
	1. When a leaf node is reached, store its photon
2. Maintain a priority queue that tracks the best candidates (nearest photons, where the queue is sorted by farthest distance among the k-nearest photons
	1. If fewer than k photons are found, add the new photon
	2. If the new photon is closer than the farthest in the heap, replace it
3. Backtrack the opposite subtree
	1. The k nearest photons may exist on both sides of the splitting plane, and the query sphere (defined by k) may overlap the opposite subtree
	2. If the distance to the splitting plane is smaller than the farthest kth photon, expand the opposite subtree, otherwise skip processing that branch
4. End the search when the queue contains at least k-nearest photons AND all necessary backtracking has been completed (no branch could obtain closer photons) and the query sphere does not overlap any unexplored subtree.

To optimize the KNN search, leaves of the KD-Tree can store small bins of photons instead of individual photons. This reduces tree depth and speeds up KNN retrieval.
## Indirect Diffuse Illumination
The goal is to simulate interreflection of light in a scene. Unlike caustics, photons are stored at diffuse bounces. There are two estimation strategies:
- Basic - Uses a query kd-tree for nearest photons and for computing density
- Improved - Uses **Final Gathering** to shoot rays for a more accurate radiance estimate
Note that we don't store direct illumination in the photon map, only indirect illumination.

Also note that querying the photon map directly at eye ray intersections causes noisy final images with the basic approach. 

### Final Gathering for Indirect Illumination
Instead of simple lookups in the kd-tree for indirect illumination which produces noisy results, shoot rays over a hemisphere to sample incoming radiance from other surfaces. Stored photon information at these surfaces provides an efficient approximation. This approach uses Irradiance Caching for efficiency, which avoids redundant computation by storing and reusing results.

For Irradiance Caching:
- When the cached values are too far away, or when surface angles are too different then recalculate the radiance
## Radiance Estimation Formula
Radiance at a point is estimated by accumulating the contributions of the k nearest photons. The outgoing radiance formula is:
$$L_o(x, k_o) \approx \sum_{p=1}^{N} \rho(x, k_i, k_o) \frac{\Delta \Phi_p}{\pi r^2}$$
- Lo is the outgoing radiance at point x in direction ko
- p(x,ki,ko) is the BRDF at the surface for incoming ki and outgoing ko
- Delta Phi p is the energy of photon p
- r is the radius enclosing the N nearest photons

## Choosing k for Radiance Estimation
Selecting k (the number of photons) is important to optimize the algorithm. Typical values include 50-100 photons, where fewer photons improves speed but is noisier, and more photons is smoother but slower and blurrier. 

## Cone Filtering
Cone filtering adjusts photon contributions based on their distances from the query point. This approach reduces blurring in low-density photon regions and enhances image sharpness. Photons closer to the query point are given more weight while distant photons are given less. This is calculated using the filter weight formula:
$$w_p = \max\left(0, 1 - \frac{d_p}{k \cdot r}\right)$$
- dp is the distance from point x to photon p

