## Additional Requirements Report

For this phase of the Pokémon Project, we are adding Persistence and Evolution as our functional features, Authorization/Authentication as the non-functional requirement. Group 6 has strong experience with RESTful APIs, particularly with the Postgres/Spring/Angular tech stack. These technologies will help us implement the features with consistency and ease. 
### Functional Feature: Persistence
To support persistent data, we’re using a PostgreSQL database with the following core tables (at least):
- Account/Trainer
- Skill
- Pokémon
- Species
- Battle
- Tournament
  
This setup will let us store and retrieve information for trainers, Pokémon, and battles across sessions.  We chose PostgreSQL because it’s reliable, integrates easily with Spring Boot through JPA/Hibernate, and we’ve already had success with it in other projects.

**Design Notes:**
- Each trainer owns multiple Pokémon, and each Pokémon references its corresponding species.
- Battle and Tournament tables will record match results, which we can later use for experience and evolution tracking.
- Cloud databases weren’t considered because of class restrictions.
- Other databases such as Oracle, MySQL, etc. were considered but ruled out due to the experience of the team.
### Functional Feature: Pokémon Evolution
For evolution, we’re splitting Pokémon and Species into separate tables.
- Species defines base stats, potential evolutions, and evolution conditions.
- Pokémon stores the individual instance data, including experience, level, and ownership.

We plan to trigger evolution automatically after a battle, either based on number of wins or experience gained.  This could be implemented with a database trigger or as part of an after-battle cleanup process in the backend.

**Pros and Cons:**
- A database trigger is simple to set up and ensures evolution happens automatically.
- However, triggers don’t easily handle more complex evolution cases (e.g., branching evolutions like Eevee) or situations where the player wants to cancel evolution.

To handle that limitation, we could add a `preventEvolution` boolean flag in the Pokémon table. This would let players opt out of evolving when conditions are met, keeping flexibility in the system.
### Non-Functional Feature: Authorization and Authentication
For security, we’re introducing user accounts to manage trainer data and enforce access control.  
We’ll add an Account/Trainer table to the database to store login credentials and associate accounts with their Pokémon.

Key requirements:
- Each account can only edit or create their own Pokémon.
- A login page will be added to the front end.
- Spring's authorization annotations should assist with implementation.

This ensures a basic level of protection for user data and keeps player interactions isolated within their own accounts. It also allows us to track and persist statistics for each account.
