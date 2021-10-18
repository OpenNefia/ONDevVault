Localizing user interface components should be easy.

Classes in C# are organized in a hierarchical namespace layout. I am thinking of having [[UiLayer]] specify a set of string fields that hold localized text.

When the UiLayer is queried, that text should be filled in with the corresponding translations if they haven't been already.

```csharp
namespace OpenNefia.Core.UI.Layer
{
    internal class TestLayer : BaseUiLayer<string>
    {
        [UILocalize]
        private IUiTextNoArgs TextFromString;
        [UILocalize]
        private IUiTextFn1<Chara> TextFromFn1;
        [UILocalize]
        private IUiTextFn2<string, string> TextFromFn2;
        
        private FontDef FontTextWhite;

        public TestLayer()
        {
            FontTextWhite = FontDefOf.TextWhite;
            TextFromString = new UiTextNoArgs(this.FontText);
            TextFromFn1 = new UiTextFn1(this.FontText);
            TextFromFn2 = new UiTextFn2(this.FontText);
            
            if ( Rand.OneIn(2) ) {
                // If a custom namespace is set in advance, UILocalize becomes a no-op.
                TextFromString.LocaleKey = "CustomNamespace.Foo.Scut";
            }
        }
        
        public override void OnQuery() 
        {
            TextFromString.ApplyArguments();
            TextFromFn1.ApplyArguments(Chara.Player);
            TextFromFn2.ApplyArguments("foo", "ほげ");
        }

        public override void SetPosition(int x, int y)
        {
            base.SetPosition(x, y);

            this.TextFromString.SetPosition(x, y);
            this.TextFromFn1.SetPosition(x, y + This.FontText.GetHeight());
            this.TextFromFn2.SetPosition(x, y + This.FontText.GetHeight() * 2);
        }

        public override void Update(float dt)
        {
            this.TextFromString.Update(dt);
            this.TextFromFn1.Update(dt);
            this.TextFromFn2.Update(dt);
        }

        public override void Draw()
        {
            this.TextFromString.Draw();
            this.TextFromFn1.Draw();
            this.TextFromFn2.Draw();
        }
    }
}
```

Then there would be a corresponding translation file (Lua or XML) that would be namespaced under `OpenNefia.Core.Ui.Layer.TestLayer` with the translations to fill in.

```lua
return {
    ["OpenNefia.Core.UI.Layer.TestLayer"] = 
    {
        TextFromString = "Hello, world.",
        TextFromFn1 = function(_1) return ("Character: %s"):format(_1.Name) _end,
        TextFromFn2 = function(_1, _2_) return ("Args: %s %s") end,
    },
    
    ["CustomNamespace.Foo"] = {
        Scut = "Scut!"
    },
    
    -- alternatively
    
    OpenNefia = {
        Core = {
            UI = {
                Layer = {
                    TestLayer = {
                        -- ...
                    }
                }
            }
        }
    }
}
```

This would be a high-level abstraction over a bare-bones API like `OpenNefia.Core.I18N.Get("CustomNamespace.Foo.Scut")` that returns a string.