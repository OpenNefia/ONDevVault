Extra data and behavior that can be added to [[Map Objects]] and [[Maps]].

I *don't* think that all Aspects should inherit from a single generic class. Is there ever a case in which you'd want to reuse a Map Object's Aspect with a Map?

On the other hand, there are numerous instances where you might want to reuse Aspects between multiple kinds of Map Objects.

One example is potion effects. A potion can be drunk if it's on an [[Items|Item]], or it can be attached to a potion puddle if it's on a [[Feats|Feat]]. Adding a potion puddle should involve taking the potion aspect on the Item and copying/moving it to the Feat somehow.

The question is: in which way should this take place? Should Feat be a sealed class, where extensibility is controlled entirely by adding new Aspects? Or should it allow inheritance to support implementing new behaviors?

In the former case, the potion aspect might get attached directly to the feat. The `PotionAspect` itself might implement an interface like `ICanStepOn`, which controls what happens when a map object containing the aspect is stepped on. This is [[OpenNefia-LÖVE]]'s current implementation of aspects.

In the latter case, a new `Feat_PotionPuddle` would be created, and would hold the `PotionAspect` as a field. The `Feat` (or `MapObject`) base class would have some sort of `virtual TurnResult Event_OnSteppedOn(Chara)` method that is called when a character steps on the feat.

## No inheritance, composability only

The problem with the former case is that it's too brittle. Why should the potion aspect itself dictate what happens when every single map object that contains it is stepped on? What if you had a new feat, `Feat_PotionFountain`, that acts like a water fountain but instead dispenses potion? It makes no sense for the fountain to act the same way as a potion puddle when it is stepped on, but that is the behavior that `PotionAspect` would come with.

One major issue is the implicit dependent aspects that other aspects assume are contained in the same object. Combining a bunch of aspects together and expecting the extensibility to sort itself out somehow is asking for trouble. And then there's the implicit, unexpected behaviors that arise by simply attaching a new aspect to a map object. It makes no sense for a `Feat_PotionFountain` to *not* have a `PotionAspect` attached, but that's exactly what is possible with the former approach.

## Inheritance, with nested aspect holders/parents

My hesitation with the latter inheritance-based approach is that it uses inheritance at all, and we are generally taught that "inheritance == bad". Yet, this design problem is another reminder that nothing in software design is an absolute. This is a case where excessive composability hinders the quality of the final result.

It shouldn't be necessary to check if an aspect that *needs* to be on the object is there or not, if it *always* needs to be there for the code to work correctly.

It doesn't make sense for a potion puddle to *not* have a potion aspect. Therefore it ought to be a field/property on the feat.

The former composition-based approach almost sounds like the kind of design that would come out of using [[Lua|a dynamic language with no classes]] as the main implementation language...

And anyways, if a modder wants to create a new feat that changes the effects of the hypothetical `Feat_PotionPuddle` to spawn Putits fused with the effects of the tossed potion, they can simply make a new subclass of `Feat` and patch the logic where the potion is broken to spawn that feat instead of the original.

If a modder wants to affect anything that has a potion effect, like turning the potion puddles, potions in one's inventory, or even the fused potion Putits to be made of poison instead, that would be a job for Aspects. `Aspect` would expose a method, `void GetChildAspectHolders(out List<IAspectHolder>)`, which returns a list of things that can hold aspects inside any nested aspects. It would also expose `IEnumerable<Aspect> GetDirectlyHeldAspects()` for enumerating the aspects in the current holder. Then you could call this on every aspect on the map/charas/items to get all the aspects for everything in a nested fashion, filter on the ones where `aspect is PotionAspect`, and perform the changes you want.

## Composition, with nested aspects

But why not just have a `PotionPuddleAspect` that wraps a `PotionEffectProperties`, and attach that to the feat instead of attaching a `PotionAspect` directly?

I think it is because it makes the use action logic more complicated.

Consider item use logic. Say this approach is implemented, and you decide you want to combine a `PotionDrinkableAspect` and a `WellDrinkableAspect` together, for some weird reason.

`PotionDrinkableAspect`'s behavior is such that the item should be consumed after the drink action finishes.

What happens when you try to drink such an object?

I think allowing people to combine arbitrary Aspects together and expecting everything to just work isn't sustainable.

So my thinking with this is that `WellDrinkableAspect` and `PotionDrinkableAspect` would be mutually incompatible. They should not be aspects.

## Testing theories

Assume option 2 is chosen.

So the potion will have some kind of `OnDrink(Chara chara)` callback, *somewhere*. The difference from feats is that [[Items]] are `sealed`, and cannot be inherited from.

And it might not be a potion. It could be anything with an `OnDrink(Chara)` callback as well.

`Feat_PotionPuddle` would store the Item responsible for the puddle. When it is stepped on, it takes the item and triggers the item's `OnDrink(Chara)` on the character that stepped on it.

So `OnDrink()` should not consume the item. There should be an ability to add two `PotionAspect`s with `OnDrink()` callbacks that are both called. You should also be able to create a filtration bottle that is not consumed when drinking out of it. Something about the aspect should determine whether or not it is consumed on drink. `bool ShouldConsumeOnDrink { get; }` might be an option. If any aspect has `ShouldConsumeOnDrink` equal to false, then don't drink it.

Consider a filtration bottle (`FiltrationBottleAspect`) that you can fill up with an `IDrinkableAspect`. This drinkable could be anything, not just a potion.

When running the `OnDrink()` callback on the parent item, `OnDrink()` should only be called on the `FiltrationBottleAspect`, *not* the `IDrinkableAspect` that it contains, so that the bottle can check if it still has any liquid left. But we still want that `IDrinkableAspect` to be available in the Aspect parent-child hierarchy for modification purposes, like turning it to poison.

So I guess a rule could be decided: when a callback is run, only the aspects that are direct descendants of the item will have their callbacks run. Anything more granular would have to be handled by the individual aspects themselves (`FiltrationBottleAspect` would call `OnDrink()` within its own logic).

And of course there should be a way to determine if this "callback rule" holds for any arbitrary Aspect in the tree. That could be a virtual method: `virtual bool RunsAspectCallbacksIfNested(IAspectHolder parent, string callbackType)`. The default impl would be `this.Parent is MapObject`, where `parent` is a thing can contain aspects. (Aspects themselves would implement this interface.)

To move an aspect between `IAspectHolder`s, you'd take the aspect properties and pass them the constructor for that aspect type. All aspects would have a uniform interface like this, requiring an `Initialize(AspectProperties)` method.

## Test Cases

### Drinking a potion

1. Potion is defined like so.
```xml
<ItemDef Id="PotionOfHealingAndSpeed">
    <Categories>
        <li>Core.Potion</li>
    </Categories>
    <Aspects>
        <li Class="Core.PotionAspect">
            <Effect class="Core.MagicEffect">
                <Id>Core.HealCritical</Id>
                <Power>100</Power>
            </Effect>
        </li>
        <li Class="Core.PotionAspect">
            <Effect class="Core.MagicEffect">
                <Id>Core.TrollsBlood</Id>
                <Power>100</Power>
            </Effect>
        </li>
    </Aspects>
</ItemDef>	
```

`PotionAspect` is defined like so.

```csharp
public class PotionAspect : MapObjectAspect, ICanDrinkAspect, ICanBeThrownAspect
{
    public PotionAspectProps PotionProps => (PotionAspectProps)this.Props;
    
    public override void Initialize(AspectProperties props)
    {
    
    }
    
#region ICanDrinkAspect implementation. 
    
    // NOTE: How many should be consumed? Is it always 1?
    public bool ShouldConsumeOnDrink => true;
    
    public void OnDrink(Chara chara) 
    {
        // NOTE: If this item is being thrown, then this.Parent 
        // still has to be the throwing character for 
        // hostile actions to work this implementation of Effects.
        MapObject potionItem = this.Parent;
        
        var args = new EffectArgs(chara, source: potionItem);
        PotionProps.Effect.Apply(args);
    }
    
#endregion

#region ICanBeThrownAspect
    
    // NOTE: How many should be destroyed? Is it always 1?
    public bool ShouldDestroyOnThrow => true;
    
    public void OnThrownImpact(InstancedMap map, int x, int y) 
    {
        MapObject potionItem = this.Parent;
        
        var chara is map.GetOptionalAtPos<Chara>(x, y).FirstOrDefault();
        if (chara != null) 
        {
            this.OnDrink(chara);
        }
        else 
        {
            Feat puddle = Feat.Create(FeatDefOf.DrinkablePuddle, map, x, y);

            var props = new Feat_DrinkablePuddleProps();
            IAspect aspect = Aspect.CreateFromProps<Feat_DrinkablePuddle>(props);

            puddle.AddAspect()
        }
    }

#endregion
}

[AspectClass(typeof(PotionAspect))]
public class PotionAspectProps : AspectProperties
{
    [DefRequired]
    IEffect Effect = null!;

    public PotionAspectProps() 
    {
    
    }
}
```
