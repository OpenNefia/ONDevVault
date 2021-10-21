Extra data and behavior that can be added to [[Map Objects]] and [[Maps]].

I *don't* think that all Aspects should inherit from a single generic class. Is there ever a case in which you'd want to reuse a Map Object's Aspect with a Map?

On the other hand, there are numerous instances where you might want to reuse Aspects between multiple kinds of Map Objects.

One example is potion effects. A potion can be drunk if it's on an [[Items|Item]], or it can be attached to a potion puddle if it's on a [[Feats|Feat]]. Adding a potion puddle should involve taking the potion aspect on the Item and copying/moving it to the Feat somehow.

The question is: in which way should this take place? Should Feat be a sealed class, where extensibility is controlled entirely by adding new Aspects? Or should it allow inheritance to support implementing new behaviors?

In the former case, the potion aspect might get attached directly to the feat. The `PotionAspect` itself might implement an interface like `ICanStepOn`, which controls what happens when a map object containing the aspect is stepped on. This is [[OpenNefia-LÖVE]]'s current implementation of aspects.

In the latter case, a new `Feat_PotionPuddle` would be created, and would hold the `PotionAspect` as a field. The `Feat` (or `MapObject`) base class would have some sort of `virtual TurnResult Event_OnSteppedOn(Chara)` method that is called when a character steps on the feat.

The problem with the former case is that it's too brittle. Why should the potion aspect itself dictate what happens when every single map object that contains it is stepped on? What if you had a new feat, `Feat_PotionFountain`, that acts like a water fountain but instead dispenses potion? It makes no sense for the fountain to act the same way as a potion puddle when it is stepped on, but that is the behavior that `PotionAspect` would come with.

The core of the issue is the implicit dependent aspects that other aspects assume are contained in the same object, and the implicit unexpected behaviors that arise by simply attaching a new aspect to a map object. Combining a bunch of aspects together and expecting the extensibility to sort itself out somehow is asking for trouble. Furthermore, it makes no sense for a `Feat_PotionFountain` to *not* have a `PotionAspect` attached, but that's exactly what is possible with the former approach.