Path planning allows agent to seek a goal and avoid obstacle.
- In discretized space, you could search existing spaces around the agent for which one decreases the distance to the target and follow that path
	- ![](../Images/Pasted%20image%2020260523124635.png)
	- Agents can get stuck in corners with this approach
	- We need a way to keep track of candidate path choices and explore possibilities until an acceptable solution is found
	- The Graph data structure is well suited to this
**Planning**
- Part of intelligence is the ability to plan
- The world can be represented as a set of States
- Each possibility is a new configuration of states, and **operators** can be applied to change states
- An **Operator** changes configuration from one state to another state
	- In case of agents, it is movements that are taken
	- Other operators will come later, but in Civilization AI can determine what upgrades you have an build counters to it (as another example of an operator)
**Path Planning**
- States:
	- Location of Agent in space
	- Discretized Space
		- Could include tiles, voxels, floor locations, waypoints, navmesh, etc.
- Operators:
	- Move from one discrete location to the next
	- Modifies state of modeled world, hopefully moving towards a goal state
	- Graphs facilitate formal planning
**Basic Path Planning Strategy**
- Create a data structure (graph) that facilitates the path planning, discretizing the game world if necessary
- Quantize the agent and goal locations to the graph
- Perform path creation with a search algorithm
- Localize the path back to the game world
- Optionally clean up the path to look more natural
- Agent moves to follow the path until the goal is reached
**Path Planning Algorithms**
- Must search the state space to move the agent towards the goal state
- There are computational issues that arise from this:
	- Completeness - will it find an answer if one exists?
	- Time complexity - How long will it take?
	- Space complexity - How much memory will it take?
	- Optimality - Will it find the best solution?
**Search Strategies**
- Blind Search
	- A naive approach where no domain knowledge is considered and only the goal state is known
	- Depth First Search is a blind search, and is not optimal for games
	- ![](../Images/Pasted%20image%2020260523130843.png)
	- Breadth First Search can return a good result if the edges of the graph are all the same weight (or unweighted)
	- ![](../Images/Pasted%20image%2020260523131250.png)
	- 
- Heuristic Search
	- Domain knowledge is represented by heuristic rules
	- Heuristics drive low-level decisions
	- Heuristics can be leveraged to improve performance
**Discretization of Continuous Space**
- Grid spaces are already discretized, but games that do not use grids are not inherently discretized
- NavMeshes can be used to discretize the space, but you also need to consider validity, quantization, localization, agent movement, and search efficiency (at the very least)
- Booleans can be used for traversable or non-traversable space, but gradients can also be used for uphill/downhill or depth quantization
**Discretized Space - Grid Lattice**
- When generating a grid lattice, verify:
	- World boundaries don't go through the grid lines
	- No obstacle point within a grid cell (if there is, mark it as non-traversable)
	- Obstacle edges do no intersect grid cell edge 
	- ![](../Images/Pasted%20image%2020260523133657.png)
	- Basically, mark the cell as non traversable in any of the cases above (if there is a red dot)
	- As soon as there is one red dot (intersection) we can mark the cell as non traversable and not test for the remainder (to save operation time)
	- ![](../Images/Pasted%20image%2020260523133849.png)
	- Also consider the above cases when generating the grid lattice
	- It may also be worthwhile marking individual edges as traversable or non-traversable
**Grid Cell Point Containment**
- We could treat each grid cell as a polygon and test for intersections that way, however if the grid is axis aligned we can perform range checks for the x and y dimensions instead
- Use appropriate less / greater than tests to determine if points are inside of cells
**Problematic Edge Case**
- ![](../Images/Pasted%20image%2020260523134232.png)
- If you don't want the orange cells marked as untraversable, you can test for certain types of intersection (see proper vs improper)
- In the case where you have a polygon that is exactly the same size and aligned with a grid cell, 

**The Problematic Case**
- The obstacle edge passes exactly through the **corner/vertex** of a cell.
- Or the polygon edge overlaps perfectly with a grid edge.
This creates an ambiguity.
A naïve algorithm might ask:
> “Does the polygon intersect the cell?”

But if the polygon only touches a corner or lies exactly on an edge:
- some algorithms say “yes”
- some say “no”
- floating-point precision can make it inconsistent
That means:
- the cell might incorrectly remain traversable
- a path planner could squeeze through impossible geometry
**What “proper” vs “improper” intersection means**
- Proper intersection
	- Two line segments cross at an interior point.
	- X shape intersection.
	- This is easy to detect.
- Improper intersection
	- Segments touch only at:
		- endpoints
		- corners
		- collinear overlap
	- touching at a vertex
	- lying on top of each other
	- These are the nasty edge cases.
	- Many simple line intersection tests ignore these.
**What the professor means by:**
> “shrink the cell integer dimensions by 1 unit”

Instead of testing against the exact grid cell boundary:
- slightly shrink the cell inward
- then perform intersection tests
So if an obstacle merely touches a corner exactly, it no longer counts ambiguously.
This prevents false negatives where the obstacle “just barely” touches a cell.

**Why the polygon containment test helps**
Your professor says:

> “Poly point containment test then catches this case”

Meaning:

Even if edge intersection fails:

- you can still test whether a cell center (or some sample point) lies inside the polygon.

**So the algorithm becomes:**

1. **Check edge intersections**
2. **ALSO check:**
    - **is the cell center inside the polygon?**

That catches cells fully engulfed by the obstacle even if no edges intersect.

**Validity**
- For continuous space while still using a grid lattice, you can test if agents can travel from one point to another within the grid:
- ![](../Images/Pasted%20image%2020260523135420.png)

**Off Grid Agents**
- What to do if an AI Entity is off a navigable grid cell?
	- Local character movement with raycasting and straight line movement can help return the agent to the grid
	- Cells marked as non traversable can still have metadata denoting edges that are fully / partially passable
	- Teleport the agent back to the grid
**Continuous Position and Movement vs. Path from Discrete Graph**
- Awkward and blocky movement can results from a grid lattice, where agents that are not aligned to waypoints / cells on the grid can move away from the target initially to re-align themselves.
- Agents might want to move to the center of each grid cell instead of following the optimal path through continuous space within the grid
- **String Pulling** in post processing can assist with this, but it depends on how the space is discretized
- ![](../Images/Pasted%20image%2020260523140617.png)

**Summary**
- Grid Lattice Pros:
	- Easy to implement
	- Work well for tile based games
	- Supports temporary obstacles easily (other units moving in the world)
	- Implicit node/edge representation with on demand querying
- Grid Lattice Cons:
	- Not great for continuous environments
	- Can be a burden on search efficiency
	- Poor path quality
