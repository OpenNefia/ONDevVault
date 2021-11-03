## Foundation

There are a couple of requirements for the localization system:

1. Arbitrary functions usable inline with the translated text for pronouns/gender/etc.
2. Translation data is easily marshallable into appropriate C# data structures. Good examples where this is needed are item description lists, dictionaries with arbitrary string keys, [[UI Localization|complex UI widgets]], etc.
3. Ability to merge/override translations with an external mod. Needed for adding the names/descriptions of new map object definitions, adding new string-type enum variants to a dictionary, etc.
4. I would strongly prefer namespacing, per-mod at an absolute minimum.

The options I see are:

### Lua

This is the way we used to do it, and the crucial thing is that *it worked.*

#### Benefits

1. Metatables allow us to develop a pseudo-DSL for specifying namespaces.
2. Flexible enough to allow for any sort of necessary logic with plain Lua functions.

#### Drawbacks

1. Too general-purpose. People would be able to mess with the marshalled C# objects from the script side.
2. Lua objects need to be munged into the correct datastructures on the C# side, instead of being able to store them directly.
3. Not as straightforward for non-programmers to use.
4. Somewhat annoying to reference other translations from within a localization function.
5. It's possible for script functions to throw errors.

### Fluent

Localization system developed by Mozilla. Has at least two .NET implementations: [Fliuent.Net](https://github.com/blushingpenguin/Fluent.Net) and [Linguini](https://github.com/Ygg01/Linguini).

####  Benefits

1. Domain-specific. Only concerns itself with the functions/logic necessary for localization, and nothing more.
2. Easier syntax for translators to work with. 
3. No possibility for runtime errors except for the built-in/bound functions.
4. Can easily validate the translation files.
5. Can easily reference other translation keys within a message.
6. It describes itself as a proper localization system that solves the problems I'm having with pronouns/etc., so maybe I feel better about myself for using it instead of trying to glom something weird onto Lua...?

#### Drawbacks

1. No namespacing. Everything is global.
2. No structural organization. Everything is a single key, and attributes cannot be nested. The resulting lists of keys are eye-watering to look at since the same prefixes are repeated throughout the entire file. 
3. No nesting also makes the proposed [[UI Localization]] system infeasible.

I will probably stick with Lua, for the sake of the things that Fluent makes impossible.

## Namespacing

Should we have it or not? To what extent should it be used?

[[OpenNefia-LÖVE]] had the problem where all the translations were haphazardly shoved into random places with little sense of organization. But I think this was an organizational issue, not a deficit of namespaces per se. Namespaces in ON-LÖVE were useful at perhaps the last one or two levels of nesting, after which it was a crapshoot if you were trying to look for something in a translation based solely on the directory structure.

### Benefits

1. Ease of mapping from a translation file to the C# namespace it concerns.
2. Per-mod separation of translations to avoid collisions, without needing to manually prefix everything. At some point, you will *need* to prefix things, whether or not the engine takes care of it for you.
3. Allows for [[UI Localization]] by mapping the namespace/type name of the class to the localization namespace.

### Drawbacks

1. Extreme verbosity if the namespace nesting is too deep.

```csharp
var text = I18N.Get("OpenNefia.Core.Logic.TargetText.YouAreTargeting", Current.Player);
```

This could be solved in a number of ways. If the class is not static, we could have a "root path" helper class.

```csharp
var root = new I18NRoot("OpenNefia.Core.Logic.TargetText");
var text = root.Get("YouAreTargeting");
```

Unfortunately this makes the code significantly less greppable if you can use any arbitrary root. Maybe we will restrict it to types only.

```csharp
var root = new I18NRoot(typeof(TargetText));
var text = root.Get("YouAreTargeting");
```

But this doesn't help with static classes. Maybe moving the namespacing into the `Get()` function would be better.

```csharp
var text = I18N.Get(typeof(TargetText), "YouAreTargeting");
```

I don't know about the performance of this.

Alternatively, we could give up on a 1:1 namespace mapping for static classes.

```csharp
var text = I18N.Get("Core:TargetText.YouAreTargeting");
```

2. It's annoying to reference locale keys outside the class they are namespaced under. Why are we assuming a single translation will only ever be used in a single class?