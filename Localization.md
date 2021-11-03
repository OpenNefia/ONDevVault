There are a couple of requirements for the localization system:

1. Arbitrary functions useable inline with the translated text for pronouns/gender/etc.
2. Translation data is easily marshallable into appropriate C# data structures. Good examples where this is needed are item description lists, dictionaries with arbitrary string keys, [[UI Localization|complex UI widgets]], etc.
3. Ability to merge/override translations in an external mod. Needed for adding the names/descriptions of new map object definitions, adding new string-type enum variants to a dictionary, etc.
4. I would strongly prefer namespacing, per-mod at an absolute minimum.

The options I see are:

## Lua

This is the way we used to do it, and it worked.

### Benefits

1. Metatables allow us to develop a pseudo-DSL for specifying namespaces.
2. Flexible enough to allow for any sort of necessary logic with plain Lua functions.

### Drawbacks

1. Too general-purpose. People would be able to mess with the marshalled C# objects from the script side.
2. Lua objects need to be munged into the correct structures on the C# side, instead of being able to store them directly.
3. Not as straightforward for non-programmers to use.
4. It's possible for script functions to throw errors.

## Fluent

Localization system developed by Mozilla. Has at least two implementations: [Fliuent.Net](https://github.com/blushingpenguin/Fluent.Net) and [Linguini](https://github.com/Ygg01/Linguini).

###  Benefits

1. Domain-specific. Only concerns itself with the functions/logic necessary for localization, and nothing more.
2. Easier syntax for translators to work with. 
3. No possibility for runtime errors except for the built-in/bound functions.
4. Can easily validate the translation files.
5. It describes itself as a proper localization system that solves the problems I'm having with pronouns/etc., so maybe I feel better about myself for using it instead of trying to glom something weird onto Lua...?


### Drawbacks

1. No namespacing. Everything is global.
2. No structural organization. Everything is a single key, and attributes cannot be nested. The resulting lists of keys are eye-watering to look at since the same prefixes are repeated throughout the entire file. 
3. No nesting also makes the proposed [[UI Localization]] system infeasible.


