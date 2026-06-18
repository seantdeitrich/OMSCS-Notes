## Phase 1 — Triangle Formation

**1. Check for degenerate triangle**  
Call `IsCollinear(V1, V2, V3)` — if true, `continue`

**2. Check for obstacle vertices "between" each edge**  
For each of the 3 edges (V1-V2, V2-V3, V1-V3):
- Call `IsLineSegmentInPolygons(A, B, obstacles)` — store the result
- If it's NOT an obstacle edge, loop through all `obstacleVertices` (skipping the two endpoints) and call `Between(A, B, vertex)` — if any vertex is between, `continue`

**3. Create the triangle polygon**  
`new Polygon(V1, V2, V3)` — if `!IsCCW()`, call `tri.Reverse()`

**4. Check for overlap with existing triangles**  
`IntersectsConvexPolygons(tri, origTriangles)` — if true, `continue`

**5. Check for enclosed obstacle vertices**  
Loop `obstacleVertices` calling `IsPointInsidePolygon(tri.pts, v)` — if any inside, `continue`  
Also check `tri.Equals(obstaclePolygon)` for each obstacle — if match, `continue`

**6. Check for intersection with obstacle edges**  
For each edge NOT already confirmed as an obstacle edge (from step 2), call `InteriorIntersectionLineSegmentWithPolygons(A, B, obstacles)` — if true, `continue`

**7. Add the triangle**  
`origTriangles.Add(tri)` and `adjPolys.AddPolygon(tri)`

**8. Test Phase 1**  
Set `navmeshPolygons = new List<Polygon>(origTriangles)` (already done) and return early to visualize yellow triangles

---

## Phase 2 — Merge Triangles

**9. Shallow copy adjPolys**  
`newAdjPolys = new AdjacentPolygons(adjPolys)`

**10. Iterative merge loop**  
Wrap in a `while` loop that tracks merge count, exits when 0 merges occur:
- Iterate `adjPolys.Keys` (each `CommonPolygonEdge`)
- Get the `CommonPolygons` value — skip if `.IsBarrier`
- Call `MergePolygons(cp.AB, cp.BA, edge.A, edge.B)` to get a candidate
- Call `IsConvex(merged.pts)` — if not convex, skip
- If convex:
  - `newAdjPolys.Remove(commonEdge)`
  - `newAdjPolys.AddPolygon(merged, cp.AB, cp.BA)` (replaces old refs)
  - Remove old two polys from `navmeshPolygons`, add `merged`
  - Increment merge count

**11. After loop**  
`adjPolys = newAdjPolys`

**12. Test Phase 2**  
Visualize — blue outlines should show larger merged polygons with fewer internal edges

---

## Phase 3 — Path Network

**13. First pass — create path nodes**  
Iterate `adjPolys.Keys`:
- Skip `.IsBarrier` edges (boundary edges, not portals)
- For non-barrier edges, compute midpoint: `(edge.A + edge.B) / 2` → convert to `Vector2`
- `pathNodes.Add(midpoint)`
- Build a dictionary: `edgeToNodeIndex[commonEdge] = pathNodes.Count - 1`

**14. Build polygon → edge list mapping**  
Build a dictionary: `polyToEdges[polygon] = List<CommonPolygonEdge>`  
Iterate `adjPolys.Keys` again — for each non-barrier edge, add the edge to both `cp.AB`'s list and `cp.BA`'s list

**15. Second pass — create path edges**  
Initialize `pathEdges` with an empty `List<int>` for each node  
Iterate `adjPolys.Keys` for non-barrier edges:
- Get node index for this edge from `edgeToNodeIndex`
- For each polygon sharing this edge (AB and BA):
  - Look up all other edges in that polygon via `polyToEdges`
  - For each other edge, get its node index
  - Add bidirectional edges (check for duplicates and no self-links)
