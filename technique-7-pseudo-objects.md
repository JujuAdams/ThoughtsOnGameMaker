# GameMaker Technique 7 - Pseudo-objects

&nbsp;

I've regularly run into situations where a full fat GameMaker instance does too much and is too "heavy". An example is the reactive grass in [The Swords Of Ditto](https://store.steampowered.com/app/619780/The_Swords_of_Ditto_Mormos_Curse/). The grass needs to react to the player's movement and the movement of enemies, it needs to animate gently in the wind, it needs to make a rustling sound as it moves, it needs to be depth-sorted (`depth = -y` my beloved), it needs to be able to be chopped down or burnt, and there is often up to a hundred grass instances on screen at a time. Grass instances in Swords Of Ditto were pretty complicated and they each had a finite state machine controlling them.

Running 100 instances used up too much processing power for The Swords Of Ditto. We were working on thin margins since there was so much going on in the game. After some thinking, I settled on a creative solution: **_Represent each grass instance as a row in a ds_grid and operate as much logic as possible through bulk ds_grid operations._** This is the heart of the pseudo-object concept. Additionally, we created sprite assets on asset layers to handle animation without us managing the process. This isn't necessarily a part of the pseudo-object mentality but it resulted in significant enough performance gains that it bears mentioning.

The rest of this page is going to cover what I did to improve grass performance. Pseudo-objects, however, have uses far beyond grass. [Scribble](https://github.com/JujuAdams/Scribble) uses pseudo-objects for text layout, for example. [Skies Of Chaos](https://www.youtube.com/watch?v=dSyWXQv3HOY&ab_channel=Netflix) uses pseudo-objects for player and enemy projectiles. If you have many instances of a (relatively) simple object and that's causing performance issues then using a ds_grid filled with pseudo-objects may be a viable solution.

Converting from instances of grass objects to pseudo-objects involved a few steps:

1. Grass needed to be stripped back to only its core variables. `x` `y` `blendColour` `burnTime` `asset` and so on. Each grass instance can then be represented as a row in a ds_grid with each column being a different variable.

2. Create a list of empty rows. When a grass instance needs to be created, pick a row from the list and fill in initial data. When a grass instance needs to be destroyed, add the leaving grass instance row index to the list of empty rows. For The Swords Of Ditto we didn't need a list of empty rows because we weren't creating grass after a room was loaded, but it's worth mentioning regardless.

3. When creating a grass instance, make a sprite asset on an asset layer and keep a reference to that sprite asset. Since we want to depth sort grass, each layer will need to be given an appropriate depth. If two grass instances exist at the same depth value we place them on the same layer. When grass is destroyed, remove the sprite asset and potentially the layer too. If an animation needs to be played, change the sprite on the sprite asset.

4. Create further lists, one for each state, that contains row indexes. At any one time, these lists would be small with only a handful of grass instances in them. When a grass instance changes state, move it from one list to another. An advanced implementation might conceivably use different grids for different states but that seemed overkill for grass.

5. Rebuild logic using as many `ds_grid_region_*` operations as possible. Timers can be incremented (or decremented) across all grass instances very easily.

6. If delta timing needs to be implemented (which was a lesser known feature that The Swords Of Ditto used on low power Android phones) then this can be applied to sprite asset animations by dynamically adjusting the image speed when the delta time factor changes.

You'll note I've not mentioned collision detection here. GameMaker's collision detection is good for general use but we needed some tailored to our use case. In this situation, and again because grass doesn't move, we could build a crude cell-based collision system to simplify collision detection. The entire room was split into small cells, and each cell contained a list of grass instances that overlapped with it. When we wanted to check for collisions around e.g. a fireball we checked what grass instances were in the cell underneath the fireball and could instantly find the list of grass instances that overlapped with it. The same logic was used for detecting grass that should be cut down by a swing of the player's sword.

There's a lot going on here. What allowed Swords Of Ditto to run well on low end hardware was a shift in perspective. Instead of seeing each grass instance as its own thing, instead I wrote code that treated grass instances as a set of repeated behaviours. Instead of each grass instance having its own state machine, instead each state machine had its own grass instances. Instead of checking for collisions every frame, we precalcuate collisions through a cell-based approximation. The same mentality can help in a lot of other places too.
