1. Prevent a character from picking up items with a potion category.
2. Prevent a character from picking up anything that has a nested `PotionAspect` with the type of magic set to poison.
3. Prompt for pickpocketing a non-owned item on the ground if trying to pick it cup.
4. Change the logic for all potions with magical effects that can break to spawn Putits or something that knows how to apply the [[Effects|Effect]] the potion holds on themselves. ("How to apply" could mean that they learn a spell, or gain a custom serialized action that isn't in any SpellDef).
5. Cast a spell that turns all potions in the entire map and all item/character/other inventories into poison.
6. Create an "eBook" item that acts as a container which only accepts "software" items.
7. Cast a spell that *mojibake*'s any digital books that are stored, but only if they are loaded into an eBook already.
8. Create a liquid system. A liquid can have a fluid volume and be stored in a container like a barrel. Destroying the container should create puddles of the liquid everywhere. An individual fluid item acts as `IDrinkable()`, but only if it's in a proper container. The amount of liquid affects the power level of the `Effects` that drinking the liquid causes.
9. Modify all existing ItemDefs that are potions to take advantage of the liquid system. (*NOTE*: Impossibly hard? Pointless???)