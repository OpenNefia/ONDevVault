## How is it designed?

`item_pocket` contains the list of items, and is typed by what it can hold (general items, ammo inside magazines, bionics inside corpses, software inside USB sticks).

`item_contents` contains a list of `item_pockets`. When storing an item, it searches for the best pocket to put it into, if any, and moves it there.

## What kinds of objects can be stored in it?

Items only.

## Can mods add new container types?

No. The list of special container types is hardcoded in `item_pocket`.

## How do I know how many objects can I store?

## How does it implement permissions/limitations?

`item_pocket` has `can_contain(item, ignore_fullness)` that returns a `contain_code`:  wrong ammo type, insufficient weight, insufficient space, liquid in non-watertight container, gas in non-airtight container.

## How do the parent object(s) know when things have been added/removed?

## How does stacking work?

## Inheritance?

No.