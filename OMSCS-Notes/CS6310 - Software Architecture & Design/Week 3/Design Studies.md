# Design Studies
A **Design Study** is a rigorous and systemic evaluation of the factors that influence a design. It includes relevant criteria, metrics, and thresholds. It should compare various possible approaches.
- Key considerations in a design study are:
	- Performance
	- Memory Footprint
	- Time to Construct
- The overall goal of a design study is repeatability
- Design studies are presented through reports with charts, descriptions, etc.

## Design Study Structure
1. Context
	- Provides background and motivation for the study.
2. Research Questions
	- Examines the tradeoffs between various non-functional requirements
	- For example, space / time. 
	- Tradeoffs should be phrased as questions
	- Each question should be neutral and unbiased
3. Subjects
	- A subject is an instance of a design
	- This section should compare each subject against each other
4. Experimental Conditions
	- This section describes the environment in which the study will take place, since software design studies usually require running several versions of a program, making measurements, and evaluating the results.
5. Variables
	- Independent and dependent variables should be identified and defined
	- Metrics can be defined here as well, with units of measurements and how the research questions address them.
	- Should include a summary table with three columns:
		- Research questions, independent variables, dependent variables
6. Method
	- Describes the number of trials, measurement devices, tools, randomization technique, significant digits, etc.
	- Should include an explicit statement of which subjects will be run and the arguments used.
	- Describe statistical techniques as well.
7. Results
	- Presents the data collected and statistical analysis.
8. Discussion
	- Interpretation of the data and discuss the implications / explanations.
	- Reflect on the experiment itself
9. Conclusions
	- Summary of results and conclusions
	- Provide explicit answers to all research questions.

Each project has three deliverables:
1. Source code that solves a specific problem in several ways
2. A project report containing project-specific content
3. A design study

