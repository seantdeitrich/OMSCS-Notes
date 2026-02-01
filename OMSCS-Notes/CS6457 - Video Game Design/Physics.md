## Physics Engines
- A physics engine provides physics simulations in a virtual environment.
- There are two types: High Precision and Real Time.
- Havok, Newton, ODE, NVidia PhysX are all examples of real time physics engines.
- Unity uses PhysX.

## Components
- Physics engines usually include the following components:
	- Bodies (Rigid Bodies)
		- Box, Sphere, Capsule, Triangle Mesh, etc.
		- **Dynamic** bodies are ones that are affected and moved by the physics engine
		- **Kinematic** bodies programmatically controlled, or controlled by the animator
		- **Static** objects are unmovable (like floors / walls)
		- Bodies can have Positions, Orientations, Velocity, Angular Velocity, Mass, Friction, Restitution, softness, etc.
		- They can also have mesh shapes with per triangle materials (like terrain)
	- Connectors
	- Forces
	- Constraints
	- Collision Detection
		- Collision hulls reduce complexity of collision calculation
		- The hulls are made from the primitive body shapes
		- Complicated shapes can have more expensive collision calculations, so generally you want to simplify your characters down to capsules or other primitive shapes if possible
		- You can have Unity calculate a convex shape to contain a more complex geometry
		- You can also make a compound collider manually with fixed joints and connectors
		- With hierarchical collision, you can use have multiple boxes and test for collisions in a tree like fashion (this helps average performance at the cost of a worse worst case scenario performance)
	- Layers and Group Membership can be used to control collisions
	- Geometries can have a 'skin' layer to deal with jitter and relax collision constraints

## Discrete Collision Detection
- Given that an object is moving a set amount of distance per frame, it usually will not have a perfect collision with another object, and therefore will move partially into it.
	- However, the moving object might simply pass through a skinny wall if it is moving quickly. This is called **Tunneling**.
- **Tunneling** can be alleviated with continuous collision detection which can be implemented with:
	- A single raycast in the direction of movement, however this can fail if the ray doesn't hit the wall (if it's slight above or below).
	  ![](../../Pasted%20image%2020260201155241.png)
	- This approach can also fail at 'seams' or wall joins:
	  ![](../../Pasted%20image%2020260201155358.png)
	- You can use multi raycasts to help alleviate these issues, but it becomes more expensive 
	- **Silhouette Extrusion** can be used instead of multiple rays. This involves using another collider based on the silhouette of the object:
	  ![](../../Pasted%20image%2020260201155621.png)
		- This is easy enough with spheres, but harder with other shapes.
- **Speculative Collision Detection**
	- Used by PhysX to help alleviate tunneling
	- PhysX will dynamically expand and contract skin width depending on the properties of the object. Small objects moving quickly will grow larger skins to help aid in collision detection.
	- However, fast accelerations can still cause tunneling to happen because the skin didn't have enough time to reshape.
- Fast rotations can make all of the previous techniques for solving tunneling tricky
	- PhysX actually sets default maximum velocities and angular velocities
	- `rigidBody.maxAngularVelocity` is the property to control in this case if needed
- **Dynamic Substeps** can also be used to alleviate tunneling, which allows for faster moving objects to have a higher number of checks (or essentially a higher framerate) relative to the speed. Substeps can also be increased when an object is a 'candidate' for collision, or in other words within a certain area and expected to collide soon.

## Collision Dynamics
- Collisions should have results that are largely consistent with the real world. Bigger objects should push smaller ones out of the way for example.
- **Penalty Force Method** was one of the early approaches to this:
	- It allows for interpenetration (objects moving into one another), and then correcting with a force.
	- Only the deepest penetration is considered for the collision
	- This had issues with oscillation, where one corner of an object would be pushed and another part of the object would then collide with the original object.
	  ![](../../Pasted%20image%2020260201161432.png)
	- This oscillation results in an 'air hockey' affect where things will slide around based on small oscillations.
	- This was a decent solution for destruction mechanics, but not an overall physics simulation.
- **Improvements on Penalty Force:**
	- Relax contact criteria by implementing skin width
		- Contact is identified upon skin interpenetration, not actual geometry
		- Soft collision correction at the start that becomes stronger with further interpenetration
		- Reduces jitter, and allows for friction simulation
	- Consider all collision contacts, not just the deepest:
		- This is implemented by using the object perimeter vertices as contact points, rather than a whole 3D intersection shape.
	- Find a solution that addresses all constraints introduced by the collision contacts
- **Real Time Solutions**
	- Based on the Linear-Complementary Problem (LCP) with Jacobian constraints (not important to know the math)
	- This method applies corrective sequential impulses for each contact point independently, then determines the error for each contact if all corrections have been applied
	- ![](../../Pasted%20image%2020260201163729.png)
- **Forces**
	- The most common in video games is gravity
	- Constantly applied
	- Have a vector for direction and magnitude
- **Impulses**
	- Instantaneous, single frame force.
	- Usually used for jumping.
	- Have a direction and magnitude
- **Connectors**
	- How shapes are attached to each other
	- Restricts motion between actors, rotation, and translation
	- Allows for multibody dynamics
	- Joints can have flexibility, strengths, degrees of freedom, springiness, etc.
	- Can have different types of joints like pulley, hinge, spherical, and prismatic
	- Joints are implemented as constraints
	- **Hard Constraints** are never violated
	- **Soft Constraints** are designed to be violated
- **Deactivation** should actively be used to minimize the amount of active simulation
	- Objects that are thrown and lose their energy should 'Sleep' 
	- Set bounce thresholds to stop vibration, and end bouncing when the object is out of view
- Physics and integrate with the animation system to allow for more realistic movement
	- Toggling ragdoll physics on and off can be useful (disabling / re-enabling the animator and setting stiffness of the skeleton to 0)
## General Guidance
- Set the gravity vector
- make a ground surface that doesn't fall
- Apply forces to bodies instead of manually adjusting their transforms
- Adjust joint parameters as needed
- Call collision detection
- Step the simulation based on time
- Keep the graphics object and physics object in synch
- Deactivate objects (manually or automatically)
- Use layers effectively