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

## Multiple Loaded Maps

There would be some kind of "locking" mechanism to keep certain maps in memory, instead of having them released automatically when transitioning between maps. The set of active map UIDs would be serialized as part of the save. This would allow stepping the turn sequence for more than one map simultaneously.

To accomplish this there would be a `MapLock` that implements `IDisposable`, which would be freed in the finalizer when the containing object is garbage collected. This would be a reference counting mechanism. When a tracked map has no `MapLock`s referencing it, that means it's safe to unload from memory.

`MapLock`s will also have to be serialized correctly for this to work.

When all map operations are finished, like during scenario startup or after traveling between maps, call `FlushInactiveMapsFromMemory()` to unload the open maps with no `MapLock`s active. That should leave only the current map and any maps with `MapLock`s active.