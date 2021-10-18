## Effects

An Effect is the most minimal piece of logic that can be applied by a source object to a target object.

An Effect is an abstraction that was not a part of 1.22's original engine design. It is a method of implementing composable game logic with its own benefits and drawbacks, and its adoption would significantly influence the design of several major engine components that require custom logic ([[Spells]], [[Buffs and Hexes]], item use effects...).

An Effect does not require identifiers, metadata, or anything else to be attached to it.

An Effect does not specify any hard requirements on instance data. An Effect can be implemented with no data needing to be passed to its constructor.

An Effect has two stages of use: construction, and application to entities.

An Effect is intended to be constructed once by declaring it on a [[Defs|Def]] along with its named constructor parameters, then reusing it for multiple applications, to avoid unnecessary allocation. But if necessary, an Effect is also capable of being instantated elsewhere in the code for advanced use cases.

An Effect is capable of being instantiated through XML by giving its class name and the list of parameters to pass to its constructor. This may need to rely on runtime reflection and introspecting named parameters in the constructor.

```xml
<ElementDef Id="Fire">
	<OnDamageLocationEffect Class="Core.DamageFireEffect">
		<Power>100</Power>
	</OnDamageLocationEffect>
	<OnDamageCharaEffect Class="Core.DamageElementalEffect">
		<Power>50</Power>
		<Element>Core.Fire</Element>
	</OnDamageCharaEffect>
</ElementDef>
```

An Effect is well-suited for composition with other Effects, by passing the child effects to its constructor.

```xml
<MagicDef Id="RandomBolt">
	<OnCastEffect Class="Core.PickOneRandomly">
		<Effects>
			<li>
				<Weight>3</Weight>
				<Effect Class="Core.BoltMagicEffect">
					<Element>Core.Fire</Element>
				</Effect>
			</li>
			<li>
				<Weight>7</Weight>
				<Effect Class="Core.BoltMagicEffect">
					<Element>Core.Cold</Element>
				</Effect>
			</li>
			<li>
				<Weight>5</Weight>
				<Effect Class="Core.BoltMagicEffect">
					<Element>Core.Lightning</Element>
				</Effect>
			</li>
		</Effects>
	</OnCastEffect>
</MagicDef>
```

Effects would be used to model things like the instant effects of spells and the passive effects of buffs/hexes/status effects. Passive and active effects should be interchangeable. Effects should not care about whether or not they affect [[Stats|Stat]] values on the target.

An Effect defined for the purpose of something like magic should be easily reusable for a completely different purpose, such as when using an item, or behavior that can be added to a trap, with minimal or no code changes.

```csharp
public class LineMagicEffect : IEffect
{
	public Animation Animation;
	
	public Effect InnerEffect;
	
	public LineMagicEffect(Animation animation, Effect innerEffect)
	{
		this.Animation = animation;
		this.InnerEffect = innerEffect;
	}
	
	public EffectResult Apply(EffectParams params)
	{
		Animation.Play(params.Source, params.Target);
		
		foreach (var (x, y) in Pos.EnumerateLine(params.Source.X, params.Source.Y, params.X, params.Y)) {
			params.X = x;
			params.Y = y;
			params.Target = Chara.At(x, y).FirstOrDefault()
			this.InnerEffect.Apply(params);
		}
        
		return EffectResult.Success;
	}
}
```

An Effect would return a result of some kind. There probably can't be generic parameters because that makes the interface less flexible than what I'm imagining. The result would probably be an enum indicating success or failure. There would need to be enough information for things like [[Spells]] to determine if a turn needs to be passed, or if control needs to be returned to the player.