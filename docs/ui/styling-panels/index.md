---
title: "Styling Panels"
icon: "🖌️"
created: 2024-09-24
updated: 2024-09-25
---

# Styling Panels

# Using Stylesheets



:::info
It is common to have a file system like this:


* Health.razor
* Health.razor.scss


If you do this, the scss file is automatically included by your Health.razor panel.


:::

If you want to specify a different location for the loaded Stylesheet, you can add this to your Panel class:

```csharp
[StyleSheet("main.scss")]
```

You can also import a stylesheet from within another stylesheet like so:

```css
@import "buttons.scss";
```

# Styling Directly

There are a few ways to style your Panels without a stylesheet. It's really recommended that you use a stylesheet to keep things organized, but there are also valid reasons to use the following methods.

### Styling the Element

You can directly style any element just like you can in HTML, but can inject C# when necessary:

```markup
<label style="color: red">DANGER!</label>
<div class="progress-bar">
  <div class="fill" style="width: @(Progress * 100f)%"></div>
</div>
```

### Style Block

Before or after your `<root>` element, you can add a `<style>` block that is read just like a Stylesheet:

```markup
<style>
  MyPanel {
    width: 100%;
    height: 100%;
  }
  .hp { color: red; }
  .armor { color: blue; }
</style>
```

### Styling in Code

You can a Panel's [Style](https://sbox.game/api/Sandbox.UI.BaseStyles) directly and modify the values however you'd like via `Tick()` or `OnUpdate()`:

```csharp
myPanel.Style.Width = Length.Percent(Progress * 100f);
```
