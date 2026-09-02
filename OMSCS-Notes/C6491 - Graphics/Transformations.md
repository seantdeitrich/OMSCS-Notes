Transformations are operations in Euclidean vector spaces that alter the configuration of points or vectors. They might affect position, orientation, or scale of points. They can be represented with matrices.

**Linear transformations** preserve linearity and origin.
For example:
- Identity
- Rotation
- Scale
- Shear
- Reflection
However, translation is not a linear transformation.
Linear transformations are additive and homogenous.

**Conformal Linear Transformations** are linear transformations that also preserve angles and shape. For example:
- Identity
- Rotation
- Uniform Scale
- Reflection (improper due to the change in orientation)
Non-uniform scaling, shearing, and translation are not conformal.

**Similarity Transformations** are transformations that preserves the shape and proportions of objects, allowing for changes in position, orientation, and uniform scale. These are also sometimes called **Rigid Transformations** or **Rigid Body Transformation**.

**Affine Transformations** preserve collinearity and parallelism, but are not considered linear because the origin is not preserved in affine transforms. An affine transformation represents a combination of one or more linear transformations followed by a translation.

## Scale
This is a self explanatory concept except for **scaling around points**.
Scaling can give unexpected results because the origin is the only point that remains fixed during scaling. To control the scaling effect, it is common to:
1. Translate the desired anchor point of the shape to the origin
2. Apply the scaling operation
3. Translate the shape back to its original position
A similar approach can be used for rotation around an anchor point. 
Shapes may also be stored such that the origin is already at the desired anchor point.

## Shear
A shear transform 'slides' points in a specific direction, distorting the shape while preserving area.
This is common in graphics for creating slanted shapes or simulating perspectives.
Shear is linear: the points remain on the same line relative to the direction of the shear.
Instead of multiplying like that which happens when scaling, shearing simply adds the shear factor sh<sub>x</sub> and sh<sub>y</sub> to each point.

## 2D Rotation
Moves points around the origin by an angle theta (in radians), preserving the distance from the origin. 
$$p_{x_{new}}=p_x\cos(\theta) - p_y\sin (\theta)$$
$$p_{y_{new}}=p_x\sin(\theta) - p_y\cos (\theta)$$
Note that rotations are negative for clockwise angles and positive for counterclockwise angles.
As a reminder:
$$\theta_{radians} = \theta_{degrees}\cdot \pi/180$$
To determine the angle of a vector with the x-axis, we can use the following formula:
$$\theta = \arctan(y/x)$$ In graphics APIs, `atan2(y,x)` is used to separately track the sign of x and y to avoid dividing by zero and correctly solving the angle for all four quadrants.

## Matrix Multiplication
Points and vectors can be multiplied by a transformation vector if the dimensions agree:
$$ \begin{bmatrix} a_{11} & a_{12} \\ a_{21} & a_{22} \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} a_{11}x + a_{12}y \\ a_{21}x + a_{22}y \end{bmatrix} $$
Matrix multiplication is not commutative, meaning the order of multiplications matters. However, it is associative and distributive.
Matrix multiplication of matrix *M* with *a* rows and *b* columns by matrix *N* with *b* rows and *c* columns, results in a matrix *R* with *a* rows and *c* columns. If the columns of M do not match the rows of N, the multiplication is undefined. 
Each element in the resulting matrix *R* is calculated as the **sum of products** of the corresponding elements from the *i*th row of M and the *j*th column of N. 
$$ r_{ij} = \sum_{k=1}^{b} m_{ik} \cdot n_{kj}$$
k iterates over the shared dimension *b*.

Example: 
![](../Images/Pasted%20image%2020260902175122.png)
To get the first element of R in this example, the following sum of products is used:
$$1*1+2*0+4*1 = 5$$

The identity matrix (essentially 1 as a matrix) is a matrix where all diagonal elements from top left to bottom right are 0. We can scale uniformly by changing that to the scaling factor *s*, and can also represent shear with the following:
$$\begin{aligned} \textit{shear\_x}(\theta) &= \begin{bmatrix} 1 & \tan\theta \\ 0 & 1 \end{bmatrix} \\ \textit{shear\_y}(\theta) &= \begin{bmatrix} 1 & 0 \\ \tan\theta & 1 \end{bmatrix} \end{aligned}$$

And rotation with the following:
$$R(\theta) = \begin{bmatrix} \cos \theta & -\sin \theta \\ \sin \theta & \cos \theta \end{bmatrix}$$
## Basis Vectors
The columns of the matrix represent the **basis vectors** of the coordinate system or frame.
The standard basis is reflected in the identity matrix:
$$\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$$

Where the x axis is: (1,0) and the y axis is: (0,1).

## Homogenous Coordinates
Similarity and Affine transformations can be supported with homogeneous coordinates which add an extra dimensions to points and vectors. The homegenous coordinate is typically referred to as: *w*.

The main reason standard $2\times2$ (or $3\times3$ in 3D) matrices fall short in computer graphics is that **linear transformations cannot perform translation (moving an object)**.

**The Core Problem**

A standard $2\times2$ matrix multiplication can easily handle scaling, rotation, and shearing:

$$\begin{bmatrix} a & b \\ c & d \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} ax + by \\ cx + dy \end{bmatrix}$$

However, if you want to shift a point by $(t_x, t_y)$, you cannot do it with matrix multiplication alone. You have to add a separate vector:

$$P_{\text{new}} = M \cdot P + T$$

Because translation requires an extra addition step, you cannot chain multiple operations (like rotate, then scale, then move, then rotate again) into a single combined matrix. Every transformation step would require its own separate multiplication and addition, which quickly becomes computationally expensive.

**The Solution: Homogeneous Coordinates**

By adding a fake extra dimension—the $w$-coordinate—you convert translation into a linear shear operation in a higher dimension.

This expands a 2D matrix into a $3\times3$ matrix, allowing you to embed the translation offsets directly into the rightmost column:

$$\begin{bmatrix} a & b & t_x \\ c & d & t_y \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ 1 \end{bmatrix} = \begin{bmatrix} ax + by + t_x \\ cx + dy + t_y \\ 1 \end{bmatrix}$$

Now, **translation is just matrix multiplication**.

## Rotation and Scaling with Homogeneous Coordinates

**Rotation in a Right Handed Coordinate System**:
- Rotation about z axis:
$$R_z(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta & 0 & 0 \\ \sin\theta & \cos\theta & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
- Rotation about x axis:
$$R_x(\theta) = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & \cos\theta & -\sin\theta & 0 \\ 0 & \sin\theta & \cos\theta & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
- Rotation about y axis:
$$R_y(\theta) = \begin{bmatrix} \cos\theta & 0 & \sin\theta & 0 \\ 0 & 1 & 0 & 0 \\ -\sin\theta & 0 & \cos\theta & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

**3D Scaling:**
Scaling matrix definition:
$$S = \begin{bmatrix} s_x & 0 & 0 & 0 \\ 0 & s_y & 0 & 0 \\ 0 & 0 & s_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
Applying scaling to a Point p:
$$
p = \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix}, \quad p' = S \cdot p = \begin{bmatrix} s_x & 0 & 0 & 0 \\ 0 & s_y & 0 & 0 \\ 0 & 0 & s_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix} = \begin{bmatrix} s_x \cdot x \\ s_y \cdot y \\ s_z \cdot z \\ 1 \end{bmatrix}
$$

## Rotation Representation
A rotation matrix describes the rotational pose of the major axes. However, it does not indicate the sequence of rotations that produces this orientation, which can lead to strange rotations in game behaviors. This ambiguity arises because there are multiple ways to build a rotation matrix. 

**Extrinsic Rotations** are described in the frame that contains the object. For example, a basketball manipulated externally by a basketball player. Each axis's angle of rotation is applied sequentially relative to the containing frame.

**Intrinsic Rotations** are described relative to the object's evolving local frame. For example, an airplane's control surfaces adjust based on the plane's orientation. Each axis's angle of rotation is applied sequentially, however preceding rotations reorient the local frame, affecting how subsequent angles are measured.

Think of these as the difference between global and local rotations in a game engine.

To build a rotation matrix:
- **Extrinsic Method**:
	- Choose a rotation order
	- Apply the rotations sequentially in global coordinates
	- Multiply the corresponding matrices together
- **Intrinsic Method**:
	- Rotations are described in the evolving local frame
	- Choose a rotation order
	- Rotate the object around the first axis in the local frame
	- Update the frame for the next rotation
	- Repeat for all axes, measuring angles in the updated frame
Both methods can build the same rotation matrix as the other, but serve different use cases.

### Euler Angles
Euler angles are a sequence of rotations around the principal axes. These include:
 - **Yaw** - Rotation around the z-axis (turning left/right)
- **Pitch** - Rotation around the y-axis (tilting up/down)
- **Roll** - Rotation around the x-axis (twisting left/right)
Euler angles can be applied intrinsically or extrinsically. 

**Gimbal Lock** occurs when two rotation axes align, reducing the degrees of freedom. For example, pitching 90 degrees aligns the roll and yaw axes. This creates a situation where further rotation becomes difficult or impossible to track accurately. 

### Axis-Angle Rotations
An **Axis-Angle** rotation is a rotation using a single axis of rotation (unit vector). This helps us avoid gimbal locks. The axis however can be arbitrary.
The approach goes as such:
1. Align the rotation axis with a major axis
	- Apply a series of transformation to align the arbitrary axis with the z-axis (or any principal axis)
2. Perform the rotation
	- Rotate about the aligned axis
3. Undo the alignment
	- Reverse the initial transformations to return the rotated object to its original frame

### Quaternions
A Quaternion is a compact 4D representation of rotation using a scalar part and a 3D vector part.
Quaternions avoid gimbal lock inherently, are efficient for interpolation, and computationally stable chaining rotations.

To combine quaternions with transformation matrices, first combine the quaternion angle into a rotation matrix. Then combine with the translation and scaling for a complete transformation matrix. 

## Compound Transformations
Compound transformations are sequences of transformations applied in a specific order. This can include translation, rotation, scaling, etc.
**Order matters**. Transformations are not commutative. The common convention for compound transformations is to use the following order:
$$T\cdot R \cdot S$$
This order scales first, then rotates, then translates. Although TRS is the matrix multiplication order, SRT is the conceptual order of transformations.

### Decomposability of Similarity Transforms
**Similarity Transform Properties:**
- A similarity transform is any combination of T,R,and S where S is uniform and non-degenerate
- Such transformations can always be decomposed into single T, R, and S
- This means that any compound transformations can always be represented by a single transformation matrix

To decompose compound similarity transforms - 
- Translation (T) - Extract the last column of the homogeneous transform matrix
- Rotation (R) - Determined using the normalized basis vectors of the upper-left 3x3 submatrix
- Uniform Scaling (S) - Obtained from the magnitude of any basis vector in the upper-left 3x3 submatrix.
More powerful methods such as Singular Value Decomposition (SVD) exist that work with any n by n matrix.
### Affine Transformations and Shear
Affine transformations include translation, rotation, scaling (non-uniform and uniform), and shear. Combining sequences of non-uniform scaling and rotation can introduce shear. You cannot represent transformations with just TRS in systems with hierarchical transformations if scale can be non-uniform.

## Inverse Transformations
Given a transform T, multiplying T by the inverse of T results in the identity matrix. This is useful for undoing transforms and transitioning between coordinate spaces.
- For inverting scales, the inverse matrix would include the reciprocals of the scaling factors.
- For rotation, rotating back by the same angle undoes the original rotation.
- For translations, you simply shift in the opposite direction to undo the translation.
For composite transforms, you must apply the inverses in the correct (reverse) order to properly undo transformations. So for a typical TRS composition, you would apply the inversion as SRT.

Inverses might not exist in certain situations, like annihilation (e.g. scaling by zero). To detect this, we use the **Determinant**. 
The **Determinant** indicates the scaling factor of a transform and reveals if any dimensions are collapsed. If det(T) = 0, the transformation is non-invertible.

Note that numerical accuracy can become an issue with inversions due to floating point arithmetic. Errors in inversion grow when the determinant of the matrix is very small. For example, scaling one axis by 10<sup>-9</sup> effectively collapses that dimension, making inversion numerically unstable.  

The **Condition Number** of a matrix measures how sensitive a matrix is to numerical errors during inversions. High condition numbers amplify small numerical inaccuracies, leading to unreliable inversions.
### The Determinant Method
The determinant method works well for 4x4 homogenous matrices and is generally efficient.
To computer the inversion:
1. Compute the adjoint matrix
2. Divide by determinant if it is not zero
For large matrices, avoid this method because it becomes computationally expensive and less practical. 




