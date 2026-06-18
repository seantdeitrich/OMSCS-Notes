**Floating Point Vectors in Computational Geometry**
- Floats are represented with a sign bit, exponent bits, and fraction bits
- The smallest representable difference between two floats varies
- Rounding errors mean that you cannot exactly represent numbers
- Epsilon values are used as a 'fudge factor' to account for the rounding errors
Integer Solution
- Converting floats to integers can help support more robust computational geometry
- This is achieved by multiplying each float by some number (usually 1000 for 32 bit ints) and then casting to an integer
- Floating point 2D/3D Vectors become IntVectors
- After the computational geometry is performed, map back to floats
- Conversion can create a lot of overhead and double memory usage
- These calculations can be performed outside of the game and baked into geometry data structures when building the game (if possible)
- Integer overflow can happen when converting from floating point vectors to integer vectors
	- Reordering equation terms, processing subsets of geometry such that integers fall within a defined safe range, and the use of 64 bit integers or doubles can alleviate this issue
- Robust computational geometry is challenging, so it's worth looking for literature that covers the problems you face
- Use integer based algorithms when possible to avoid floating point errors
- Avoid angle calculations where possible
- Testing is important, especially if you are building up to more complicated implementation