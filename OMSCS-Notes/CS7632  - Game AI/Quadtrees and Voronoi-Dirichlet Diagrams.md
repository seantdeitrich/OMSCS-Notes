# Quadtrees & Voronoi-Dirichlet Diagrams

## Why Improve Grid Lattices?
- Grid lattices use the same resolution everywhere.
- Inefficient memory usage in large uniform regions.
- More nodes = slower pathfinding.
- Goal: Keep simple convex regions while reducing storage costs.
- Strategy:
  - Low resolution for large uniform areas.
  - High resolution near obstacles and movement-critical details.

# Quadtrees

## Definition
- Recursive spatial hierarchy.
- Starts as one large square.
- If a cell contains mixed traversable/non-traversable space, subdivide into 4 equal cells.
- Continue recursively until each cell is uniform.

## Advantages
- Efficient space usage.
- Fewer nodes than a full grid.
- Faster path searches.
- Supports fast runtime edits.
- Good for large open environments.

## Construction
1. Start with root cell.
2. Test for obstacles.
3. If mixed content -> split into 4.
4. Repeat until cells are entirely traversable or untraversable.
5. Leaf nodes store traversability information.

## Challenges
- More complex data structure than grids.
- Neighbor detection is difficult.
  - Especially for non-sibling cells.
  - Often requires tree traversal.
- Poor path quality without post-processing.
- Can create excessive subdivisions when obstacles do not align with cell boundaries.
- Performs poorly with buildings, hallways, and complex geometry.
- Level design may need to align with quadtree boundaries.

## Star Trek Armada Example
- Used quadtrees in a sparse 2D RTS space environment.
- Pre-built a neighbor graph.
- Solved neighbor lookup issues.
- Initial paths were inefficient.
- Applied string pulling to improve paths.

## String Pulling
- Post-processing step.
- Uses A* plus visibility tests.
- Removes unnecessary turns.
- Produces straighter, more natural paths.

## Best Uses
- Outdoor environments.
- Sparse maps.
- RTS games.
- Large open areas with few obstacles.

# Voronoi-Dirichlet Diagrams

## Definition
- Space divided into regions around generator points.
- Every location inside a region is closest to that region's point.
- Boundaries are equidistant from neighboring points.

## Path Planning
- Build a graph from Voronoi boundaries.
- Polygon corners become graph nodes.
- Polygon edges become graph edges.

## Advantages
- Good obstacle avoidance.
- Naturally keeps paths away from isolated obstacles.
- Supports weighted regions.
- Popular in game AI and robotics.

## Manual Placement Approach
- Place generator points manually.
- Adjust point weights to change region sizes.
- Useful for handcrafted navigation spaces.

## Challenges
- Less effective in hallways and complex shapes.
- May require multiple points per region.
- Some generated edges may cross obstacles.
- Often requires manual cleanup or edge removal.

## Clearance-Based Voronoi
- Uses obstacle areas rather than points.
- Produces paths with maximum obstacle clearance.
- Useful for:
  - Large vehicles
  - Robotics
  - Piano mover problems
- Not ideal for shortest-path optimization.
- Usually requires string pulling afterward.

## Delaunay Triangulation
- Closely related to Voronoi diagrams.
- Use Constrained Edge Delaunay Triangulation when obstacles exist.
- Obstacles are treated as holes.
- Easily converted into a Voronoi diagram.

## Best Uses
- Obstacle avoidance.
- High-clearance navigation.
- Robotics motion planning.
- Manual navigation graph generation.

# Comparison

| Feature | Quadtree | Voronoi |
|----------|----------|----------|
| Space Efficiency | Excellent | Good |
| Dynamic Updates | Excellent | Moderate |
| Neighbor Lookup | Difficult | Simple |
| Path Quality | Poor without string pulling | Good clearance but not optimal |
| Obstacle Avoidance | Moderate | Excellent |
| Hallways/Buildings | Poor | Requires tuning |
| Open Environments | Excellent | Good |
| Robotics | Sometimes | Very Common |

# Exam Notes
- Quadtrees = variable-resolution grid hierarchy.
- Main quadtree benefit = reduced storage and faster search.
- Main quadtree problem = neighbor determination and path quality.
- String pulling improves quadtree paths.
- Voronoi regions contain points closest to a generator point.
- Voronoi boundaries are equidistant from neighboring generators.
- Voronoi graphs maximize obstacle clearance.
- Good for large vehicles and robotics.
- Bad for shortest/optimal paths.
- Delaunay triangulation can generate Voronoi diagrams.