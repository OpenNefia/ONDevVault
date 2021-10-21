Extra data and behavior that can be added to [[Map Objects]] and [[Maps]].

I *don't* think that all Aspects should inherit from a single generic class. Is there ever a case in which you'd want to reuse a Map Object's Aspect with a Map?

On the other hand, there are numerous instances where you might want to reuse Aspects between multiple kinds of Map Objects.

One example is potion effects. A potion can be drunk if it's on an item, or it can be attached to a potion puddle if it's on a feat.