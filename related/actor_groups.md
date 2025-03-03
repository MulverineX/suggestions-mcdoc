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