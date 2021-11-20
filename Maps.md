## Loading and Unloading

A major problem with [[OpenNefia-LÖVE]]'s design was the fact that there was no central store for tracking loaded maps. This meant that a map loaded twice from disk would reference two separate copies of the same map, which became a headache when the same map needed updates in multiple places.

See [issue #205](https://github.com/Ruin0x11/OpenNefia/issues/205).

```lua
local map_uid = 1
local ok, map1 = Map.load(map_uid)
local ok, map2 = Map.load(map_uid)
assert(map1 == map2) -- fails
```

In OpenNefia.NET, there will be some kind of global store that will manage maps. Maps will not be saved or loaded individually anymore, only when they're being managed as part of a game save. A map can still be instantiated as a standalone object, but it cannot be set as the active map until the map storage is tracking it.

## Map Transactions

For simplicity's sake, editing a map that isn't the current one could be handled with a "transaction" system.

You can specify a closure with logic to run on the loaded map, and the map is modified and saved back to disk in a single transaction.

```csharp
var mapId = 14;
MapLoader.EditMap(mapId, (map) => map.Clear(TileDefOf.Grass));
```

This would take care of multiple concurrent mutations, where the map currently being edited in memory is passed by `MapLoader` if `EditMap` is inadvertently called twice on the same map.

Then you could call `MapLoader.FlushPendingTransactions()` at some point to run all the closures in sequence.