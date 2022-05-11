## Glue42 Windows Configuration

The global behavior of [Glue42 Windows](../../../glue42-concepts/windows/window-management/overview/index.html) in [**Glue42 Enterprise**](https://glue42.com/enterprise/) is configured via the `stickywindows.json` configuration file located in the `%LocalAppData%\Tick42\GlueDesktop\config` folder. Some of the configurable properties are explained below. For more details, see the [Glue42 Windows configuration schema](../../../assets/configuration/stickywindows.json).

*Note that some properties in the `stickywindows.json` may be overridden by properties defined in the [application configuration](../application/index.html) file of your app.*

## Glue42 Window Properties

### Edge Distance

Configure the distance between the edges of neighboring Glue42 Windows:

```json
// Constrained to values between `1-9`.
{
    "edgeDistance": 9
}
```

![Edge Distance](../../../images/sw-configuration/edge-distance.png)

### Group Caption

Hide or show the group caption:

```json
{
    "hideGroupCaption": true
}
```

![Group Caption](../../../images/sw-configuration/hide-group-caption.gif)

### Resizing with Windows Gestures

Configure the resize behavior of Glue42 Windows in a group when resizing the group with Windows gestures. Some Glue42 Windows may have size restrictions in their configuration - e.g., `"maxWidth": 200`. Specify whether the rest of the windows in the group should stay relative in size to the restricted windows or should fill the rest of the space. In the examples below, the "Client List" application has a width restriction of 400 pixels:

```json
{
    "groupMaximizedMode": "Proportional"
}
```

![Proportional](../../../images/sw-configuration/proportional.gif)

```json
{
    "groupMaximizedMode": "Fill"
}
```

![Fill](../../../images/sw-configuration/fill.gif)

### Resizing Windows in a Group

Configure the resize behavior when resizing Glue42 Windows within a group by dragging an inner border:

```json
{
    "sizingMode": "Proportional"
}
```

![Proportional Sizing](../../../images/sw-configuration/sizing-proportional.gif)

```json
{
    "sizingMode": "Single"
}
```

*Note that when using `"sizingMode": "Single"`, the `borderSize`, `frameThickness` and `singleFrameThickness` theme properties must not be set to `0`, otherwise you won't be able to resize or stick the Glue42 Windows properly.*

![Proportional Sizing](../../../images/sw-configuration/sizing-single.gif)

### Flydown Windows

*All values for the flydown delay times and intervals are in milliseconds.*

To set the delay time for showing the flydown window after the user hovers over the flydown zone:

```json
{
    "flydownShowDelay": 250
}
```

To set the delay time for allowing the callback for handling the flydown window to return a result when changing the flydown zone:

```json
{
    "flydownChangeDelay": 200
}
```

To set the delay time for hiding the flydown window (e.g., when the user moves the cursor out of the flydown zone):

```json
{
    "flydownHideDelay": 600
}
```

To set the interval at which the system will check whether the flydown window should be hidden:

```json
{
    "flydownHideCheckInterval": 150
}
```

*Note that the value of `flydownHideCheckInterval` should be less than the value of `flydownHideDelay`, otherwise the flydown window will be hidden at the end of the interval specified in `flydownHideCheckInterval` and not at the end of the delay time specified in `flydownHideDelay`.*

### Tab Header Buttons

Keep buttons on tab windows always visible or show them only on hover:

```json
{
    "tabs": {
        "buttonsAlwaysVisible": false
    }
}
```

![Proportional Sizing](../../../images/sw-configuration/buttons.gif)

### Flat When Single

Show tab windows as flat windows (without a tab header) when they are single:

```json
{
    "tabs": {
        "flatWhenSingle": true
    }
}
```

![Flat When Single](../../../images/sw-configuration/flat-single.gif)