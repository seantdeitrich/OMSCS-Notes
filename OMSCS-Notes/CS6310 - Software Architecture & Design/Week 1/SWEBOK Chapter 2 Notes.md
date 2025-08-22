# SWEBOK Chapter 2 Notes

**Acronyms**

| Acronym   | Name                              |
| --------- | --------------------------------- |
| ADL       | Architecture Description Language |
| CBD       | Component-Based Design            |
| CRC       | Class Responsibility Collaborator |
| DFD       | Data Flow Diagram                 |
| ERD       | Entity Relationship Diagram       |
| IDL       | Interface Description Language    |
| MVC       | Model View Controller             |
| OO        | Object-Oriented                   |
| PDL       | Program Design Language           |
| KA        | Knowledge Area                    |
| D-Design  | Decomposition Design              |
| FP-Design | Family Pattern Design             |
**Software Design**:
- The process of defining architecture, components, interfaces, and other characteristics  of a system or component, and the result of that process
- It is essentially the process of blueprinting a software or solution
- Analyzing designs allows us to determine whether or not they will fulfill requirements before the software is implemented
- It also allows us to evaluate alternative solutions and tradeoffs
- According to IEEE, software design consists of two activities
	- Software Architectural Design - Developing top level structure and organization of the software, and identifying all the components
	- Software Detail Design - Describing components with enough detail in order to aid and facilitate construction of the solution
- The output of the Architectural Design and Detail Design processes is a set of models and artifacts that record major decisions.
	- Explanations and rationales should be included as well for the sake of long-term maintainability

**Software Design Principles**

| Software Design Principle                   | Definition / Explanation                                                                                                                                                                                                                                                                                         |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Abstraction                                 | Includes **Parametrization** and **Specification**.<br>- Parameterization abstracts details of data representations, by representing data as named parameters.<br>-Specification includes procedural abstraction, control/iteration abstraction, and data abstraction (very unclear what these are in chapter 2) |
| Coupling and Cohesion                       | **Coupling** is defined as a measure of interdepence among modules in a program. <br>**Cohesion** is defined as a measure of the strength of association of elements within a module (how focused a package or class is on its purpose).                                                                         |
| Decomposition and Modularization            | Large software should be divided into a number of smaller named components with well defined interfaces that describe component interactions                                                                                                                                                                     |
| Encapsulation and Information Hiding        | Group and pack the internal details of an abstraction, and make those details inaccessible to external entities                                                                                                                                                                                                  |
| Separation of Interface and Implementation  | Separate interface and implementation by defining components with an interface that is available to the clients. That interface is separate from the details of how the component is implemented.                                                                                                                |
| Sufficiency, Completeness and Primitiveness | Achieving sufficiency and completeness ensures that a software component captures all important characteristics of abstraction, with no additional features. Primitiveness means that the design should be based on patterns that are easy to implement.                                                         |
| Separation of Concerns                      | Separate all concerns into their own views so that stakeholders can focus on a few things at a time. This approach helps manage high complexity solutions.                                                                                                                                                       |
**Key Issues in Software Design**
- Performance, security, reliability, usability, etc. are all major concerns when designing software
- Another important issue is how to decompose, organize, and package components
- **Aspects** are defines as other issues that deal with some aspect of a software's behavior that is not in the application domain, but addresses some of the supporting domains

| Software Design Issue                        | Definition                                                                                                                                                                                                                                                                   |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Concurrency                                  | Designing for concurrency places focus on decomposing software into processes, tasks, and threads to deal with efficiency, atomicity, synchronization, and scheduling.                                                                                                       |
| Control and Handling of Events               | How data is organized, how the control flow is structures, and how to handle reactive / temporal events through various mechanisms such as implicit invocation and call-backs.                                                                                               |
| Data Persistence                             | How long-lived data is handled                                                                                                                                                                                                                                               |
| Distribution of Components                   | How to distribute software across the hardware (including both computer and network hardware), how the components communicate, and how middleware can be used to deal with heterogeneous software.                                                                           |
| Error/Exception Handling and Fault Tolerance | How to prevent, tolerate, and process errors, as well as how to deal with exceptional conditions.                                                                                                                                                                            |
| Interaction and Presentation                 | Concerned wtih how to structure and organize interactions with users, as well as how to present information. (Model-View-Controller approach is a good example of this). This topic does not specify user interface details, which is the task of the user interface design. |
| Security                                     | How to prevent unauthorized disclosure, creation, change, deletion, or denial of access to information and other resources. How to tolerate attacks, limiting damage, speed of repair and recovery, etc. Also includes access control and cryptology.                        |
**Software Structure and Architecture**
- **Views** are specific representations of a partial aspect of a software architecture (self-explanatory).
	- Logical View - Functional Requirements 
	- Process View - Concurrency Issues
	- Physical View - Distribution Issues
	- Development View - Implementation Units and Dependendencies
- Architectural Styles are specializations of elements and relation types, together with a set of constraints on how they can be used.
	- General Structures (Layers, Pipes and Filters, Blackboard)
	- Distributed Systems (Client/Server, Three-Tiers, Broker)
	- Interactive Systems (Model-View-Control, Presentation-Abstraction-Control)
	- Adaptable Systems (Microkernel, Reflection)
	- Others (Batch, Interpreters, Process Control, Rule-Based, etc.)
- Design Patterns
	- Common solutions to common problems in a given context
	- Architectural styles describe high-level organization of software, while design patterns describe details at a lower level.
		- Creational Patterns (Builders, Factories, Prototypes, Singletons)
		- Structural Patterns (Adapter, Bridge, Composite, Decorator, Facade, Fly-Weight, Proxy)
		- Behavioral Patterns (Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template, Visitor)
- Families of Programs and Frameworks
	- One approach to providing reuse of software designs it to design **families of programs**. These are also known as **product lines**.
	- This can be done by identifying the commonalities among members of such families and by designing reusable and customizable components to account for the variability among family members.

**User Interface Design**
- Ensures that all interaction between the human and the machine provides for effect operation and control
- Software should be designed to match the skills, experience, and expectations of its anticipated users

| User Interface Design Principle | Definition                                                                                                     |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Learnability                    | Software should be easy to learn                                                                               |
| User Familiarity                | The interface should use terms and concepts drawn from the experiences of the people who will use the software |
| Consistency                     | The interface should be consistent so that comparable operations are activated in the same way                 |
| Minimal Surprise                | The behavior of the software should not surprise users                                                         |
| Recoverability                  | The interface should provide mechanisms allowing users to recover from errors                                  |
| User Guidance                   | The interface should give meaningful feedback when errors occur and provide context-related help to users      |
| User Diversity                  | The interface should provide appropriate interaction mechanisms for diverse types of users (508 Compliance)    |
- User interface design should solve two key issues:
	- How should the user interact with the software
	- How should information from the software be presented to the user
- **User Interaction Styles** can be classified into the following styles:
	- **Question / Answer** - A single question / answer exchange between the user and the software. 
	- **Direct Manipulation** - User interaction with objects on the screen, such as clicking
	- **Menu Selection** - Selecting a command from a menu list of commands
	- **Form Fill-In** - The user fills in fields of a form, supplying information to the software
	- **Command Language** - The user issues a command and provides related parameters to direct the software what to do
	- **Natural Language** - The user issues a command in natural language. This natural language is then parsed and translated into software commands.
- **Design of Information Presentation**
	- A good design keeps the information presentation separate from the information itself
	- The [MVC (Model-View-Controller)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) approach is an effective way to keep information presentation separate from the information being presented
	- Response time should also be considered (the time between when a user executes a command and the program responds)
	- Abstract visualizations can be used when large amounts of information are to be presented
	- Designers can use color to enhance the interface with the following guidelines:
		- Limit the number of colors used
		- Use color change to show the change of software status
		- Use color coding to support the user's task
		- Use color coding in a thoughtful and consistent way
		- Use colors to facilitate access for people with color blindness or color deficiency (508 compliance)
		- Don't depend on color alone to convey important information
- **User Interface Design is an Iterative Process** consisting of the following steps:
	- **User Analysis** - The designer analyzes the user's tasks, the working environment, other software, and how users interact with other people
	- **Software Prototyping** - Developing prototype software to help users to guide the evolution of the interface
	- **Interface Evaluation** - Designers observe user experiences with the evolving interface
- **Localization and Internationalization**
	- User interface design needs to consider internationalization and localization
	- It should adapt to different languages, regional differences, and technical requirements of the target market as needed
- **Metaphors and Conceptual Models**
	- User interface designers can use metaphors and conceptual models to set up mappings between software and reference systems that are already known to the user
		- For example, the operation 'Delete File' can be made into a metaphor by using the icon of a trash can

**Software Design Quality Analysis and Evaluation**
- **Quality Attributes** - Various attributes contribute to the quality of a software design such as maintainability, portability, testability, usability, correctness, robustness, etc.
- Some quality attributes are only discernible at runtime, such as performance, security, availability, functionality, and usability. This is opposite from attributes that are not discernible at runtime such as modifiability, portability, reusability, and testability.
- Furthermore, there are quality attributes related to the architecture's intrinsic qualities like conceptual integrity, correctness, and completeness.
- 