# Object Constraint Language
- OCL is a specification language that is declarative and strongly typed
- OCL extends UML with
	- Class invariants
	- Pre/Post conditions
	- Guards on state-machine transitions
- OCL is a declartive language, not procedural
	- There are no assignment statements or other side effects
- Syntax:
	- `context <identifier> <constraintType>:<Boolean expression>`
		- The context could be a class or method, and identifier is the name of the context
		- constraintType is one of the following:
			- Precondition `pre`
				- The conditions that must be in place before an operation
			- Postcondition `post`
				- Shows the guaranteed effects after an operation completes
			- invariant `inv`
		- The boolean expression is the actual constraint
- OCL constraints are inherently connected with class model diagrams

**Invariants** are statements of properties that are always true. It is a way to express key system requirements.
- The keyword used to indicate an invariant is `inv`
- `context LargeCompany inv: numberOfEmployees > 50`
	- This means all large companies must have more than 50 employees
	- Invariants are also called **Integrity Constraints**

*Honestly, just google it if you feel you need it. The TA's have said that OCL is optional for all diagrams in the class.*