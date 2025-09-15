# Formal Specification
**First Order Logic (FOL)**, also known as **Mathematical Logic**, enables you to precisely express, combine, and quantify propositions.

**Object Constraint Language (OCL)** provides a syntax for FOL that can be used to annotate UML diagrams.
## Sorting Example

Problem in English:
- Given is a vector of integers named X
- Produced is a vector of integers named Y
- The output vector Y must be ordered
- The context of Y must be the same as the contents of X
	- Everything in X must be in Y
	- Everything in Y must have come from X
	- The number of occurrences of each item in Y must be the same as its number of occurrences in X

**FOL** typically breaks the specification into three pieces
- **Signature** - Gives the name of the program, names and types of inputs, and names and types of the result.
	- For the sort example the signature would be:
	- `Vector<int> Y = SORT(Vector<int> X)`
- **Precondition** - Assertions about the functions input arguments
	- For a square root function, the precondition would be asserting that the input is a real number (unless we're supporting imaginary numbers)
	- For the sort example, there is no precondition
- **Postcondition** - Assertions about the output of a function, usually by relating it to the input.
	- For square root, the postcondition would be:
	- `Y * Y = X`
	- Note that indexing of vectors starts from 1 and goes to n.
	- Note that |Y| indicates the length of Y
	- Note that the dot stands for 'and' or 'it must be the case'
$$ \forall i : 1 \le i \lt |Y| \bullet Y[i] \le Y[i+1] $$

For a function called RORDERED (reverse ordered):
- Signature - `Vector<int> Y = RORDERED(Vector<int> X)`
- Precondition - None (or True)
- Postcondition - 
$$ \forall i : 1 \le i \lt |Y| \bullet Y[i] \ge Y[i+1] $$

**Pure Functions** are functions where the output is completely determined by the input.
For **Impure Functions**, in addition to how the output relates to the input, you should also indicate any side effects. This could be changes to global variables or objects.

 