**Pre-planned Behaviors**
- All contingencies / outcomes are anticipated and if something is missed the agent is unable to adapt
- Decisions are made by conditional logic
- Can include deterministic or non-deterministic conditions

**Types of Decision Making**
- Reactive - Respond to environmental state with pre planned conditional actions
	- Most game agents are reactive
- Deliberative - Models the discretized environment and infers the plan to achieve a goal state
- Reflective - Learn from experience

## Reactive Decision Making Patterns
- Examples include Decision Trees, Rule Based Systems, Finite State Machines, and Behavior Trees
### Decision Trees
- ![](../Images/Pasted%20image%2020260707190442.png)
- Decision trees are good for simple decision making but can quickly become difficult to manage
- Common for character and animation control
- Simple to implement, easy to understand and read, modular with ability to stick parts of decision trees together
- Training / Learning is possible but is not covered
- Decision trees are best kept simple with basic boolean checks
- Well balanced trees optimize the number of decisions that need to be made
- Usually best as a binary tree as opposed to an n-ary tree
- One weakness is that D-Trees are tightly coupled with game state. If the game state changes or the implementation of other aspects of the game change, the D-Tree might also need to be updated
- You can lazy load uncommon tree paths if needed to improve performance
- Randomness can be introduced intro decision trees, for example flipping a coin at a given node that decides between attacking and defending
	- However, D-Trees are evaluated often. Stickiness can be introduced to stick to a random choice until the game state changes or an optional time out is reached, which avoids flickering between the two states

### Rule Based Systems
- Define actions to be executed in response to conditions
- Very simple architecture but it is difficult to organize sometimes because there isn't always a straightforward way to order the rules
- Essentially just ordered if statements
- Can be difficult to read
- Advanced production rule systems use an **arbiter** that selects matching rules, for example we can evaluate the first that that matches from an ordered list
- Dynamic priority can also be introduced to change the priority of rules as game state changes
	- An `activatedRules` list can be used and then evaluated at the end to see which one has the highest priority
### Behavior Trees
- Increases flexibility from FSMs and have become very popular (and even replace FSMs)
- Includes Composite Nodes, Decorator Nodes, and Leaves
- Each node can be in Success, Failure, Running, or Uninitialized
- Has simple reactive planning through a tree of behaviors
- Transitions are externalized from states, instead what you normally think of as states in a FSM become behaviors:
	- ![](../Images/Pasted%20image%2020260707194036.png)
- **Composite Nodes** operate similar to boolean operations
	- Sequence composite nodes are AND nodes
	- Selector composite nodes are OR nodes
	- Random nodes also exist
	- Different from decision trees, these nodes may have multiple combinators
	- Each composite node returns success or fail based on children
	- Priority can be introduces in the order of combinators
		- Try opening the door before climbing the roof to break into a house in a sequence node
- **Decorator Nodes** only have one child and can control its child
	- Changes the return, terminates, controls the logic, etc.
	- Examples are Inverter Nodes (like NOT), Succeeders (always true, runs childs but doesn't care about success/failure), Repeater, Repeat until fail, etc.
- **Leaf Nodes** are behaviors that return success, fail, or processing
	- Includes game logic
- Behavior Trees have one action per tick
- ![](../Images/Pasted%20image%2020260707200312.png)
	- The top selector is choosing between opening the door and the window
	- This could be extended easily by adding another sequence (breaking in through the chimney)
	- The professor recommended putting a succeeder node between the sequence and close door nodes, since it doesn't necessarily matter whether the door actually closes
- **Non Deterministic** Behavior Trees introduce random selectors or sequence nodes
	- It can also introduce prioritized versions which select or sequence according to world state
- **Semaphores Nodes** check for restricted resources and keep a tally of available resources and number of users
- **Blackboard** are shared memory structures between nodes that share data between tasks
	- Can be as simple as shared variables that are assigned and read by various nodes
	- Dictionaries can also be used, and subtrees may have their own sub-blackboards

## Fuzzy Logic
- Embraces the vagueness of language in processes with **truth degrees** and subjectivity
- Allows for non-numeric expressions of rules and facts (e.g cautious vs. confident)
- An FSM might look unnatural when it switches between two states (between cautious / confident), but fuzzy logic can mix the two states essentially
- Relatively popular in the games industry, but largely dismissed in academic AI
- Uses **Degrees of Membership** that identifies how much something belongs to a certain state (usually a float between 0 and 1)
	- "It's a little hot / cold"
	- ![](../Images/Pasted%20image%2020260708182625.png)
- **Fuzzy Logic, Sets, and Fuzzy Rules** are **notably not using probability**
- ![](../Images/Pasted%20image%2020260708182734.png)
- If you were coding an AI golfer, you might set up rules and ranges like:
	- "When putting, if the ball is far from the hole and the green is sloping gently left to right, hit the ball firmly and at an angle slightly left of the flag"
	- If we then set up ranges for Close, Medium, and Far, problems can occur at the edges of those ranges with abrupt switches in power
- When there are multiple degrees of membership it makes sense that they sum to 1.0, but in practice it isn't necessary and a weighted average is often sufficient
### Fuzzy Logic Rules
- Start with **crisp**, discrete values, then apply **Fuzzification**
	- Fuzzification establishes fuzzy boundaries (partial membership) described with **Membership Functions**
	- **Fuzzy Rules** - "If ***antecedent*** then ***consequent***" 
		- The antecedent has a degree of membership (high, low, etc)
		- The consequent fires by degree
	- **Defuzzification** turns the degrees of membership and the consequents back to crisp values
- **Fuzzy Inference**
	- For each antecedent, calculate the degree of membership of the input data (crisp values)
	- Calculate the rules inferred conclusion based upon the values in the previous step
	- Then combine all the inferred conclusions into a single conclusion (**Fuzzy Set**)
	- For crisp values, the conclusion must be **defuzzified**
- **Fuzzy Rules**
	- We can relate the known membership of fuzzy sets to generate new DOM values for other fuzzy sets
		- Rules must be created for each possible combination of antecedent sets
		- "If I am close to the corner AND I am travelling fast, then I should break"
	- ![](../Images/Pasted%20image%2020260708190255.png)
- **Fuzzification**
	- Produces degrees of membership from game state and membership functions
	- ![](../Images/Pasted%20image%2020260708190459.png)
	- Min should be used for AND combinators, Max for OR
- **Defuzzification**
	- You can use the DOMs as weighted averages, or take a **Center of Gravity** approach by integrating under the curves and finding the center of that space:
	  ![](../Images/Pasted%20image%2020260708193038.png)
	- The center of gravity approach is expensive however, and blending typically works just as well
### Example
![](../Images/Pasted%20image%2020260708193147.png)

- ![](../Images/Pasted%20image%2020260708193631.png)
## Racecar Examples
![](../Images/Pasted%20image%2020260708194025.png)
![](../Images/Pasted%20image%2020260708194122.png)
