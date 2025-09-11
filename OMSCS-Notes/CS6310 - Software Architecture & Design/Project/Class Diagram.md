```mermaid
classDiagram
	note "Group 6, CS6310, Fall2025, UML2.0"
	Pokemon o-- Move
	PokemonBank o-- Pokemon
	CommandMap o-- Command
	CommandMap <.. Thunderdome
	Command <.. Thunderdome : executes
	PokemonBank <.. Battle
	Battle ..|> Command
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
		- boolean isBattling
		- boolean isDefending
		- List~Move~ moveList
		- void attack() 
		- void defend()
		- decideAction()
		+ void processOpponentAttack(Move move)
		+ void takeTurn()
		+ void rest() // Restores pokemon to full hp
		+ void initialize()
	}
	
	class PokemonBank{
		- static List~Pokemon~ bank
		- static addPokemon(Pokemon pokemon)
		- static deletePokemon(Pokemon pokemon)
	}
	
	class PokemonFactory{
		- static Pokemon createPokemon(String name)
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
		- static HashMap~String name,Command command~ commandMap
	}
	


```
- Does PokemonFactory need a relationship to Pokemon?
	- Add comment about reflection implementation
- Pokemon affects Pokemon relationship?
- Automatically add pokemon to the bank after it's been created (relationship between bank and factory)
- Where/when does seed get set?
	- Battle class in thunderdome, setseed method for battle
 