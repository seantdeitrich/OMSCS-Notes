title Le Pokemon Squence

actor User
participant thunderdome
participant battleContext


participant battle
participant pokemonBank
participant pokemonFactory
rparticipant pokemon;pokemon1
rparticipant pokemon;pokemon2

entryspacing 0.1
User->thunderdome:setseed, 10
activate thunderdome #lightgrey
thunderdome->battleContext:setseed(10)
activate battleContext #lightgrey
battleContext->battleContext:seed set to 10

thunderdome<--battleContext:return null
deactivate battleContext

User->thunderdome:battle, pokemon1, pokemon2
space 1
group battle(pokemon1, pokemon2)
thunderdome->pokemonBank:pokemon1 exists?
activate pokemonBank #lightgrey
pokemonBank->pokemonFactory:else; pokemon1 does not exist
activate pokemonFactory #lightgrey
pokemonFactory->pokemonFactory:<<creates pokemon>>
pokemonFactory-->pokemonBank:pokemon1 created
pokemonBank-->thunderdome:pokemon1 exists
deactivate pokemonFactory
thunderdome->pokemonBank:pokemon2 exists?
pokemonBank->pokemonBank:pokemon2 exists
pokemonBank-->thunderdome:pokemon2 exists
space 1
deactivate pokemonBank
note over thunderdome:both pokemon exist, init battle
thunderdome->battle:init battle
activate battle #lightgrey
group initBattle
activate pokemon;pokemon1 #lightgrey
activate pokemon;pokemon2 #lightgrey
battle->pokemon;pokemon1:pokemon1(seed=x)
battle->pokemon;pokemon2: pokemon1(seed=x+1)
space 1
note over battle:seeds set, start battle
space 1
end
space 1
group startBattle
battle->pokemon;pokemon1:startBattle()
pokemon;pokemon1->pokemon;pokemon1:takeTurn()
note left of pokemon;pokemon1:processOpponentMove()\ndecideAction()\naction [attack() or defend()]
pokemon;pokemon1-->(5)pokemon;pokemon2:attack()
space -6
pokemon;pokemon2->pokemon;pokemon2:takeTurn()
space 1
note right of pokemon;pokemon2:processOpponentMove(attack)\ndecideAction()\naction [attack() or defend()]
pokemon;pokemon2-->battle:health check
[
battle-->pokemon;pokemon1:continue
pokemon;pokemon2-->(5)pokemon;pokemon1:defend
space -6
pokemon;pokemon1->pokemon;pokemon1:takeTurn()
note left of pokemon;pokemon1:processOpponentMove()\ndecideAction()\naction [attack() or defend()]
pokemon;pokemon1-->(5)pokemon;pokemon2:attack()
space -6
pokemon;pokemon2->pokemon;pokemon2:takeTurn()
space 1
note right of pokemon;pokemon2:processOpponentMove(attack)\ndecideAction()\naction [attack() or defend()]
pokemon;pokemon2-->battle:health check
deactivate pokemon;pokemon1
deactivate pokemon;pokemon2
space 1

end
battle-->thunderdome:battle complete
deactivate battle
space 1
end
thunderdome-->User:

