## Text Groups

In [[OpenNefia-LÖVE]], lists of text were specified as a successive sequence of locale keys.

```lua
      quest_dialog = {
         text = {
            "talk.unique.renton.quest.dialog._0",
            "talk.unique.renton.quest.dialog._1",
            "talk.unique.renton.quest.dialog._2",
            "talk.unique.renton.quest.dialog._3",
         },
         choices = {
            {"quest_check", "ui.more"},
         },
      },
```

Probably shouldn't do this, because translations might require more or less lines for the final text. Instead allow translations to just specify a key that points to a list of text.

```lua
      quest_dialog = {
         text = {
            "talk.unique.renton.quest.dialog"
         },
         choices = {
            {"quest_check", "ui.more"},
         },
      },
```