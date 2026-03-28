# **DeadEx: Dungeon Delivery**
**Game Design Document**  
Group Name: The Thought Cabinet 
Sean, Foster, Marcel, Henrique, Louie
Confidential
Date: 2/16/13
## Abstract
**DeadEx** is a top-down, puzzle-action game where logistics meets necromancy. Players control an undead courier tasked with delivering packages across dangerous dungeons filled with gimmicks. The core twist: the package is a physical object that must be protected, solved, and occasionally weaponized.
## Game Overview
**Genre:** Action-Puzzle  
**Core Conflict:** Deliver packages through dungeons while fending off the rival *United Sorcerers Parcel Service (USPS)*.
## Core Mechanics
### The Package System
The package can have different attributes per level:
- **Package Durability:** Some packages have durability. Dropping it, getting hit while holding it, or throwing it reduces its HP.  
- **Offensive Delivery:** You can throw the package at enemies to stun them.  
- **Tradeoff:** Using the package as a weapon risks destroying your payout.  
- **Interactivity:** Moving **Ice Cream** (melts in heat) and **Hot Stew** (cools in cold) through a dungeon with lava and freezer zones.
### Logistical Puzzles
Levels are designed around the specific properties of the cargo:
- **Multi-Package Routing:** The player may carry multiple items with conflicting needs.  
- **Weight Mechanics:** Heavy packages cannot be carried over jumps; they must be placed in minecarts or pneumatic chutes to traverse rooms, forcing the player to separate from the package to retrieve it on the other side. Light packages can be thrown or used as distractions.
## The World & Characters
### The Hub: The Necropolis Post Office
A safe zone similar to *Enter the Gungeon’s* Breach:
- **The Postmaster:** A Charon-like Lich who assigns deliveries.  
- **The Shop:** Spend Stamps to buy equipment or cosmetics.  
- **The Sorting Floor:** Where players select levels (Contracts).
### Enemies
1. **Porch Pirate Goblins:** They do not want to kill the player; they want to *steal the package*. If they grab it, they run toward a goblin hole. The player must chase them down.  
2. **USPS Wizards:** Rival sorcerers who use standard projectile magic to destroy your cargo or otherwise wreak havoc.
## Level Design Concepts
The goal is to reach the end of the stage. No mid-level saves; runs are short (5-10 minutes).
### Level 1: The North Pole (Client: Santa Claus)
**Hazard:** Ice Physics (Inertial Movement)  
**Mechanic:** The player slides until they hit a wall. The package adds momentum. Combat on ice requires predicting sliding trajectories.
### Level 2: The Industrial Zone (Client: Mad Scientist)
**Hazard:** Electro-Magnetism  
**Mechanic:** The package randomly switches polarity.
- **Attract Mode:** Metal debris and enemies are pulled toward the package (shielding the player, but damaging the box).  
- **Repel Mode:** Metal enemies are pushed away, but the package might fly out of your hands if you pass a metal pillar.
## Economy & Progression
### Currency: Stamps
Stamps are found by:
- Completing deliveries with high Package HP  
- Smashing breakables (crates, vases) in the dungeon  
- **Side Quests:** Finding lost letters or office supplies (staples, tape) requested by coworkers in the Hub
### Upgrades
Players spend Stamps to improve their courier abilities:

| **Upgrade**         | **Effect**                                                                   |
| ------------------- | ---------------------------------------------------------------------------- |
| Spectral Recall     | Reduces cooldown on pulling the package back after a throw                   |
| Grip Strength       | Increases movement speed while carrying heavy loads                          |
| Void Pockets        | Adds inventory slots to carry multiple packages simultaneously               |
| Bubble Wrap         | Provides a 1-hit shield for the package per room                             |
## Work Plan
The team follows a **Domain Ownership** model to maximize accountability and minimize technical conflicts. Each member is the primary architect for their specific area, though they collaborate across systems as needed.
### **Development Pillars**
- **Player Systems:** Movement, abilities, and controls.
- **Enemy AI:** Combat logic, pathfinding, and behaviors.
- **Level Design:** World building and environment assembly.
- **UX & Polish:** UI, HUD, and "game feel."
- **Audio:** Music and sound effects.
- **Engineering:** Project architecture and performance.
### **Workflow Rules**
- **Single-Scene Ownership:** To avoid version control errors, only one person may work in a specific scene at a time.
- **Cross-Functional Syncs:** Frequent communication ensures that "owned" mechanics integrate seamlessly across the project.