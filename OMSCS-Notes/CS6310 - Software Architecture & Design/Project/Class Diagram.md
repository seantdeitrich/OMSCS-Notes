```mermaid
classDiagram
	note "Group 6, CS6310, Fall2025, UML2.0"
	Pokemon o-- Move
	PokemonBank o-- Pokemon
	CommandMap o-- Command
	CommandMap <.. Thunderdome
	Command <.. Thunderdome : executes
	PokemonBank <.. Battle
	Command <|-- Battle
	PokemonFactory <.. Battle

	class Move{
		<<Abstract>>
		- boolean isDefensive
		- String name
		- int power
	}
	
	class Pokemon{
		<<Abstract>>
		- int seed
		- String name
		- int maxHp
		- int currentHp
		- int numWins
		- int numLosses
		- boolean isDefending
		- List~Move~ moveList
		- void attack() 
		- void defend()
		- void processOpponentAttack(Move move)
		- void decideMove()
		- void rest() // Restores pokemon to full hp
		- void initialize()
	}
	
	class PokemonBank{
		- List~Pokemon~ bank
		- addPokemon(Pokemon pokemon)
		- deletePokemon(Pokemon pokemon)
	}
	
	class PokemonFactory{
		- Pokemon createPokemon(String name)
	}
		
	class Thunderdome{
		- Scanner scanner
	}
	
	class Command{
		<<Interface>>
		- getName()
		- execute(String[] arguments)
	}
	
	class Battle{
		- Pokemon pokemon1
		- Pokemon pokemon2
		- showMatchup()
	    - initializeBattle() // Create/Get pokemon via PokemonFactory
	}
	
		
	class CommandMap{
		- HashMap~String name,Command command~ commandMap
	}
	


```
- Battle Command is a Command (make relationship)
- Pokemon have a list of moves (aggregation?)
- Include command map?

- Before the battle starts
	- Create the two pokemon IF they don't exist in the storage
		- If they do exist, send the existing pokemon to the thunderdome
	- Move those two pokemon instances into a list or some sort of storage
		- (Not in thunderdome, storage class?)
	- Send those two to the thunderdome


- Thunderdome gets/uses/executes commands
- Connects the commands to the pokemon

Thunderdome accepts user input?
Can battles exist without 2 pokemon
 