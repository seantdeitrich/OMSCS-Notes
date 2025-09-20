```mermaid
sequenceDiagram
    participant battle as Battle
    participant pokemon1 as Pokemon1
    participant pokemon2 as Pokemon2
    participant rand as Random
    activate battle
    loop pokemon1.getHealth()>0 && pokemon2.getHealth()>0
	    activate pokemon1
	    activate pokemon2

        alt pokemon2.queuedSkill != null && !pokemon2.queuedSkill.getIsDefensive()
            pokemon1->>pokemon2: getQueuedSkill()
            pokemon2-->>pokemon1: queuedSkill
            pokemon1->>pokemon1: processOpponentSkill(queuedSkill)
        else
            note right of pokemon1: No skill to process
        end
        battle->>pokemon1: takeTurn()
        pokemon1->>pokemon1: shouldAttack() : returns true
		pokemon1->>pokemon1: getRandomOffensiveSkill() : returns offensiveSkill
		pokemon1 ->> pokemon1: setQueuedSkill(offensiveSkill)

        alt pokemon1.queuedSkill != null && !pokemon1.queuedSkill.getIsDefensive()
            pokemon2->>pokemon1: getQueuedSkill()
            pokemon1-->>pokemon2: queuedSkill
            pokemon2->>pokemon2: processOpponentSkill(queuedSkill)
        else
            note right of pokemon2: No skill to process
        end
        battle->>pokemon2: takeTurn()
        pokemon2->>pokemon2: shouldAttack()
        pokemon2->>pokemon2: setQueuedSkill()
        deactivate pokemon1
        deactivate pokemon2
    end

    deactivate battle
```

**TODO:**
- Show basic implementation for processOpponentSkill
	- Apply the damage from the opponents move (that skills power)
	- If we're defending reduce the incoming damage
- Show implementation details for should attack, and add calls to random
	- Show getting the current health ratio
	- Show getting aggressiveness value
	- Show random generation and compare against aggressiveness 
	- return true / false
- Show implementation for getRandomOffensive / Defensive Skills (Choosing randomly from the list)
	- Show filtering list on the isDefensive boolean
	- Show random selection from the filtered list