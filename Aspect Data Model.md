How should [[Aspects]] be serialized? How will it be possible to clone an Aspect's data when splitting an item stack?

There seem to be two approaches: 

1. When cloning the object, create an entirely new object using the parent object's definition, and set the amount on the new object.
2. Deep copy all the fields.

As far as component architecture is concerned, RimWorld appears to use the first option. For corpse items, the engine will generate a single corpse item variant programmatically for every single race that is defined. This makes it significantly harder to change the behavior of a single corpse, but makes it significantly easier to serialize the corpse data, since there is nothing more to serialize except a pointer to the generated definition's name.

It is also not possible to clone components in RimWorld. "Cloning" a component must always occur by creating a fresh object and using the list of component properties defined in the object definition.

Perhaps the most significant limitation with RimWorld's components is that it is not possible to add new components to an object at runtime. Only the components in the definition's component list will ever be instantiated. At the same time, this means that there is no need to serialize the list of components for each object, because that list is always defined in terms of the object's definition, so only the pointer to that definition is necessary to save.

[[OpenNefia-LÖVE]] takes the second approach. Aspects added to a game object store all their data on the object directly, and static definitions are not involved. With this approach I was worried about bloating up the save data with a huge number of redundant aspect definitions (like having 200 stacks of potions in a map, all the same type, and with all the same spell effect/power data repeated for each one). At the same time, this allows you to dynamically store generate information like the character type on a corpse.

But in RimWorld, components still have instance data, and they can have stacking/splitting logic. It's just that you can't deep copy components directly, and you cannot add components at runtime.

So why would we need to add arbitrary components at runtime?

1. Dynamic object behaviors. Like a puddle that accepts any number of `ICanDrinkAspect` components and runs the `OnDrink()` callback for each.
2. Granting a specific object special powers, but only for that specific instance. Something like allowing an object to be counted as valuable for house/museum points.

I have to wonder if 1) is unnecessary. All that would be needed is to store a pointer to the [[Defs|Def]] and the aspect type on the object, then instantiate it at runtime. But it sounds like that's using aspects outside of their intended purpose. But at a higher level, is creating such a puddle even necessary in the first place? If all that matters is potion effects, then only potion effects should be a concern.

I can't think of any good examples for 2). That doesn't mean there are none, but I'm not really inventive enough to come up with them.

So maybe this runtime aspects feature is bunk and will only significantly increase complexity. Consider the downsides:

1. Dynamic aspects means that `AspectProperties` requires game data serialization, as well as `Def` serialization.
2. The list of aspects defined on the object is unlikely to change. What *is* likely to change is its instance data. However, that does not mean all of it is necessary to serialize.
3. Save data size will signifi