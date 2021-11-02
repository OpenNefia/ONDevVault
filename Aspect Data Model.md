How should [[Aspects]] be serialized? How will it be possible to clone an Aspect's data when splitting an item stack?

There seem to be two approaches: 

1. When cloning the object, create an entirely new object using the parent object's definition, and set the amount on the new object.
2. Deep copy all the fields.

As far as component architecture is concerned, RimWorld appears to use the first option. For corpse items, the engine will generate a single corpse item variant programmatically for every single race that is defined. This makes it significantly harder to change the behavior of a single corpse, but makes it significantly easier to serialize the corpse data, since there is nothing more to serialize except a pointer to the generated definition's name.

It is also not possible to clone components in RimWorld. "Cloning" a component must always occur by 

[[OpenNefia-LÖVE]] takes the second approach. Aspects added to a game object store all their data on the object directly, and static definitions are not involved.