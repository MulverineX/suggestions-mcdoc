# custom `relation` proposal
Many datapacks have the common pattern of having an entity or many entities in the world they want to select/execute via later, this provides a way to efficiently access them with comparable performance to hard coded UUIDs. Motivations/Background for this feature are detailed at the bottom of this doc.

## Syntax
```mcfunction
# create a new private relation containing an entity/entities, or append new entities to an existing relation
relation <target> private add <relation> <targets>

# remove an entity or entities from this private relation
relation <target> private remove <relation> <targets>

# remove the entire private relation
relation <target> private remove <relation>

# remove unloaded/dead entity/entities from this private relation, if the resulting list is empty the relation will be removed
relation <target> private forget <relation>

# create a new relation group and join the targeted entity/entities to it
relation <targets> group join <relation>

# join the targeted entity/entities to a specific relation group, if the group doesn't exist yet it will be created
relation <targets> group join <relation> <uuid>

# join the group of another entity
relation <target> group join <relation> from <entity>

# leave the group
relation <target> group leave <relation>

# execute via the entity/entities contained in this relation from the context entity
execute on <relation> ...

# select entity/entities contained in this relation from the context entity
# cannot be negated nor be used multiple times within a selector
# made available for multi-context commands
execute as @e[relation=<relation>] ...
```

```rust
```

## Implementation
Relation references are represented in a new entity data component, `minecraft:relation`. In all locations where a relation id is being referenced the `minecraft:` namespace is optional/implicit like in other registries.

Private relations are stored on the entity as a list of entity UUID integer arrays.
Note: Some built-in private relations are "ephemeral"; they cannot be removed and do not get stored on the entity but can be used the same way (eg. `minecraft:passengers`).

Relation groups are stored on the entity as a single group reference UUID integer array.
They are an in-memory entity reference set managed by the game. Entities that are loaded/created/modified that have a corresponding `relation` component are added to a corresponding reference set indexed by the reference UUID.

## Motivations & Background
This proposal is specifically for Minecraft: Java Edition and the performance problems that have plagued datapack creators forever.

One of the greatest scourges to datapack performance is the widespread usage of a common syntax pattern, `execute as @e[tag=example]`. This is largely due to the fact that when this evaluates, every single entity in the world is iterated through and checked for this tag. 

Why not cache all entities that have any given tag, allowing for faster selectors? 
1. Most tags are not exclusively used to select entities out of the entire world, most come with an optimized distance-limited syntax or are used to manage flags/state on an entity you already have selected from the world. 
2. Storing entity lists for all of these tags would be a poor performance decision
3. Mojang already implicitly decided to not do this when implementing tags; they optimized other selector arguments, but not this one, and if they were going to optimize it they probably would’ve done so at this point. Additionally, many discussions on the poor performance of global entity tag checks have been conducted with mojang devs lurking in MCC, to no avail.

Why not use the `tick` enchantment trigger along with the `run_function` entity effect?
1. Doesn’t allow for multi-context management; eg. `ride @s mount @e[relation=example:mount,limit=1]`
2. Every function call has a cost; when executing as many entities, it's more efficient to do `execute as @e[tag=example] at @s if block ~ ~-1 ~ dirt run particle angry_villager ~ ~1 ~ 0 0 0 0 1` than to run the particle command in a function as each entity, and this even applies if you’re running several commands at once

Why not cache all entities that are in a team? (example here: https://github.com/Slackow/EfficientTeams)
1. Entities can only be on one team at a time, which would arbitrarily limit complex/interconnected system designs.
2. Similar performance concerns to tags; not every usage of teams needs this.
3. Only LivingEntity extended mobs can be joined to a team.

Another consideration: single-entity relations are just as efficient as a hardcoded UUID, without being hardcoded or forced to use a less efficient macro.
