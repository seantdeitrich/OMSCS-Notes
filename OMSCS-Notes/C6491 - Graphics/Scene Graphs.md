**Scene Graphs** are used to manage hierarchical relationships and transformations in complex scenes. Transformations applied to a parent node affect the mapping of its descendant nodes to world coordinates. Local coordinate transforms for any node are always defined relative to the parent coordinate system. 

Scene graphs are implemented with a directed acyclic graph (DAG) with:
- Geometry or objects represented as nodes
- Transformation matrices computed at each node via matrix stack, defining the geometry or object's local-to-world transformation

Setting up scenes using this hierarchy allows for transformation chains, where moving a parent also moves all of its children:
![](../Images/Pasted%20image%2020260908163729.png)

However, it is no longer common to impose this rigid hierarchy for entire scenes. Traditional hierarchies offer less flexibility, although the spacial relationships are still useful for compound objects like a base robot object that contains an upper arm, lower arm, etc. Parts can then be independently animated but still connected spacially.

## Matrix Stack
The purpose of a matrix stack is to allow hierarchical transformations in scene graph traversal.
There are two stack operations:
- `pushMatrix()` saves the current transformation matrix and makes a copy to the top
- `popMatrix()` restores the previous transformation matrix
- Note that the push and pop calls must be balanced
New transformations are multiplied to the right side of the active transformation:
$$M_{new} = M_{current}\cdot T$$
This ensures that more local transformations are applied first to an object before ascendant transformations.

In the room hierarchy seen above, here is what the matrix stack would look like:
![](../Images/Pasted%20image%2020260908164312.png)

An example of transforming a triangle mesh within a hierarchy would happen as such:
$$p_i = [x_i, y_i, z_i,1]^T$$
$$p_{i_{transformed}} = M_{room}\cdot M_{table}\cdot M_{saucer}\cdot M_{cup}\cdot p_i$$
### Building Transforms via Immediate-Mode API Approach
The goal here is to construct the transformation matrix M<sub>object</sub> for a scene object using matrix stack operations. 
1. Start with the identity matrix: M = I with `loadIdentity()`
2. Apply a translation: `translate(Tx,Ty,Tz)`
3. Apply a rotation: `rotate(angle, axis)`
4. Apply a scale: `scale(Sx,Sy,Sz)`
Then the result is:
$$M_{obect}=T\cdot R \cdot S$$

### Scene Graph Transforms Declaratively
In this approach, a declarative scene graph is defined in a format such as JSON. The structure specifies the geometry, local transformations, and parent/child relationships. Before rendering, the scene graph is analyzed, and local-to-world and world-to-local (inverse) transformations are generated and cached. A separate rendering pass leverages the cached transformations. Logic similar to a matrix stack is used to generate the transformations, but this is typically an internal process.

## Inverse Transforms in Scene Graphs
In a scene graph, inverse matrices allow us to coordinate world coordinates to local coordinates. This can then be used for ray tracers to cast rays using those local coordinates rather than the global coordinates for simpler intersection tests.

The challenges with inverse transforms in scene graphs is that they must be inverted and applied in reverse order. Efficient handling is essential for dynamic or complex scenes.

There are two approaches to tracking inverse transforms in a scene graph: **Determinant-Based Inversion** and **Tracking Individual Transform Operations**.

**Determinant Based Inversion**:
- Precompute the inverse of the full  transformation matrix at each level
- Works for any invertible transformation, regardless of composition
- Easy to implement and is the status quo
**Tracking Individual Transform Operations**:
- Track translation, rotation, and scaling all separately
- Compute the inverses per operation and accumulate them in reverse order
- This requires parallel matrix stacks (an additional matrix stack)
- Potentially more efficient in some cases, but this approach is uncommon

## Scene Graphs and Matrix Stacks
When using a scene graph and a matrix stack there are two main objectives
- Computing forward transformations from local space to world space for each node
	- These transforms are computed and stored in a matrix stack as the scene graph is parsed
- Computing inverse transformations from world space to local space for each node
	- A parallel inverse matrix stack could be used, or we can compute and store the inverse of the forward transforms

## Changing Coordinate Systems
Geometry can be transformed between different levels in a scene graph: ![](../Images/Pasted%20image%2020260908170522.png)
To accomplish this:
1. Forward the transform the object to world space by applying all the transformations leading to that object
2. Invert to the desired level
![](../Images/Pasted%20image%2020260908170638.png)

## Summary
- Scene Graphs
	- Manage hierarchical relationships and transforms
	- Parent transformations propagate to child nodes
- Matrix Stacks
	- Efficiently manage transformations in hierarchical scenes
	- Simplify scene graph traversal

Combined, they simplify rendering and animation of complex scenes while ensuring correctness.