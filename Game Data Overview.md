OpenNefia.NET is currently being designed around a set of data models, each with their own specific purpose.

However, of course, nothing is final, and everything is still subject to change. The final design will depend on how easy it becomes to mod the game, as well as the potential headaches that arise from trying to use the engine and API on a day-to-day basis.

## Overarching Concepts

There are six fundamental "building blocks" in OpenNefia.NET's ecosystem, starting from least to most granular: Worlds, Areas, Maps, Map Objects, Aspects, and Effects.

A World holds a complete saved game. It holds a list of all generated Areas, as well as the extra, global saved data for mods that doesn't belong in a Map.

An [[Areas|Area]] contains a collection of maps. Each map is given a numeric floor number.

A [[Maps|Map]] contains a set of Map Objects. It also contains tile/memory/visibility data.

A [[Map Objects|Map Object]] is a game object that can be displayed on a map. They include characters, items, map features (feats) and map effects (mefs). Each map aspect can have zero or more Aspects attached to it.

An [[Aspects|Aspect]] is a piece of data that is parented to an active Map Object. Each Aspect can implement one or more interfaces related to manipulating Map Objects, like controlling how items are stacked, or what happens when a character steps on a trap, by overriding the virtual methods on them. Aspects typically encapsulate Map Objects or references to them. They are also used to encapsulate Effects for implementing things like magical items.

An [[Effects|Effect]] is a piece of reusable logic that can be run on a {character/Map Object}(?). It is not attached to any active game state and can be serialized and deserialized directly. Effects can be passed parameters inside their definitions to change their behavior in a declarative way. An Effect could be used as an item use action, the logic run when an elemental attack damages a tile on the map, damage triggered by opening a door, and so on.

The difference between Aspects and Effects is that Effects do not need an active Map Object parent, and Effects should not act as owners of Aspects or Map Objects. Effects can be defined inside [[Defs]] in a standalone fashion, and then called like a function without any extra instantiation. Effects are more like serializable first-class functions with additional parameters that can be tweaked, and a single `Apply()` method that acts like calling the function.

The intention with this system is 