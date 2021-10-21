## How is it designed?

`item_pocket` contains the list of items, and is typed by what kinds of items it can hold (general items, ammo inside magazines, bionics inside corpses, software inside USB sticks, etc).

`item_contents` contains a list of `item_pockets`. When storing an item, it searches for the best pocket to put the item into, if any, and moves it there.

## What kinds of game objects can be stored in it?

Items only.

## Can mods add new container types?

No. The list of special container types is hardcoded in `item_pocket`.

## How do I know how many objects can I store?

`item_pocket` has `weight_capacity()` and `volume_capacity()`.

**Note**: There is also an option in `volume_capacity` for getting the remaining "charges" of an item pocket based on its volume.

It's worth considering how item amount and charges will affect item weight.

## How do I get the thing that owns this container? (character, container item, map, etc.)

## How do I get all the objects in this container and its nested containers?

`item_contents` uses the Visitor pattern with `visit_contents(Func<item*, item*> &func, item* parent)`. It returns a `VisitResponse` (next, skip, abort).

## How does it implement permissions/limitations?

`item_pocket` has `can_contain(item, ignore_fullness)` that returns a `contain_code`:  wrong ammo type, insufficient weight, insufficient space, liquid in non-watertight container, gas in non-airtight container.

The permissions logic is hardcoded for the set of container types (chest, corpse, eBook, magazine, etc).

## How do I query for a list of objects on a tile?

## How do the parent object(s) get notified when objects have been added/removed?

## How does stacking work?

## Inheritance?

No.