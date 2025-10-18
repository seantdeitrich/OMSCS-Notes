# Architectural Views

**Logical View** (L4)
- Structural breakdown of computation, communicational, and behavioral responsibilities
- UML Class Diagrams, Interaction Overview Diagrams, and Collaboration diagrams can display a logical view
- Primarily concerned about how the system will execute
**Developmental View** (L4)
- Concerned with packages, classes, sybsystems, libraries, and files
- UML Package Diagrams and Component Diagrams can display a developmental view
**Process View (L4)**
- Processes and threads into which execution is divided
- UML Deployment Diagrams are the primary means of displaying a Process View 
**Physical View** (L4) 
- Machines used for system execution and how processes are allocated to them
- UML Deployment Diagrams and Sequence Diagrams can display Physical Views
**Use Case View** (L4+1)
- Important execution sequences from the external actor's point(s) of view
**Feature View**
- Conceptual units from the user's point of view
- Conveyed with Feature Diagrams
- Example feature diagram:
- ![](../../Images/Pasted%20image%2020251018123236.png)\
**Non-Functional View**
- How non functional requirements affect the software architecture
	- This could include performance constraints, security details, extensibility, cross-platform compatibility, etc.

## Text Browser Exercise Summary
**Phase 0** - Specify Properties
- Construct a context diagram
- Indicate external actors but only one activity, the system itself
- Indicate external stimuli (events) that can effect the system
- Indicate how the system communicates its results back to the external actors (percepts)
- Specify in English the behaviors you want the system to have
**Phase 1** - Compentize
- Decompose the system into components
- Allocate responsibilities to them
- Handling of events
- Delivery of percepts
- Provision of the property guarantees
**Phase 2** - Determine Architectural Styles
- Determine how the components will interact
- For layered architecture
- Assign the components to layers
- Determine the dependencies between the layers
- Update the guarantees
- Select an invariant maintenance strategy