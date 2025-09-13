```mermaid
--- 
title: CS6310 Group 6 Fall 2025 Thunderdome Project
---
classDiagram
    PokemonBank <.. Thunderdome
    PokemonFactory <.. Thunderdome
    PokemonBank o-- Pokemon
    Pokemon o-- Move
    Thunderdome "1" o-- "1" BattleContext
    Thunderdome --> "*" Battle
    Battle -- "2" Pokemon
    Battle --> BattleContext

    class Move {
        <<Abstract>>
        - boolean isDefensive
        - String name
        - int power
    }

    class Pokemon {
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
        - attack()
        - defend()
        - decideAction()
        + processOpponentAttack(Move move)
        + takeTurn()
        + rest() // Restores pokemon to full hp
        + initialize()
    }

    class PokemonBank {
        - static List~Pokemon~ bank
        - static addPokemon(Pokemon pokemon)
        - static deletePokemon(Pokemon pokemon)
        - static Pokemon getPokemon(String pokemonName)
    }

    class PokemonFactory {
        - static Pokemon createPokemon(String pokemonName)
        - static Move createMove(String moveName)
    }

    class Thunderdome {
        - Scanner scanner
        - BattleContext context
        - start()
        - processInput(input)
        - battle(String pokemonName, String otherPokemonName)
        - setSeed(int seed)
        - removeSeed()
        - help()
        - exit()
    }

    class BattleContext {
        - int seed
    }

    class Battle {
        - BattleContext context
        - Pokemon pokemon1
        - Pokemon pokemon2
        - initializeBattle()
        - startBattle()
    }

```
- Does PokemonFactory need a relationship to Pokemon?
	- Add comment about reflection implementation
- Pokemon affects Pokemon relationship?
- Automatically add pokemon to the bank after it's been created (relationship between bank and factory)

- Thunderdome has one battlecontext
- Thunderdome can create any amount of battles
- Each battle has 2 pokemon
- 
