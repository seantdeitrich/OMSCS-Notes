# Additional Requirements Report 
For this phase of the Pokémon Project, we are adding Persistence and Evolution as our functional features, Authorization/Authentication as the non-functional requirement. Group 6 has strong experience with RESTful APIs, particularly with the Postgres/Spring/Angular tech stack. These technologies will help us implement the features with consistency and ease. 
# Functional Feature: Persistence 
To support persistent data, we’re using a PostgreSQL database with the following core tables (at least): 
- Account/Trainer
- Skill
- Pokemon
- Species
- Battle
- Tournament

This setup will let us store and retrieve information for trainers, Pokémon, and battles across sessions. A docker volume will allow for the database to persist across application run instances. We chose PostgreSQL because it’s reliable, integrates easily with Spring Boot through JPA/Hibernate, and we’ve already had success with it in other projects. Design Notes: 

**Design Notes:**
- Each trainer owns multiple Pokémon, and each Pokémon references its corresponding species. 
- Battle and Tournament tables will record match results, which we can use for experience and evolution tracking. 
- Cloud databases weren’t considered because of class restrictions. 
- Other databases such as Oracle, MySQL, etc. were considered but ruled out due to the experience of the team.
# Functional Feature: Pokémon Evolution 
For evolution, we’re splitting Pokémon and Species into separate tables. 
- Pokémon instances will have foreign key to their current species
- Each species may have a foreign key to the next evolutionary form. If it is null, that Pokémon does not evolve.
- Pokémon will evolve after a set amount of battle wins
- Pokémon stats will be slightly boosted when they evolve, allowing them to be stronger in battle
- Evolution will trigger in the 'cleanup' phase of a battle, so a Pokémon may evolve mid tournament. 
# Non-Functional Feature: Authorization and Authentication 
For security, we’re introducing user accounts to manage trainer data and enforce access control. We’ll add an Account/Trainer table to the database to store login credentials and associate accounts with their Pokémon. Additionally we are introducing an Admin role that is able to create new Pokémon species through the graphical user interface. 

Key requirements: 
- Users will be able to login to the application and view their own Pokémon
- Each account can only edit their own Pokémon
- Admins are able to create new species, and can set evolutions as well. 
	- Note that the topmost evolution of a species must be created first to properly set this up.
	- For example, Charizard would be created first, then Charmeleon, then Charmander. This approach allows Charmeleon's evolution foreign key to be set to the already existing Charizard species.
- Once a new species is created, users can instantiate those species as level 1 Pokémon for their own account.

This ensures a basic level of protection for user data and keeps player interactions isolated within their own accounts. It also allows us to track and persist statistics for each account. 