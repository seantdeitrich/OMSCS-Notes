## Basic Definitions
**Software Design** - the process of building a program while satisfying a problem's functional requirements, and not violating its non-functional constraints.
- Design - Deliberative, purposive planning
- Engineering -Skillful or artful contrivance applying scientific and mathematical principles
- Craft - Skilled occupation
- Art - Use of skill, taste, and imagination in the production of aesthetic objects

What is the difference between software design and programming?
- Scale
- Emphasis on non-functional requirements

Software Design is usually broken down into two phases:
- **Architectural Design** - The process of identifying and assigning the responsibility for aspects of behavior to various modules or components of a software.
- **Detail Design** - The process of specifying the behavior of each of the system components that you've identified during architectural design.
	- Here are some core elements or examples of Detail Design:
		- Pseudo code - Program Design Language (PDL)
		- Structured Programming - Sequences, conditions, repetition and chunking
		- Flowcharts - Directed graphs where nodes are computational units and the arc is the flow of control
		- Decision Tables - Rules, conditions, and actions
If you were designing for performance, would you use an array or a collection of interdependent objects to hold data about weather for a region?
- If I was strictly designing for performance, most likely an array since it's easier to iterate through than a collection of interdependent objects.
- More realistically, I would use interdependent objects for their flexibility.

What are the negative consequences of testing a design for **the first time** once the project has been completed? In other words, what are the consequences of not testing before the project is completed?
- It is not cost or time effective. Earlier testing allows for earlier detection of design issues.
- Note that the designers themselves shouldn't be the ones necessarily testing, since they may be blind to other options. Independent design reviewers are preferred.

Note that when designing, it's easy to focus on function behavior and more difficult to focus on the non-functional constraints.
- Specification defines what the project will do
- Design determines how the project will function

Design Documentation is also essential. Scale and complexity must have good design documentation.