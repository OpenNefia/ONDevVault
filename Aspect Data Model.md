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
2. Granting a specific object special powers, but only for that specific object instance. Something like allowing an object to be counted as valuable for house/museum points.

I have to wonder if 1) is really necessary. All that would be needed is to store a pointer to the [[Defs|Def]] and the aspect type on the object, then instantiate it at runtime. But it sounds like that's using aspects outside of their intended purpose. And at a higher level, is creating such a puddle in the most general way possible even necessary in the first place? If all that matters is potion effects, then only potion effects should be a concern for that specific feat.

I can't think of any good examples for 2). That doesn't mean there are none, but I'm not really inventive enough to come up with them.

So maybe this runtime aspects feature is bunk and will only significantly increase complexity. Consider the deficiencies:

1. Dynamic aspects means that `AspectProperties` now requires game data serialization as well as `Def` serialization. This feels strange to me.
2. The list of aspects defined on the object is unlikely to change at runtime, so storing all the aspect data per-object is wasteful.
3. The size of the save data will significantly (?) increase.

Reducing save data size and keeping things simple is going to lean heavily on using pointers to Defs at every possible turn.

How I might implement the potion puddle:

1. Forget about aspects. The important thing is that the puddle has an effect definition and a power.
2. Also, forget about needing an aspect to trigger the "drinking" behavior. The entire point of effects is that they can be decoupled and triggered standalone.
3. Store a pointer to the `EffectDef` and the power on the puddle.
4. Run the effect when it's stepped on.

Defining `EffectDef` means we now have a stable, well-defined place to store serializable pointers to [[Effects]] without having to worry about aspects when dealing with serialization, because *aspects do not play well with serialization*, and *aspects are not dynamic, they are static*.

Also, I am not liking the inheritance approach for feats. I want some kind of component for triggering events when objects are stepped on instead.

When defining an aspect, the important thing is to separate non-serialized, static definition data from the dynamic, instanced data. In the potion puddle's case, the pointer to the `EffectDef` and power are *instance data*. When defining the puddle feat in XML, the effect and power might be specifiable inside `AspectProperties`, e.g. *definition data*, but those would be *optional* and merely copied to the instance on creation, since the puddle expects dynamic arguments for those properties.

How I might implement corpses:

1. Corpses have a `CorpseAspect` that handles the eating behavior.
2. They also have an `ItemFromChara` aspect holding the `CharaDef` pointer. This is useful for providing a generic interface for grabbing what character an item was harvested from, for corpses, remains, figures, cards, etc.

The problem is: what if someone wants to treat the static data as dynamic data for the purpose of modding? As in, a corpse that acts like a vegetable instead of meat? [[OpenNefia-LÖVE]] allows this by treating everything as dynamic and blatantly duplicating all the definition data.

I thought we decided that wasn't a good idea.