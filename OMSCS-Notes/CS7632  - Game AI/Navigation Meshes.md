# Space Representation & NavMeshes

## Space Discretization Architectures

* 
**Grid Lattice (Uniform Structure):** 


* Uses implicit mapping to translate coordinates directly to game world positions.


* Derived mathematically via `(gridX, gridY) * cellWidth + gridWorldCorner`.


* Coordinates are calculated dynamically on demand.




* 
**Navigation Mesh / NavMesh (Non-Uniform Structure):** 


* Relies on explicit mapping to handle dynamic discretization of the environment.


* Requires significantly higher memory per discretized unit compared to uniform grids.


* Offers structural flexibility that yields substantial pathfinding and performance benefits over uniform architectures.





## Properties of Convex Polygons in Game AI

* 
**Computational Efficiency:** Simpler to calculate point containment and ray-intersection tests than concave shapes.


* 
**Point Containment Test:** Executed by iterating through vertices counter-clockwise (CCW) using a `LeftOn(a, b, pt)` geometric orientation test.


* 
**Line Segment / Ray Intersection Test:** Verified by checking edge intersections sequentially along a counter-clockwise traversal.


* 
**Line-of-Sight Guarantee:** For any convex polygon, any point $A$ inside or on the border can directly see and reach any other point $B$ within that same polygon without crossing an obstruction.



## Graph Connectivity Fundamentals

* 
**Polygonal Nodes:** Navigable spaces are mapped out as collections of convex polygonal regions.


* 
**Edge Intersections:** Graph connectivity is explicitly defined by adjacent polygon edges.


* 
**Search Graphs:** The underlying pathfinding graph is drawn directly through the boundaries of the navigation mesh.



---

# NavMesh Generation & Optimization

## 1. Voxelized Rasterization (The Recast Approach)

A popular method used by commercial engines like Unity to generate meshes through volumetric generation:

1. 
**Rasterization:** Input triangle meshes are rasterized into a multi-layer volumetric heightfield (voxel mold).


2. 
**Filtering:** Filters evaluate agent height and slope clearance to prune unnavigable voxels from the mold.


3. 
**Region Partitioning:** The remaining walkable voxel spaces are grouped into distinct, simplified 2D overlayed regions with non-overlapping contours.


4. 
**Polygon Extraction:** The system traces boundaries, refines contours, and peels them away into clear convex polygons optimized for pathfinding algorithms.



## 2. Greedy Triangulation Algorithm

An alternative approach designed to create mesh triangles exhaustively across a scene environment:

### Basic Loop

```text
For each obstacle/boundary point "a"
    For each obstacle/boundary point "b"
        For each obstacle/boundary point "c"
            Define Candidate Triangle(a, b, c)

```

* Validate that the candidate triangle contains a non-zero area when normalized to Counter Clockwise (CCW) orientation.


* Ensure the triangle does not intersect any obstacles or previously constructed valid mesh triangles.
* ![](../Images/Pasted%20image%2020260610174621.png)
* Collinear tests are needed to prevent the BC error
* If valid, append it to the master list; repeat until no more candidate triangles can be generated.
### Core Technical Hurdles & Solutions
#### Algorithmic Complexity
* **Problem:** The brute-force implementation runs at an expensive complexity of $O(n^4)$.
* **Performance Improvements:**
* Prune nested evaluation loops to avoid vertex repetition via combinatorial indexing ($N \text{ choose } 3$).
* Evaluate individual lines sequentially: immediately abandon a candidate triangle the moment any single edge fails an obstacle intersection check.
* Store intersection test data in an edge-indexed hash table to reuse cached results when edges are re-evaluated.
#### Adjacency Blocking
* **Problem:** Suboptimal triangular alignments can inadvertently cross navigable channels, blocking adjacent polygons from linking together correctly.
* **Solution:** Introduce a geometric screening check to determine if any obstacle vertices lie directly collinear between the endpoints of a candidate edge. Skip the edge immediately if an intersection is found.
#### Floating-Point Fragility
* 
**Problem:** Numerical precision errors and inconsistent `Epsilon` tolerances break collinear intersection checks, leading to corrupt mesh geometry and broken fans of triangles.


* 
**Solution:** Enforce strict mathematical alignment between collinearity validation and degenerate triangle filtering. Convert vector coordinates to integer spaces to ensure deterministic execution.



#### Long, Skinny Triangles

* 
**Problem:** Narrow sliver triangles degrade pathing accuracy and cause rendering artifacts due to precision limits in floating-point math.


* 
**Solution:** Swap out greedy construction routines for Constrained Delaunay Triangulation algorithms (such as the `poly2tri` framework) to maximize the minimum interior angles of triangles.



---

# Graph Extraction & Waypoint Placement

## Adjacency & Topology Mapping

* Identify shared, overlapping edges between adjacent polygons to map out mesh connectivity.


* Cache common edges in a hash table to build connection paths quickly.


* Points bounding a shared edge run in opposing directions on adjacent polygons (e.g., $A \rightarrow B$ vs. $B \rightarrow A$).


* Compute a commutative, order-invariant hash code utilizing vector dot products to match shared edges accurately.



## Greedy Convex Merging

* Combine adjacent triangles greedily to simplify the final search graph and minimize total node count.


* 
**Validation Check:** Merge two test polygons into a unified candidate shape and evaluate its structural convexity.


* If the shape remains convex, swap out the original triangles for the newly merged polygon and refresh the edge map.



## Waypoint Node Placement Strategies

Extracting clean waypoints from an underlying mesh generally relies on five primary node-placement strategies:

| Strategy | Technical Trade-offs & Implementation Notes |
| --- | --- |
| <br>**Polygon Centroid Placement** 

 | Places graph nodes directly at the geometric center of each polygon. While simple, centroids in concave mesh clusters or across borders can occasionally generate path coordinates that lead straight through geometry walls.

 |
| <br>**Edge Midpoint Placement** 

 | Anchors pathing nodes directly on the midpoints of shared polygon borders. This guarantees that graph links cross cleanly between adjacent regions.

 |
| <br>**Obstacle Corner Placement** 

 | Positions navigation nodes right at the outer vertices of surrounding obstacles. This allows agents to plan paths that hug corners efficiently.

 |
| <br>**Combined Edge & Corner Placement** 

 | Combines border midpoints and obstacle corners to increase local node density, producing smoother tracking routes. |
| <br>**Perimeter Placement** 

 | Anchors nodes firmly along region perimeters. This provides clear lines of sight across all connected polygons.

 |

---

# Advanced NavMesh Geometry

## Expanded Obstacle Geometry

* 
**Concept:** Inflate obstacle bounds outward by the agent's bounding radius before generating the navigation mesh.


* 
**Benefit:** Simplifies path searches by reducing complex agent collision tests down to single-point raycasts.



```text
       Original Obstacle              Expanded Obstacle (Bake Time)
          +--------+                           +--------------+
          |        |                           |  +--------+  |
          |        |  ===> (Agent Radius) ===> |  |        |  |
          +--------+                           |  +--------+  |
                                               +--------------+
                                            Agent acts as a single point

```

### Corner Geometry Offsets

* 
**Edge Expansion Problems:** Offsets calculated from edge normals often overestimate clearance zones around acute corners.


* 
**Vertex Expansion Problems:** Radial vertex projections can inadvertently underestimate clearances along flat walls.


* 
**The Curve Problem:** True equidistant dilation introduces non-linear curves at corners, requiring dense triangle tessellation.


* 
**Compromise Solution:** Enforce standard miter limits or square off corner extensions to maintain linear bounds without creating curved surfaces.



---

# Path Quantization & Refinement

## Localization & Spatial Partitioning

Locating an arbitrary world coordinate on a navigation mesh is highly optimized through specialized spatial data structures:

* 
**Graph Traversal Methods:** Run an $A^*$ search from the closest known mesh node using distance heuristics to evaluate visited regions.


* 
**Bounding Circles:** Store structural bounding radii on mesh nodes to filter out invalid containment checks quickly.


* 
**Spatial Partitioning:** Organize mesh polygons inside spatial grids, Quadtrees, or Octrees to ensure fast $O(\log N)$ point lookup queries.


* 
**Coherence Optimization:** Track moving targets by checking adjacent polygons sequentially, assuming agents move smoothly from one region to the next.



## Simple Stupid Funnel Algorithm (String Pulling)

Converts jagged, zig-zag node sequences into straight paths. This algorithm works best on navigation structures that use pre-baked expanded obstacle geometries.

```text
                  Left Funnel Edge
                     \        Portal Point (Left)
                      \      /
                       \    /  
     Agent Position === *==*------------------=> Smooth Target Vector
                       /    \
                      /      \
                     /        Portal Point (Right)
                  Right Funnel Edge

```

### Execution Steps

1. Initialize the funnel's left and right border vectors from the agent's start position toward the first polygon portal.


2. Step forward along the planned sequence of polygons, narrowing the funnel edges as new portal points are evaluated.


3. If a new portal point falls inside the existing bounds, shrink the funnel to match it.


4. 
**The Crossing Condition:** If the left and right funnel edges cross, lock in the apex point on the opposite side, add it to the final path, and restart the funnel from that new position.



## Horizon Zero Dawn Funnel Variant

An adjusted funnel algorithm optimized for unexpanded raw environments that calculates agent clearances dynamically.

```text
             Funnel Apex (Start)
                   /   \
                  /     \
                 /       \
  Funnel Crossing Point   \
               / \         \
              /   \         \
             /     \         \
  [Wall Offset] === * * Portal Point (Unchanged Side)
                    |
              [Clamped Bound]
                    |
             Opposite Geometry

```

### Execution Steps

1. Execute the standard funnel algorithm until a crossing condition is triggered.


2. Instead of locking in the raw portal vertex directly, compute a clearance vector combining the funnel's path trajectory and the wall normal.


3. 
**Offset Calculation:** Determine the target displacement using a combination of the wall's interior angle and the character's turn radius.


4. 
**Safety Clamp:** Clamp the final clearance offset against the opposite wall boundary to guarantee that coordinates never push past the printable mesh area.


5. Append the offset coordinate to the final path and restart the funnel routine from there. This allows characters to cut corners cleanly while avoiding walls.



---

### Obsidian Setup Tips

* **Graph View:** These notes are linked using standard concepts like `[[Convex Polygon]]` and `[[Funnel Algorithm]]`. If you break these sections out into individual files, Obsidian's Graph View will automatically map their relationships.
* **Tags:** `#game-ai`, `#path-planning`, and `#navmesh` are placed at the top of the file for quick workspace organization and tag searches.