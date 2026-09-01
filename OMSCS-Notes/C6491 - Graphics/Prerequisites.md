# Prerequisites

## Linear Sum of Points
- Generally a sum of points is *invalid* with two exceptions:
- **Exception 1:**
	- A weighted average of points yields a valid point if the weights sum to one
	- $$ A+ C - B$$ Is the same as A plus vector **BC**
	- Midpoint of line segment with endpoints A and B:
		- $$ C = (A+B)/2$$
- **Exception 2:**
	- A weighted average of points yields a vector if the weights sum to zero
	- $$ P-S+Q-P+E-Q $$ Is the same as vector **SE**

## Vector Operators
### Magnitude
- Magnitude can be acquired with the square root of the sum of squares
$$ ||V|| = sqrt(Vx^2+Vy^2+Vz^2)$$
### Squared Magnitude
$$ ||V||^2 = Vx^2+Vy^2+Vz^2$$
- Often has performance benefits because it avoids the square root
- Squared magnitudes are inherently non linear, so you must be careful if you're using it for anything other than comparing whether one magnitude is larger than another

### Distance Between Two Points
- Form a relative vector between two points
- Then solve the magnitude of the relative vector

### Dot Product
- Also called **Inner Product**
- The dot product measures how much of a vector A lies in the direction of vector B, scaled by the magnitude of B
- ![[Pasted image 20260827200441.png]]
- Dividing the dot product by the magnitude of B gives the scalar projection of A onto B
- $$projBA = (A \cdot B) / ||B||$$
- Produces a scalar that varies given the angle of the two vectors
- $$ A \cdot B = |A||B|cos(\theta) $$
- The dot product of a vector with itself is the same as solving the square magnitude of the vector $$ V \cdot V = ||V||^2$$
- In parametric form:

### Cross Product
- The magnitude of the cross product of two vectors A and B is equal to the area of the parallelogram formed by those vectors
- $$ A \times B = (AyBz-AzBy),(AzBx-AxBz),(AxBy-AyBx)$$
- $$ ||A \times B|| = ||A||*||B|||*|sin(\theta)|$$
### Euclidean Vector Space
- A vector space with additional characteristics that make it useful for describing classical physical reality
- Consists of a set of points and a set of vectors
- **Features**
	- A dot product is defined
	- The magnitude of vectors can be computed
	- The distance between points can be computed
	- The distance between vectors can be computed
	- Typically two or three dimensions

### Frame
- A frame consists of an origin and basis vectors (2D or 3D)
- Can be thought of as a frame of reference for describing the positions of points and vectors within a space
- Often represented as a transformation matrix

### Ambiguity of 3D Vectors
- Can be either right or left handed


