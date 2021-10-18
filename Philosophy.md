## Extensibility only when necessary.

One of the core tenants of [[OpenNefia-LÖVE]] was to make everything as extensible as possible. No real effort was put into having a system that can be maintained by multiple people for years on end.

Imagine what would happen if Minecraft were written *entirely* in Lua, with no Java components whatsoever. Now imagine what happens when you have hundreds of people adding on more code to that Lua monolith in ways that are not possible for a single maintainer to imagine.

That is the situation I'm foreseeing with ON-LÖVE.

With too much extensibility, the problem is that there is no limit to the number of ways people can break things.

As a person who is proficient at programming and enjoys building a system by oneself, alone, in a sterile environment, this makes ON-LÖVE a boon. I originally wanted a version of Elona you could program like Emacs. That meant having runtime reloading of nearly the entire codebase, introspection features for running the entire game detached from a renderer, a large number of intricate APIs to build entirely new systems on top of, and so on.

It was a fun experiment, and the development of ON-LÖVE has become the most personally engaging activity I've ever partaken in.

However, ON-LÖVE was not created with the intention of shipping a finished project.

ON-LÖVE was built to satisfy my own curiosity, and to actualize a tantalizing vision, and nothing more.

That means that ON-LÖVE was not designed to be an application that multiple people could build on top of without significant extensibility issues arising. It is impossible to know what people will want, and because ON-LÖVE gives them unlimited power to obtain those wants themselves, the number of cross-interactions between those wants and their implementations is limitless.

And every time something breaks, it is going to fall onto the shoulders of whoever maintains ON-LÖVE to make sure that every single bug, feature and ridiculous metaprogramming construct can exist in the same digital space without things falling apart within the span an hour.

For a game that you can potentially log hundreds of hours into in a single save, a single issue arising from the near-unparalleled extensibility that ON-LÖVE offers has the potential to be devastating.

Game-as-Emacs is an interesting approach to game development. It is also a fun approach. But Game-as-Emacs will not allow OpenNefia to become a properly moddable version of Elona that *everyone* can approach, not just a programmer in their solitude with a bottomless passion to build new things.

Emacs as a philosophy mostly works out because the people who buy into that philosophy the most are solitary programmers that like tinkering with things for the take of tinkering with them, in addition to reaping the benefits the tinkering produces, and *also* have the capability to program themselves out of a hole if things become unstable. Using Emacs without programming anything is missing out on a lot of its power, but it is not necessarily the case that people who enjoy modded games ought to be programmers with deep knowledge of the system to not have their game fall apart.

And even as a programmer, the worries about forgetting what you named an object property and things breaking because of that are persistent and unlikely to ever go away.

ON-LÖVE is extensible to a fault.

The point is this: because OpenNefia's new codebase is going to be produced with the express intention of ease of maintainability, the ability to add frequently requested interfaces for extensibility at a later time should be easy, even if the implementation language isn't quite as dynamic as Lua. Code quality and a decent enough design are what allows us to not have to worry about that ideal of modding the things we want to becoming unattainable, neither of which 1.22 had in the beginning.

There are a lot of things you can mod with ON-LÖVE's current design, so that isn't an issue. Taking that design and stabilizing it is the next step.

My initial worry was that I didn't know at the time what OpenNefia would look like in its Lua incarnation, so I opted for pushing excessively into the extensibility direction, if only to get a sense of the specific kinds of extensibility I was wanting from the perspective of a modder. In the beginning, I was not confident that I could produce a design that would be capable of future extensibility. I managed to prove myself wrong, but in the process I produced a massive wad of Lua that doesn't lend itself well to a future in which people just want to play modded Elona without their save becoming unrecoverable.

And recent developments in language design have meant that it is possible to use statically typed languages without having to give up on the juicy introspection features that a dynamic language like Lua provides. Lua is ON-LÖVE's Achilles' heel. It is an excellent language for what it intends to do, but for the ultimate goal of shipping OpenNefia as a stable and collaborative modding effort, writing the *entire game* in it it wasn't the right choice.

## The simple things should be easy. The complex things should be possible.

Take the act of adding a new potion to the game. Imagine that potions and other items are specified declaratively through XML. Adding a new potion should be as easy as copying one of the earlier declarations and putting it in a new mod.

Potions would be able to specify numerical power levels for the spell effects they trigger. This allows the creation of healing potions with the same base effects on the player, but different magnitudes of the effect based on power.

Adding a new potion with a stronger power level shouldn't require the modder to download 20GB worth of .NET toolchain and learn a programming language, go through the arduous process of setting up a development environment on their box, compiling the thing, and then finally going into the game to see their work.

The less coding that is necessary to do something as basic as changing the numerical strength of a potion, the better. Preferably, *no* coding should be necessary to accomplish such a change.

If you have someone who knows how to translate English into Korean, having them add new localizations to existing items across various mods should also be possible without the use of a compiler.

Translators shouldn't have to learn programming just to be able to contribute to the community.

Now for the other case: Imagine you want to add a new potion, but this time the power level is not a constant number. Instead you want it to vary based on something related to the player character, like their current hitpoints or hunger. The hungrier or closer to death they are, the stronger the effect of the potion.

This is not something that can be accomplished with XML alone. But it's wrong to think of this as a limitation of *XML* because it's declarative. The inability to allow for this sort of behavior would be a limitation of the *engine*.

Code-wise, this wouldn't be terribly difficult to implement as a class with a single method for scaling the power level.

This is where [[Effects]] would come in.

Imagine the potion effect you want to scale triggers an Effect, like `HealingEffect`. This Effect would have an `Apply(source, target, item, power...)` method of some kind.

To accomplish this special scaling, create a new `ScaledByHealthEffect` that is composed with `HealingEffect`. This scaled Effect would wrap the existing effect and call its `Apply()` method with the scaled power.

```csharp
public class ScaledByHealthEffect : IEffect 
{
	public Effect InnerEffect;
	
	public ScaledByHealthEffect(Effect innerEffect)
	{
		this.InnerEffect = innerEffect;
	}
    
    public virtual int ApplyScaling(Chara target, int power) 
    {
        var newPower = power;

        // Apply the scaling...

        return newPower;
    }
	
	public EffectResult Apply(EffectParams params)
	{
        params.Power = ApplyScaling(params.Target, params.Power);
        return this.InnerEffect.Apply(params);
	}
}
```

If the original healing potion looks like this as a declaration:

```xml
<ItemDef Id="PotionOfHealing">
	<OnUseEffect Class="Core.HealingEffect">
		<Power>100</Power>
	</OnUseEffect>
</ItemDef>
```

Then the custom "scaled" version might look like this:

```xml
<ItemDef Id="PotionOfHealingScaled">
	<OnUseEffect Class="MyMod.ScaledByHealthEffect">
		<InnerEffect Class="Core.HealingEffect">
			<Power>100</Power>
		</InnerEffect>
	</OnUseEffect>
</ItemDef>
```

Ultimately, despite the overarching system operating at a declarative level, it is still possible to drop down into a code-driven, imperative approach if necessary. And the good thing is that other mods would be able to reuse this `ScaledByHealthEffect` if they really wanted to, without needing to write any more code.
