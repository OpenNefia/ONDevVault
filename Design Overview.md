OpenNefia.NET is currently being designed around a set of data structures, each with their own specific purpose.

However, of course, nothing is final.

There are six fundamental "building blocks" in OpenNefia.NET's ecosystem. starting from least to most granular: Worlds, Areas, Maps, Map Objects, Aspects, and Effects.

A World holds a complete saved game.

An Area contains a collection of maps. Each map is given a numeric floor number.

A Map contains a set of Map Objects. It also contains tile/memory/visibility data.

A [[Map Objects|Map Object]] is a game object that can be displayed on a map. They include characters, items, map features (feats) and map effects (mefs). Each map aspect can have zero or more Aspects attached to it.

An [[Aspects|Aspect]] is a piece of data that is parented to an active Map Object. Each Aspect can implement one or more interfaces related to manipulating Map Objects, like controlling how items are stacked, or what happens when a character steps on a trap.

An [[Effects|Effect]] 

The difference between Aspects and Effects is that Effects do not need an active Map Object parent, and Effects should not act as owners of Aspects or Map Objects. Effects are more like first-class functions with additional parameters that can be tweaked, and a single `Apply()` method that acts like calling the function.

