Spells are [[Effects]] that can be cast. They wrap an Effect with a cost, target character and range. They might also have callbacks that are triggered when the skill is cast.

In turn, [[Skills]] wrap Spells to attach learned status, potential and skill leveling for each Spell on an individual character. Spell stock is tracked separately from those quantities that Skills track per character, because it does not make sense for arbitrary Skills to have spell stocks.

Elona's actions are very similar to Spells, but most do not have spell stocks or skill potential/level. It is worth thinking about whether actions should be separate from Spells, or if the type of Spell should be specified in its declaration as `Spell` or `Action`.

Also, a select set of actions (Pickpocket, Riding, Cooking, etc.) *do* have a corresponding level/potential, along with an action that can be triggered from the actions menu. This must be taken into account.

## Definition Parameters

- Effect: The [[Effects|Effect]] this spell causes when triggered. Spells could use a compositional Effect implementation to support running multiple Effects in sequence, randomly from a pool, or in any number of custom ways.
- Type: Action/Spell. As mentioned above.
- Alignment: Positive/neutral/negative. Affects aggro when casting the spell on a neutral target.
- Cost: Cost in MP or stamina to apply when using the spell/action, respectively. It may be worth splitting this into separate stamina and MP costs, instead of having them depend on the spell type.
- Range: Range in tiles of the spell. Also affects AI targeting and spell usage.
- Difficulty: Difficulty of triggering the spell.
- Target Type: The target the spell affects when casting it.
  + Source: Affects the caster.
  + Source Or Nearby: Affects the caster or someone next to them. Requires a target character. Used with rods.
  + Nearby: Affects someone next to the caster. Requires a target character.
  + Location: Affects an (X, Y) position on the map.
  + Target Or Location: Affects the character's current target or the targeted (X, Y) position. Used with breath actions and bolt magic.
  + Enemy: Affects the currently targeted character, prompts if friendly.
  + Other: Affects the currently targeted character, does not prompt.
  + Direction: Affects an (X, Y) position next tho the caster.

## Relation to Effects

A major problem with Spells in regards to [[Effects]] is the hard requirement to quantify the power of each effect. When a spell is cast, certain parameters like its power level must be available to the spell logic. But Effects do not impose requirements on what data is passed to them besides the source and target [[Map Objects]], so how they are instantiated in relation to spells will need to be worked out.

Spells may or may not implement the same exact interface as Effects. That would mean that any Spell could be used as an Effect as well, separate from its metadata for cost and range. This is a very powerful idea that is worth considering, especially since Spells are really just light wrappers around Effects to begin with.

There will always be a tradeoff between having the flexibility to define whatever custom parameters an Effect needs, and the pressing urge to impose requirements on how the Effect must be used.

## Casting Parameters

Spells in vanilla Elona have a uniform interface for casting. When casting a spell, the following parameters have the potential to be used. This means that Spells *must* have these parameters available as instanced data or arguments to a `Cast()` method or similar.

- Target: Character the spell affects. This must be specified in all cases.
- Source: Character casting or triggering the spell. If the spell is triggered from an item or trap, the Source will be the same as the Target.
- Item: Item that triggered the spell effect, such as a potion, rod, material kit, or other usable item.
- X: Target map X position. The map to be used is assumed to be the same as that of the Target.
- Y: Target map Y position.
- Power: Power level of the spell. This is a number value which scales with the level of the spell, action or item. It is meant to be interpreted in a freeform manner.
- Triggered by: What triggered the spell. The known variants are:
	+ Wand
	+ Scroll
	+ Spellcasting
	+ Actions (Lockpicking, Riding, etc.)
	+ Potion
	+ Thrown potion
	+ Spilt potion
	+ Trap

For simplicity, the `Apply()` function in the interface for [[Effects]] may just use these exact parameters for every kind of effect, to simplify interoperability. 