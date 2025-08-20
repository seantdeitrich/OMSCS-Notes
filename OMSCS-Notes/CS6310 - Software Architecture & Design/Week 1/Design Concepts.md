# Design Concepts
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
- Military organizations require formal documentation
- Small exploratory research teams may not need it

Traditional **Design Documentation** includes all **Design Rationale**:
- Subcomponents
- Processes and Activities
- Data and Data Flow
- Control Flow and Regime
- Performance Considerations and Resource Consumption
- Dependencies amongst components
- Tradeoffs among non-functional constraints
- Assumptions about users, hardware, and customer base
- Stakeholders
- Issues during the course of design 
	- Possible resolutions, analyses of those solutions, rationales, impacts, and conflicts
- Temporal relations
	- Histories, versions, schedules, transformations, revisions, releases, and variants
- Constraints
	- Internal, external, requirements, and specifications
- Aggregates
	- Configurations, packages, and components

**Design Decisions** are explicit choices of how to trade off two non-functional aspects of a design, such as speed versus size.

**Coupling** - the extent to which components depend on each other for successful execution
- Aim for low coupling
- Changes to one module should not affect other modules
- Java's package support is a good for reducing coupling
	- It allows programmers to only import the necessary dependencies
- Java's class inheritance mechanics actually increase coupling, because the child class depends on the parent class.

**Cohesion** - The extent to which a component has a single purpose or function
- High cohesion is good

Note: The video states that the Java packages system reduces coupling but does not increase cohesion. I believe this to be false since different classes can be packaged together to produce a cohesive package with one function.

**Information Hiding** - Encapsulating the capabilities that a particular module has behind an abstract interface. It seems like this is the same as abstraction.
- If you're dealing with a system that has access to many hardware devices, hiding access to those devices behind an abstract interface is an example of information hiding.

**Abstraction Mechanisms** 
- **Declaration (Declarative)** defines what something does, but now how it is implemented
- **Aggregation** defines a container, but not the contents of the container
- **Generalization** allows abstraction of special cases, this is found in the class hierarchy of OOPLs
- **Parameterization** allow abstraction of various possible values and function calls
- **Non-determinism** leaves choices unspecified

**Design Philosophy**
- Use Models
- Understand social context of design solutions (User-Centered Design)
- Consider different tools for accomplishing goals
- Use language, vocabulary, and metaphors to naturally deliver understanding
	- Think of icons, firewalls, client-server, etc.

