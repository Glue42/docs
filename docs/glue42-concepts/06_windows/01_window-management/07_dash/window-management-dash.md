## Opening Windows

The `Windows` component enables you to open new Glue42 Windows. Instantiate the component and assign an ID to it:

```python
import dash
import dash_glue42

​app = dash.Dash(__name__)
​
​app.layout = dash_glue42.Glue42(id="glue42", children=[
    dash_glue42.Windows(id="g42-windows")
])
```

To open a Glue42 Window, define a callback and pass the ID of the `Windows` component and its `open` property in the `Output`. For `Input` pass the ID and the property of the component that you want to trigger opening the window. When opening a new Glue42 Window, it is required to specify a unique name and a URL. Pass an `options` object if you want to specify window bounds, mode, etc.

For more details on the available window settings, see [Window Settings](#window_settings).

The following example demonstrates how to open a new window with specific bounds and a title when the user clicks a button:

```python
@app.callback(
    Output("g42-windows", "open"),
    Input("open-window", "n_clicks"),
    prevent_initial_call=True
)
def open_window(_):

    return {
        # Each Glue42 Window must have a unique name.
        "name": "glue42-docs",
        "url": "https://docs.glue42.com",
        "title": "Glue42 Docs",
        "options": {
            "width": 400,
            "height": 500,
        }
    }
```

*See the Dash [Window Management example](https://github.com/Glue42/glue-dash-example/tree/master/window-management) on GitHub.*

## Window Settings

You can specify settings per Glue42 Window either by using its [app configuration](../../../../developers/configuration/application/index.html) file, or by passing an `options` object when opening a new window.

- using the app configuration settings:

```json
{
    "type": "window",
    "name": "glue42-docs",
    "details": {
        "url": "https://docs.glue42.com",
        "height": 640,
        "width": 560,
        "left": 100,
        "top": 100,
        "mode": "flat",
        "title": "Glue42 Documentation",
        "backgroundColor": "#1a2b30",
        "focus": false
    }
}
```

- passing an `options` object when opening a new window:

```python
@app.callback(
    Output("g42-windows", "open"),
    Input("open-window", "n_clicks"),
    prevent_initial_call=True
)
def open_window(_):

    options = {
            "height": 640,
            "width": 560,
            "left": 100,
            "top": 100,
            "mode": "flat",
            "title": "Glue42 Documentation",
            "backgroundColor": "#1a2b30",
            "focus": False
        }

    return {
        "name": "glue42-docs",
        "url": "https://docs.glue42.com",
        "title": "Glue42 Docs",
        "options": options
    }
```

The table below shows all available window settings:

| Name | Type | Description | Default | Supported By | Runtime Update Support |
|------|------|-------------|---------|--------------|------------------------|
| `allowClose` | `boolean` | If `false`, the window won't contain a "Close" button. | `true` | All | Flat and HTML |
| `allowCollapse` | `boolean` | If `false`, the window won't contain a "Collapse" button. | `true` | All | Flat and HTML |
| `allowForward` | `boolean` | If `false`, the window won't contain an activity related "Forward" button. | `true` | HTML | None |
| `allowLockUnlock` | `boolean` | If `false`, the window won't contain a "Lock/Unlock" button. | `false` | All | Flat and HTML |
| `allowMaximize` | `boolean` | If `false`, the window won't contain a "Maximize" button. | `true` | All | Flat and HTML |
| `allowMinimize` | `boolean` | If `false`, the window won't contain a "Minimize" button. | `true` | All | Flat and HTML |
| `allowTabClose` | `boolean` | If `false`, the tab header won't contain a "Close" button. | `true` | Tab | None |
| `allowUnstick` | `boolean` | If `false`, the window won't unstick from other windows. | `true` | All | None |
| `autoAlign` | `boolean` | If `true`, a snapped window will adjust its bounds to the same width/height of the window it has stuck to, and/or will occupy the space between other windows (if any). | `true` | All | None |
| `autoSnap` | `boolean` | If `true`, when moving the window operation ends, the window will snap to one of the approaching edges of another window (if any edges of the other window are within the defined snapping distance).| `true` | All | None |
| `base64ImageSource` | `string` | Image as Base64 string that will be used as a taskbar icon for the window. The supported formats are PNG, ICO, JPG, APNG. | `-` | All | All |
| `borderColor` | `string` | Can be a color name such as "Red", or a hex-encoded RGB or ARGB value. | `-` | Flat | None |
| `collapseHeight` | `number` | Defines the height of the window when collapsed (in pixels). | `System titlebar height` | Flat and HTML | Flat and HTML |
| `devToolsEnable`| `boolean` | If `true`, allows opening a developer console (using `F12`) for the new window. | `true` | All | None |
| `downloadSettings`| `Object`| Object that defines file download behavior in the window. | `-` | All | None |
| `downloadSettings.autoSave`| `boolean` | If `true`, will auto save the file (without asking the user where to save it). If `false`, a system save dialog will appear. | `true` | All | None |
| `downloadSettings.autoOpenPath`| `boolean`| If `true`, will open the folder that contains the downloaded file after the download is completed. | `false` | All | None |
| `downloadSettings.autoOpenDownload`| `boolean` | If `true`, will open the download file after the download is completed. | `false` | All | None |
| `downloadSettings.enable`| `boolean`| If `true`, enables the window to download files. | `true` | All | None |
| `downloadSettings.enableDownloadBar`| `boolean`| If `true`, a download bar tracking the progress will appear at the bottom of the window when downloading. If `false`, the download process will be invisible. | `true` | All | None |
| `downloadSettings.path`| `string`| Path where the downloaded file will be saved. Due to security reasons, it is only possible to provide two download paths: the Windows "Temp" or "Downloads" folder. | `-` | All | None |
| `focus` | `boolean` | If `false`, the window won't be on focus when created. | `true` | All | All |
| `hasMoveAreas` | `boolean` | If `false`, the window can't be moved. | `true` | Flat and HTML | Flat |
| `hasSizeAreas` | `boolean` | If `false`, the window can't be resized by dragging its borders, maximizing, etc. | `true` | Flat and HTML | Flat |
| `height` | `number` | Window height (in pixels). | `400` | All | All |
| `hidden` | `boolean` | If `true`, the window will be started as a hidden window. | `false` | All | All |
| `historyNavigationEnabled` | `boolean` | If `true`, this will allow the users to navigate back (`CTRL + Left`) and forward (`CTRL + Right`) through the web page history. | `GLOBAL CONFIG` | All | None |
| `isChild` | `boolean` | If `true`, the window will open as a child window, sharing the lifetime and the environment of the opener. | `false` | All | None |
| `isCollapsed` | `boolean` | If `true`, the window will start collapsed. | `false` | All | None |
| `isSticky` | `boolean` | If `true`, the window will stick to other Glue42 Windows forming groups. | `true` | All | All |
| `left` | `number` | Distance of the top left window corner from the left edge of the screen (in pixels). | `0` | All | All |
| `maxHeight` | `number` | Specifies the maximum window height (in pixels). | `-` | All | All |
| `maxWidth` | `number` | Specifies the maximum window width (in pixels). | `-` | All | All |
| `minHeight` | `number` | Specifies the minimum window height (in pixels). | `30` | All | All |
| `minWidth` | `number` | Specifies the minimum window width (in pixels). | `50` | All | All |
| `mode` | `string` | The type of the Glue42 Window. Possible values are `"flat"`, `"tab"` and `"html"`. | `"flat"` | `-` | None |
| `moveAreaThickness` | `string` | How much of the window area is to be considered as a moving area (meaning you can move the window by grabbing it). The string value corresponds to the left, top, right and bottom borders. | `"0, 12, 0, 0"` | HTML | None |
| `moveAreaTopMargin` | `string` | Margin for the top window move area. The string value corresponds to the left, top, right and bottom borders of the move area. Setting `moveAreaTopMargin` to `"10, 0, 0, 10"` will take away 10 pixels from the left and the right side of the move area. | `"0, 0, 0, 0"` | HTML | None |
| `onTop` | `boolean` | If `true`, the window will appear on top of the z-order. | `false` | All | None |
| `relativeDirection` | `string` | Direction (`"bottom"`, `"top"`, `"left"`, `"right"`) of positioning the window relatively to the `relativeTo` window. Considered only if `relativeTo` is supplied. | `"right"` | All | None |
| `relativeTo` | `string` | The ID of the window that will be used to relatively position the new window. Can be combined with `relativeDirection`. | `-` | All | None |
| `showInTaskbar` | `boolean` | If `false`, the window won't appear on the Windows taskbar. | `true` | All | None |
| `showTitleBar` | `boolean` | Determines whether the window will have a title bar. | `true` | Flat | None |
| `sizeAreaThickness` | `string` | How much of the window area is to be considered as a sizing area (meaning you can resize the window using that area). The string value corresponds to the left, top, right and bottom borders. | `"5, 5, 5, 5"` | HTML | None |
| `snappingEdges` | `string` | Specifies the active Glue42 Window snapping edges. Possible values are: `"top"`, `"left"`, `"right"`, `"bottom"`, `"all"` or any combination of them (e.g., `"left, right"`). | `"all"` | All | None |
| `startLocation` | `string` | Specifies the start window location. Possible options are `"Automatic"` (The Glue42 Window decides where it will be positioned) and `"CenterScreen"`. | `"Automatic"` | All | None |
| `stickyFrameColor` | `string` | Specifies the Glue42 Window frame color. Accepts hex color as a string (`"#666666"`) or named HTML colors (`"red"`). | `"#5b8dc9"` | All | All |
| `stickyGroup` | `string` | If set, the Glue42 Window can only stick to windows that have the same group. | `"Any"` | All | None |
| `tabGroupId` | `string` | Specifies the tab group ID. If two or more tab windows are defined with the same ID, they will be hosted in the same tab window. | `"Automatic"` | Tab | None |
| `tabIndex` | `number` | Specifies the tab position index. Tab windows in the same tab group are ordered by their position index. Use negative index to make the tab active. | `-` | Tab | None |
| `tabSelected` | `boolean` | Whether the tab is selected. | `false` | Tab | None |
| `tabTitle` | `string` | Sets the tab title. | `""` | Tab | Tab |
| `tabToolTip` | `string` | Sets the tab tooltip. | `""` | Tab | Tab |
| `title` | `string` | Sets the window title. To work properly, there should be a `<title>` HTML tag in the page. | `-` | All | All |
| `top` | `number` | Distance of the top left window corner from the top edge of the screen (in pixels). | `0` | All | All |
| `url` | `string` | The URL of the app to be loaded in the new window. | `-` | All | All |
| `useRandomFrameColor` | `boolean` | If `true`, this will set a random (from a predefined list of colors) frame color to the new window. | `false`| All | None |
| `width` | `number` | Window width (in pixels). | `400` | All | All |
| `windowName` | `string` | The name of the window. | `-` | All | None |
| `windowState` | `string` | If set, the window will start in the specified state (`"maximized"`, `"minimized"`, `"normal"`). | `"normal"` | All | All |