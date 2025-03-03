# actors proposal
Many datapacks have the common pattern of having an entity or many entities in the world they want to select/execute via later, this provides a way to efficiently access them with comparable performance to hard coded UUIDs. Your next custom entity or summoned mount is an “actor” in your “stage play” of a datapack. Additionally, those familiar with actual game engines may be familiar with “actor” being used to describe game objects with custom behavior, which is also applicable here. Motivations/Background for this feature are detailed at the bottom of this doc.

## Syntax
```mcfunction
#
# savePersist (boolean): Defaults to true, controls whether the actor group will be saved to a namespace file (similarly to storage).
#
# unload: behavior when an entity in the group is unloaded.
# - remove; entity is removed from the group.
# - ignore; entity is kept in the group, load/unload hooks are disabled.
# - separate (default); entity is separated from the main group to prevent indexing attempts, upon load the entity is restored to the main group.
#
# dyingGroup: Defaults to none, actor group to place entities in when they are dying (not yet dead, but otherwise unselectable).
#
actors group add <namespaced id> [<savePersist>] [<unload: remove|ignore|separate>] [<dyingGroup: namespaced ID | none>]


#
# Copies an existing actor group to a new one, depending on unload some entities may be lost.
#
actors group from <source id> <new id> [<savePersist>] [<unload: remove|ignore|separate>] [<dyingGroup: namespaced ID | none>]


#
# Responds with a list of entity chat components, group settings, count of currently active entities (result), and, if applicable, count of tracked unloaded entities.
#
actors group get <namespaced id>


actors group remove <namespaced id>


actors entity add <MultipleEntityArgument> <namespaced id>


#
# In order to remove unloaded entities when applicable, use of a static (or macro) UUID is supported.
#
actors entity remove <MultipleEntityArgument> <namespaced id>
actors entity remove * <namespaced id>


#
# A selector acting as the group, will use the provided entity reference map. Distance arguments no longer define how entities are found; chunk data isn’t accessed.
#
execute as @e[actors=<actors group id>]


#
# Lighter method of acting as the group, additionally allowing for a dynamic id without use of macros.
#

execute via <actors group id>|<data storage|entity|block <target> <path.To.Actors.Group.ID>>
```

## Implementation
Actors groups are a datapack-accessible manager of custom entity reference maps, allowing for very optimized selection of groups of entities, similarly to how `@a` limits the entity list to just players without any filtering required. To put it in perspective, it's like each actor group is your own custom selector, where instead of getting all living entities in the world, you get your list, which is inherently better for performance due to a much shorter iteration time.

todo: more technical explanation, see [EfficientTeams](https://github.com/Slackow/EfficientTeams) for an example

## Motivations & Background
This proposal is specifically for Minecraft: Java Edition and the performance problems that have plagued datapack creators forever.

One of the greatest scourges to datapack performance is the widespread usage of a common syntax pattern, `execute as @e[tag=example]`. This is largely due to the fact that when this evaluates, every single entity in the world is iterated through and checked for this tag. 

Why not cache all entities that have any given tag, allowing for faster selectors? 
1. Most tags are not exclusively used to select entities out of the entire world, most come with an optimized distance-limited syntax or are used to manage flags/state on an entity you already have selected from the world. 
2. Storing entity lists for all of these tags would be a poor performance decision
3. Mojang already implicitly decided to not do this when implementing tags; they optimized other selector arguments, but not this one, and if they were going to optimize it they probably would’ve done so at this point. Additionally, many discussions on the poor performance of global entity tag checks have been conducted with mojang devs lurking in MCC, to no avail.

Why not use the `tick` enchantment trigger along with the `run_function` entity effect?
1. Doesn’t allow for multi-context management; eg. `ride @s mount @e[actors=example:mount,limit=1]`
2. Every function call has a cost; when executing as many entities, it's more efficient to do `execute as @e[tag=example] at @s if block ~ ~-1 ~ dirt run particle angry_villager ~ ~1 ~ 0 0 0 0 1` than to run the particle command in a function as each entity, and this even applies if you’re running several commands at once

Why not cache all entities that are in a team? (example here: https://github.com/Slackow/EfficientTeams)
1. Entities can only be on one team at a time, which would arbitrarily limit complex/interconnected system designs.
2. Similar performance concerns to tags; not every usage of teams needs this.

Another consideration: single-entity actor groups are just as efficient as a hardcoded UUID, without being hardcoded or forced to use a less efficient macro.
