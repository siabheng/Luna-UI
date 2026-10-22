# Luna UI Documentation

This guide describes the API implemented by the Luna source in this repository. It is a reference for this version of the library; check the source when changing or adding public methods.

## Load the library

The raw `README.md` URL below serves the Lua source for this library:

```lua
local Luna = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/siabheng/Luna-UI/refs/heads/main/README.md",
    true
))()
```

Only load code from a repository and branch you trust. The library relies on Roblox UI services and executor-specific functions, so this loading pattern is not suitable for a normal Roblox `Script` or `LocalScript` in Studio. For a Studio project, package the source as a `ModuleScript` and require it instead.

## Create a window

The library creates one window. Unspecified settings use the defaults shown here.

```lua
local Window = Luna:CreateWindow({
    Name = "Example",
    Subtitle = "",
    LogoID = "6031097225",
    LoadingEnabled = true,
    LoadingTitle = "Luna Interface Suite",
    LoadingSubtitle = "by Nebula Softworks",
    ConfigSettings = {
        RootFolder = nil,
        ConfigFolder = "Big Hub",
    },
    KeySystem = false,
    KeySettings = {
        Title = "Example key",
        Subtitle = "Key System",
        Note = "Enter your key",
        SaveInRoot = false,
        SaveKey = true,
        Key = {"Example Key"},
        SecondAction = {
            Enabled = false,
            Type = "Discord",
            Parameter = "",
        },
    },
})
```

`KeySettings.SecondAction.Type` supports `"Discord"` or `"Link"`. Key checks performed by a client UI are not a substitute for server-side authorization.

### Change the window key

Set `Window.Bind` to an `Enum.KeyCode`. The default is `Enum.KeyCode.K`.

```lua
Window.Bind = Enum.KeyCode.Quote
```

### Destroy the window

```lua
Luna:Destroy()
```

Destroying the UI does not stop callbacks or other script tasks you started. Stop those separately before destroying the window.

## Tabs and sections

Create tabs from the window. The first tab activates automatically.

```lua
local Tab = Window:CreateTab({
    Name = "General",
    Icon = "home",
    ImageSource = "Material",
    ShowTitle = true,
})
```

`ImageSource` may be `"Material"`, `"Lucide"`, or `"Custom"`. With `"Custom"`, set `Icon` to the numeric asset ID without the `rbxassetid://` prefix. Named icons must exist in the selected built-in icon catalog; for example, `"dark_mode"` is available in the Material catalog. The tab settings default to a `"Tab"` name, the Material `view_in_ar` icon, the Material icon source, and a visible title. Unsupported icon names produce a descriptive error rather than assigning a missing image.

### Home tab

The window can show its built-in home page:

```lua
local Home = Window:CreateHomeTab({
    SupportedExecutors = {"Studio"},
    DiscordInvite = "invite-code",
    Icon = 1,
})
```

`Icon = 1` uses the default home icon; `Icon = 2` uses the dashboard icon. `Home:Activate()` selects the page.

Group controls in a section. Sections can also be created directly on a tab.

```lua
local Section = Tab:CreateSection("Controls")
Section:Set("Updated heading")
Section:Destroy()
```

Both tabs and sections support the interaction and text elements in this guide. `Tab:Activate()` selects a tab; `Tab:CreateDivider()` and `Section:CreateDivider()` add dividers.

## Notifications

```lua
Luna:Notification({
    Title = "Saved",
    Content = "Your settings were saved.",
    Icon = "info",
    ImageSource = "Material",
})
```

Defaults are provided for omitted fields. Icons use the same `ImageSource` conventions as tabs.

## Interactive elements

All interactive elements may be created on a tab or section. A returned element generally exposes its settings in `.Settings`, supports `:Set({...})` to update settings, and supports `:Destroy()` to remove it.

### Button

```lua
local Button = Section:CreateButton({
    Name = "Run",
    Description = "Run this action",
    Callback = function()
        print("Button pressed")
    end,
})
```

`Description` is optional. `Callback` is called without arguments.

### Toggle with a keybind

`CreateToggleBind` combines a clickable toggle with a rebindable key selector. It is available on both tabs and sections.

```lua
local ToggleBind = Section:CreateToggleBind({
    Name = "Auto action",
    Description = "Click the toggle or press the key",
    CurrentValue = false,
    CurrentBind = "Q",
    Callback = function(active)
        print("Auto action enabled:", active)
    end,
    OnChangedCallback = function(keyName)
        print("New key:", keyName)
    end,
}, "AutoActionFlag")
```

`CurrentValue` defaults to `false` and sets the initial displayed state; the callback is not called just because the control is created. If the initial value is `true`, the switch appears on, but the callback is only called after the user changes the state, presses its bound key, or loads a saved state. The toggle switch appears before the key selector. `CurrentBind` is a Roblox `Enum.KeyCode` name such as `"Q"` or `"F"`. Clicking the toggle or pressing the bound key switches the active state and calls `Callback` with the new boolean value. `OnChangedCallback` receives the new key name as a string when the key is rebound. `HoldToInteract = true` changes the key behavior to held-state callbacks rather than toggling on each press.

`:Set({CurrentValue = true})` updates the active state and calls `Callback` with the requested state; `:Set({CurrentBind = "F"})` updates the displayed bind. Setting `Active` is also supported for compatibility with button-bind configs. Loading a flagged toggle-bind restores its active state through the callback as well as restoring its key. `:Destroy()` disconnects the key listener. `CreateButtonBind` remains available as a backwards-compatible method.

### Toggle

```lua
local Toggle = Section:CreateToggle({
    Name = "Enabled",
    Description = "Enable this option",
    CurrentValue = false,
    Callback = function(value)
        print("Enabled:", value)
    end,
}, "EnabledFlag")
```

`CurrentValue` is a boolean. The second argument is an optional configuration flag.

### Keybind

```lua
local Bind = Section:CreateBind({
    Name = "Action",
    Description = nil,
    CurrentBind = "Q",
    HoldToInteract = false,
    Callback = function(active)
        print("Active:", active)
    end,
    OnChangedCallback = function(keyName)
        print("New key:", keyName)
    end,
}, "ActionBind")
```

With `HoldToInteract = false`, each press toggles the active state. With it enabled, the callback receives the held state. The optional second argument is a configuration flag. `CreateKeybind` is an older method; prefer `CreateBind`.

### Slider

```lua
local Slider = Section:CreateSlider({
    Name = "Amount",
    Range = {0, 200},
    Increment = 5,
    CurrentValue = 50,
    Callback = function(value)
        print("Amount:", value)
    end,
}, "AmountFlag")
```

`Range` is `{minimum, maximum}`; `Increment` controls value steps. The second argument is an optional configuration flag.

### Text input

```lua
local Input = Section:CreateInput({
    Name = "Player name",
    Description = nil,
    PlaceholderText = "Enter a name",
    CurrentValue = "",
    RemoveTextAfterFocusLost = false,
    Numeric = false,
    Enter = true,
    MaxCharacters = nil,
    Callback = function(text)
        print("Input:", text)
    end,
}, "NameInput")
```

When `Enter` is true, the callback runs after the user submits the field. When false, it runs as the text changes. `Numeric` restricts input to numeric characters, and `MaxCharacters` optionally limits the length. The second argument is an optional configuration flag.

### Dropdown

```lua
local Dropdown = Section:CreateDropdown({
    Name = "Mode",
    Description = nil,
    Options = {"First", "Second"},
    CurrentOption = {"First"},
    MultipleOptions = false,
    SpecialType = nil,
    Callback = function(selection)
        print("Selected:", selection)
    end,
}, "ModeFlag")
```

With `MultipleOptions = false`, the callback receives the selected option as a string. With it enabled, the callback receives a table of selected strings. `SpecialType = "Player"` fills the options from the player list. The second argument is an optional configuration flag.

### Color picker

```lua
local ColorPicker = Section:CreateColorPicker({
    Name = "Accent",
    Color = Color3.fromRGB(86, 171, 128),
    Callback = function(color)
        print("Color:", color)
    end,
}, "AccentFlag")
```

The callback receives a `Color3`. The second argument is an optional configuration flag.

## Text elements

### Label

```lua
local Label = Section:CreateLabel({
    Text = "Status: ready",
    Style = 1,
})
```

Styles `1`, `2`, and `3` are the standard, information, and warning styles. Use `Label:Set("New text")` to change the label.

### Paragraph

```lua
local Paragraph = Section:CreateParagraph({
    Title = "Information",
    Text = "Longer text is sized to fit the paragraph.",
})
```

Use `Paragraph:Set({Title = "Updated", Text = "Updated text"})` to change it.

## Updating, reading, and removing elements

Use `:Set({...})` to update an element. Any settings you omit keep their existing values.

```lua
Toggle:Set({
    Name = "New name",
    CurrentValue = true,
})

print(Toggle.CurrentValue)
print(Toggle.Settings.Name)
Toggle:Destroy()
```

Value properties depend on the element: for example, toggles, sliders, and toggle-binds expose `CurrentValue`; button-binds also expose `Active` and `CurrentBind`; dropdowns expose `CurrentOption`; and color picker callbacks receive a `Color3`.

## Configuration and theme helpers

Add a configuration UI to a tab with:

```lua
local ConfigTab = Window:CreateTab({Name = "Configuration"})
ConfigTab:BuildConfigSection()
```

Configuration persistence uses executor file APIs and is unavailable in Studio. Elements must have unique flags to participate in configuration saving. Toggle-binds are saved with their current key and active state. The library also provides:

```lua
Luna:SaveConfig("settings-name")
Luna:LoadConfig("settings-name")
Luna:RefreshConfigList()
Luna:LoadAutoloadConfig()
```

`LoadAutoloadConfig()` loads the selected autoload configuration when one is set. Call it after creating the controls that should receive saved settings.

Add the built-in theme controls to a tab with:

```lua
ConfigTab:BuildThemeSection()
```

## Callback and configuration notes

- Use a unique flag for each element you want to store in configuration.
- A callback error is reported by the affected control; keep application logic in your own callback functions.
- The library expects Roblox UI services and, for remote loading and configuration persistence, environment-specific capabilities. Verify the capabilities of your target environment before using those features.
