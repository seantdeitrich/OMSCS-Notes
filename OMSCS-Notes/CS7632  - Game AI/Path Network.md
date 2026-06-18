- Uses nodes and weighted edges in the world with no implicit structure
- Each node has it's own position, and edges are just weighted paths between nodes
- Reduces memory and path search burden, and the 'jagged' nature of paths found on a grid lattice
- Manual placement and automatic placement of nodes are both solid options, but it's usually most efficient to do both

## New Agent Movement Paradigm
- With a path network there is no navigable area defined, only connected line segments
- The agent is therefore allowed two states of movement: **Local Movement** and **Remote Movement**
- **Local Movement** is when the agent makes decisionss based on when the target is in sight
- **Remote Movement** is used when accessing something out of sight -- This is where agent queries for a path on the network
	- The agent can be on the path or off the path during the remote movement state
	- Once the agent is off the path, it relies on local movement to get back on the path at the position it left at `leftPosition`
- To use a path network, local movement must be effective at detecting obstacles and have reliable movement on terrain of various types (or whatever makes sense for the game)
- Finding the closest node in the network is a common issue
	- You can naively check all nodes in the graph, but for efficiency you can use a grid lattice to group nodes into 'neighborhoods', and then check the neighborhood the agent is in for the closest node
- Finding the closest visible to to the goal is a necessity as well

## Automatic Node Placement
- The **Flood Fill** process starts with a seed, and expands by uniformly adding nodes as it expands from the seed
	- Nodes and edges must be far enough from walls and obstacles when compared to the agent radius
	- Note that this process can create a dense grid like structure which loses out on the memory savings that we're trying to achieve
- **Points of Visibility** is another process that uses convex angle obstacles to create **inflection points** for efficient paths
	- Optimal paths will always have inflection points at convex vertices, and those points are good candidates for path nodes
	- ![](../Images/Pasted%20image%2020260530121744.png)
	- The points must be offset by the radius of the agent
	- For this process, the vertices define the nodes, then a brute force process is used to determine visibility between nodes (to build the edges)
	- This process tends to find natural and efficient paths ![](../Images/Pasted%20image%2020260530122119.png)
	- One weakness of this approach is that nodes and edges can grow to a very large size, and then need a lot of manual tweaking
- **Dynamic / Breadcrumb Networks** are built by agents dropping waypoints at important locations
	- It starts with a waypoint at the home location as the first node in the graph
	- Each `Think()` update, it checks to see if from the current position you can raycast cleanly to the last waypoint
		- If yes, then you store as `lastSafePos` variable
		- If no, add the `lastSafePos` to the grpah as a new node and connect to `lastWaypoint`, then set `lastWaypoint` to `lastSafePos`
	- The cleanup process for this method can be expensive, and runaway waypoint formation can eat up memory
	- However, this method can be used for procedurally generated worlds where the paths can't be baked into the map
## Building a Path Network from Candidate Nodes
- With our nodes places (automatically or manually), we now must evaluate whether obstacles are in the way of edges between those nodes
- If we consider edges to be **corridors**, the corridor size is defined by the agent radius
- We can use a combination of `pointDistanceToLineSegment` tests and `lineSegmentLineSegmentIntersection` tests
- ![](../Images/Pasted%20image%2020260530122849.png)

## Greedy Steering
- You can improve the agent movement by steering or skipping ahead to the furthest node in the path that the agent can see
	- Raycasts can be used for this, "but watch out for pits"
- Not a perfect fix, and ray casting can easily become expensive
- Especially problematic on variable height terrain or different terrain types

## Path Network Summary
- Advantages:
	- Discretization of space can be very small in memory
	- Doesn't require the agent to be at one o f the path nodes at all times (unlike grids)
	- Allows for switching between remote and local navigation
	- Works well with continuous movement agents, and in FPS and RPGs
	- Nodes can include useful metadata
- Disadvantages:
	- Agents in local movement might not be able to see a node in the network, and therefore won't be able to switch to remote movement
	- Jagged path shapes and backtracking
	- Issues with dynamic and rolling terrain
	- Potentially puts a lot of real time computation pressure on the agent rather than on pre-processed discretization
