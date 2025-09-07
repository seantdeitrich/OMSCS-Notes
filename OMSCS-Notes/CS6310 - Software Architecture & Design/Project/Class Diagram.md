```mermaid
classDiagram
	note "Group 6, CS6310, Fall2025, UML2.0"
	Pokemon o-- Move

	class PokemonFactory{
		- Pokemon createPokemon(String name)
	}
	
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
		- void rest()
		- void initialize()
	}
	
	class Application{
		- Scanner scanner

	}
	
	class Thunderdome{
		- initiateBattle(Pokemon pokemon1, Pokemon pokemon2)
	}
	
	class Command{
		<<Interface>>
		- getName()
		- execute(String[] arguments)
	}
	
	class BattleCommand{
		- Pokemon pokemon1
		- Pokemon pokemon2
		- displayStats()
	}
	
	class PokemonBank{
		- HashMap~Name,Pokemon~ bank
	}
	
	class CommandMap{
		- HashMap~Name,Command~ commandMap
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