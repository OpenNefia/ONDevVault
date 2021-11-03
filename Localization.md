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

## Fluent

Localization system developed by Mozilla. Has at least two implementations: Fluent.NET and L

### 

