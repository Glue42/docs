## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

The Themes API is accessible through the [`glue.themes`](../../../../reference/glue/latest/themes/index.html) object.

## Listing All Themes

To get a list of all available themes, use the [`list()`](../../../../reference/glue/latest/themes/index.html#API-list) method:

```javascript
const allThemes = await glue.themes.list();
```

## Current Theme

To get the currently selected theme, use the [`getCurrent()`](../../../../reference/glue/latest/themes/index.html#API-getCurrent) method:

```javascript
const currentTheme = await glue.themes.getCurrent();
```

## Selecting Themes

To select a theme, use the [`select()`](../../../../reference/glue/latest/themes/index.html#API-select) method:

```javascript
const themeName = "dark";

await glue.themes.select(themeName);
```

## Theme Events

To get notified when the theme changes, use the [`onChanged()`](../../../../reference/glue/latest/themes/index.html#API-onChanged) method:

```javascript
glue.themes.onChanged(newTheme => {
    console.log(newTheme.name);
});
```

## Reference

For a complete list of the available Themes API methods and properties, see the [Themes API Reference Documentation](../../../../reference/glue/latest/themes/index.html).