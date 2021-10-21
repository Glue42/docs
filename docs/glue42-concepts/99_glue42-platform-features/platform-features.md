## Running Multiple Instances

[**Glue42 Enterprise**](https://glue42.com/enterprise/) supports running multiple instances in different environments/regions and any combination of these. Environments usually include development, testing, quality assurance, production environments in which [**Glue42 Enterprise**](https://glue42.com/enterprise/) is developed, tested or integrated. Regions can be any semantic groups, defined by the clients - geographic regions, user groups, product categories, etc. This allows running multiple instances of [**Glue42 Enterprise**](https://glue42.com/enterprise/) simultaneously on a single machine with different settings and configurations which is useful when testing application integration in various environments. Environment/region can be set in the `system.json` configuration file located in `%LocalAppData%\Tick42\GlueDesktop\config`. You can run multiple instances of [**Glue42 Enterprise**](https://glue42.com/enterprise/) by defining different system configuration files for the respective environments/regions.

## Service Windows

Service windows are hidden windows which perform a specific supporting role for your applications. They can be configured as any normal window (name, URL, etc.), the difference being that a UI configuration is not necessary as it is assumed that the purpose of these windows is to provide some "behind-the-scenes" (hidden) service to your applications. Therefore, the user doesn't need to see them or interact with them.

Service windows may be useful in many scenarios. For instance, you may have a number of applications that need to receive and process data from several different providers. Instead of setting up each application to receive and then process the data from every provider, you can create a hidden service window which will communicate with the providers, collect the data, pre-process it and route it to the respective applications. This way, your applications have to handle communication with only one end point, all the necessary data is consolidated, processed, filtered, etc., at one central data hub from where it can be sent to any window needing it. Depending on your needs or goals, you can configure your service windows to auto start on system startup, or to start when an application requests that service. The service windows approach offers you additional flexibility and versatility in designing solutions for the application services you need.

![Service Window](../../images/platform-features/service-windows.png)

## Citrix Applications

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

[**Glue42 Enterprise**](https://glue42.com/enterprise/) provides *experimental* support for Citrix Virtual Apps. Citrix applications can participate in the Glue42 environment as first-class citizens - they can be configured and added to the Glue42 Toolbar, saved in Layouts and Workspaces, and can use all Glue42 functionalities like Interop, Channels, etc.

For more details on configuring a Citrix application, see the [Application Configuration](../../developers/configuration/application/index.html#application_configuration-citrix_app) section. For details on configuring the system-wide Citrix Virtual Apps support, see the [System Configuration](../../developers/configuration/system/index.html#citrix_apps) section.

*Note that this feature is experimental – although it has been properly tested, additional tests and adjustments might be necessary for your specific Citrix environment.*

### .NET Citrix Apps

To Glue42 enable a .NET Citrix app:

1. In your Visual Studio project, reference the `Glue42.dll` available in the [Glue42 NuGet package](https://www.nuget.org/packages/Glue42/).

2. Follow the standard procedure for [Glue42 enabling .NET applications](../../getting-started/how-to/glue42-enable-your-app/net/index.html). 

3. After initializing Glue42, you can check whether your application is connected to Glue42 in the following manner: 

```csharp 
using Tick42.StartingContext; 
if (InitializeOptions.IsCitrixGD) 
{
    // Running in Citrix, connected to Glue42 Enterprise. 
} 
else if (InitializeOptions.IsCitrixVirtualApp) 
{
    // Running in Citrix, not connected to Glue42 Enterprise.
}
```

When your Citrix app is connected to [**Glue42 Enterprise**](https://glue42.com/enterprise/), you may want to remove any custom window decorations, since the top-level window chrome will be handled by [**Glue42 Enterprise**](https://glue42.com/enterprise/). 

4. Add `%**` to the application arguments in the Citrix Application Settings:

![Citrix Application Settings](../../images/citrix/citrix-settings-net.png)

5. Configure your application as a [Glue42 Citrix app](../../developers/configuration/application/index.html#application_configuration-citrix_app).

You will now be able to run your .NET Citrix application from [**Glue42 Enterprise**](https://glue42.com/enterprise/) and interoperate with it using the various Glue42 APIs.

### Java Citrix Apps

To Glue42 enable a Java Citrix app:

1. Follow the standard procedure for [Glue42 enabling Java applications](../../getting-started/how-to/glue42-enable-your-app/java/index.html).  

2. In the Citrix Application Settings, set the path to a `javaw.exe` or `java.exe` file, use standard VM arguments to launch your Java app, and add `%**` at the end of the application arguments:

![Citrix Application Settings](../../images/citrix/citrix-settings-java.png)

3. Configure your application as a [Glue42 Citrix app](../../developers/configuration/application/index.html#application_configuration-citrix_app).

You will now be able to run your Java Citrix application from [**Glue42 Enterprise**](https://glue42.com/enterprise/) and interoperate with it using the various Glue42 APIs.

## Splash Screen

[**Glue42 Enterprise**](https://glue42.com/enterprise/) has a built-in splash screen, but also supports showing a custom splash screen. Your custom splash screen is loaded from a local file. 

**Configuration**

You can replace the splash screen HTML file in `%LocalAppData%\Tick42\GlueDesktop\assets\splash` with your own custom file. You can also set the size of the splash screen - the splash screen configuration can be set in the `system.json` file under the `"splash"` key:

```json
"splash": {
        "width": 350,
        "height": 233
    }
```

![Splash](../../images/platform-features/splash.png)

For the splash screen setup to work, you need to handle the following events: 

```javascript
// using `ipcMain`

const { ipcRenderer, remote } = require("electron");

// updateStatus event
ipcRenderer.on("updateStatus", (event, arg) => {
    console.log(`updating status to ${arg.text}`);
    var status = document.getElementById("status");    
    status.innerHTML = arg.text + "...";
});

// setVersion event
ipcRenderer.on("setVersion", (event, arg) => {
    var status = document.getElementById("version");
    status.innerHTML = arg.text;
});

// setEdition event
ipcRenderer.on("setEdition", (event, arg) => {
    var edition = document.getElementById("version");
    edition.innerHTML += ` (${arg.text})`;
});

// setEnvRegion event
ipcRenderer.on("setEnvRegion", (event, arg) => {
    var edition = document.getElementById("version");
    edition.innerHTML += ` -${arg.text}`;
});
```

## Global Protocol Handler

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

When you install [**Glue42 Enterprise**](https://glue42.com/enterprise/), it is registered as the default handler of the Glue42 global protocol. The protocol is in the following format:

```cmd
glue42://<option>/<identifier>[&args]
```

The Glue42 global protocol allows you to create and send links which will open a URL in a Glue42 Window. You can also create links that will start a Glue42 enabled app, load a specified Workspace or Layout and even invoke Interop methods with custom arguments.

When the link is clicked, [**Glue42 Enterprise**](https://glue42.com/enterprise/) will be started in order to handle it. If an instance of [**Glue42 Enterprise**](https://glue42.com/enterprise/) is already running, it will be reused. If multiple instances are running (e.g., in different environments), then the user will be presented with a dialog from which to choose the Glue42 instance that will handle the link:

![Protocol Dialog](../../images/platform-features/protocol-dialog.png)

### Configuration

The Glue42 global protocol can be configured from the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) using the `"potocolHandler"` top-level key:

```json
"protocolHandler": { 
    "enabled": true,
    "allowOpeningURLs": { 
        "allowed": ["https://glue42.com"], 
        "forbidden": ["https://youtube.com/.*", "https://facebook.com/.*"]
    }
}
```

The `"protocolHandler"` key has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"enabled"` | `bolean` | If `true` (default), will enable the Glue42 global protocol handler. |
| `"defaultExecutable"` | `string` | Path to an executable file that will be started if no other Glue42 instance is running. Defaults to the Glue42 executable file. |
| `"allowOpeningURLs"` | `boolean` \| `object` | Specify allowed and forbidden URLs for the Glue42 global protocol handler. Can be set to a `boolean` value or to an `object` containing a list of allowed or forbidden URLs. If `true`, will allow handling of all URLs. |

The `"allowOpeningURLs"` property can be set to an object with the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"allowed"` | `string[]` | List of allowed URLs. All other URLs will be forbidden. |
| `"forbidden"` | `string[]` | List of forbidden URLs. All other URLs will be allowed. |

You can use exact URL values or regular expressions to specify allowed and forbidden URLs.

### Protocol Options

The Glue42 global protocol can be used in different formats depending on what you want to do.

**Applications**

To start a Glue42 enabled application, use the `app` protocol option and pass the application name: 

```cmd
glue42://app/clientlist
```

To pass startup options for a Glue42 enabled application, use `&` before each setting. The following example demonstrates passing a location and context for the started app:

```cmd
glue42://app/clientlist&left=100&context.clientID=1
```

*To specify a property of an object as an option, use the standard dot notation - e.g., `&context.clientID=42`.*

**Layouts**

To restore a Global Layout, use the `layout` protocol option and pass the name of the Layout:

```cmd
glue42://layout/StartOfDay
```

**Workspaces**

To open a Workspace, use the `workspace` protocol option and pass the Workspace name:

```cmd
glue42://workspace/StartOfDay
```

To pass context for the Workspace, use `&context`:

```cmd
glue42://workspace/StartOfDay&context.clientID=1
```

*To specify a property of an object as an option, use the standard dot notation - e.g., `&context.clientID=42`.*

**Glue42 Windows**

To open a URL in a Glue42 Window, use the `url` protocol option and pass the URL:

```cmd
glue42://url/https://google.com 
```

To specify [Glue42 Window settings](../../reference/glue/latest/windows/index.html#WindowSettings) when opening a URL, use `&&` after the URL and `&` before each setting. The following example demonstrates passing a location for the newly opened window: 

```cmd
glue42://url/https://google.com&&left=100&top=200
```

*To specify a property of an object as a setting, use the standard dot notation - e.g., `&downloadSettings.autoSave=false`.*

**Interop Methods**

To invoke an Interop method, use the `invoke` protocol option and pass the method name:

```cmd
glue42://invoke/Shutdown
```

To pass arguments and/or target when invoking an Interop method, use `&args` and `&target`:

```cmd
glue42://invoke/ShowClient&args.clientId=1&target=best
```

## Downloading Files

[**Glue42 Enterprise**](https://glue42.com/enterprise/) allows for files to be downloaded by clicking on a link in the web page or by invoking an [Interop](../data-sharing-between-apps/interop/javascript/index.html#method_invocation) method.

![Downloading Files](../../images/platform-features/download.png)

**Configuration**

The file download behavior is controlled by the system configuration. It can be alternatively overridden by the application configuration. The system download behavior configuration can be set in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) from the `"downloadSettings"` property of the `"windows"` top-level key:

System configuration example:

```json
"windows" {
    "downloadSettings": {
        "autoSave": true,
        "autoOpenPath": false,
        "autoOpenDownload": false,
        "enable": true,
        "enableDownloadBar": true,
        "path": "%DownloadsFolder%"
    }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `"autoSave"` | `boolean` | If `true`, will auto save the file (without asking the user where to save it). If `false`, a system save dialog will appear. |
| `"autoOpenPath"` | `boolean` | If `true`, will open the folder that contains the downloaded file after the download is completed. |
| `"autoOpenDownload"` | `boolean` | If `true`, will open the download file after the download is completed. |
| `"enable"` | `boolean` | If `true`, enables the window to download files. |
| `"enableDownloadBar"` | `boolean` | If `true`, a download bar tracking the progress will appear at the bottom of the window when downloading. If `false`, the download process will be invisible. |
| `"path"` | `string` | Path where the downloaded file will be saved. Due to security reasons, it is only possible to provide two download paths: the Windows "Temp" or "Downloads" folder. |

You can also override the default system download behavior in the application configuration JSON file:

```json
{
    "title": "Download Test Files",
    "type": "window",
    "name": "download-test-files",
    "details": {
        "url": "https://downloadtestfiles.com/",
        "top": 100,
        "left": 400,
        "width": 800,
        "height": 800,
        "allowCollapse": false,
        "startLocation": "center",
        "downloadSettings": {
            "autoSave": false
        }
    }
}
```

**Clicking on a Link in the Web Page**

When a user clicks on a download link in the website, the download will start and [**Glue42 Enterprise**](https://glue42.com/enterprise/) will show the download bar in the page.

The user has options to:

- cancel the download; 
- pause and later resume the download; 
- open the downloaded file in the containing folder;

**Invoking an Interop Method**

On startup, [**Glue42 Enterprise**](https://glue42.com/enterprise/) registers an Interop method `"T42.Wnd.DownloadFile"` which you can [invoke](../data-sharing-between-apps/interop/javascript/index.html#method_invocation) with the following arguments in order to download files programmatically:

| Parameter | Type | Description |
|-----------|------|-------------|
| `url` | `string` | **Required**. Download link.|
| `name` | `string` | *Optional*. File name. |
| `path` | `string` | *Optional*. Location for the file. |
| `autoOpenDownload` | `boolean` | *Optional*. If `true`, will open the file after download. Defaults to `false`. |
| `autoOpenPath` | `boolean` | *Optional*. If `true`, will open the location of the file. Defaults to `false`. |

When downloading the selected file, the cookies for that domain are taken and sent together with the request.

*Note that if the file name already exists, the downloaded file will be saved with an incremented number in the name (e.g., `my-docs(2).pdf`).*

**Using the Window Management API**

Download settings can also be specified using the [Window Management](../windows/window-management/javascript/index.html) API (for more details, see [Downloads](../windows/window-management/javascript/index.html#window_settings-downloads).

## Web App Search

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.11">

Search in web apps opened in Glue42 Windows just like in a browser with the `CTRL + F` command:

![Search](../../images/platform-features/search-document.gif)

Use `ENTER` and `SHIFT + ENTER` to scroll through the results. Click `ESC` to close the search bar. 

## Context Menu

[**Glue42 Enterprise**](https://glue42.com/enterprise/) has a right-click context menu available in all Glue42 apps for which it has been enabled. It offers standard cut/copy/paste actions, zoom and spelling controls: 

![Context menu](../../images/platform-features/context-menu.png)

Enable the context menu:

- globally for all apps, under the `"windows"` top-level key in the `system.json` file:

```json
{
    "windows": {
        ...
        "contextMenu": true
    }
}
```

- per application, under the `"details"` top-level key of the application configuration file:

```json
[
    {
       ...
        "details": {
            ...
            "contextMenuEnabled": true
        }
    }
]
```

## Hotkeys

The Hotkeys API allows applications to register key combinations and receive notifications when a key combination is pressed by the user irrespective of whether the application is on focus or not. Hotkeys is useful for web applications that do not have access to system resources and cannot register global shortcuts.

### Configuration

You can control the hotkeys behavior from the `system.json` file under the `"hotkeys"` top-level key. You can find the `system.json` in the `%LocalAppData%\Tick42\GlueDesktop\config` folder.

Hotkeys configuration example:

```json
{
    "hotkeys": {
        "enabled": true,
        "blacklist": ["appManager"],
        "reservedHotkeys": ["ctrl+c", "ctrl+p", "ctrl+s"]
    }
}
```

The hotkeys object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"enabled"` | `boolean` | If `true`, hotkeys will be enabled. |
| `"whitelist"` | `string[]` | List of applications that can register hotkeys. Any app not on the list won't be able to register hotkeys. |
| `"blacklist"` | `string[]` | List of applications that can't register hotkeys. Any app not on the list will be able to register hotkeys. |
| `"reservedHotkeys"` | `string[]` | List of reserved (system or other) hotkeys that can't be overridden by other applications. |

### Hotkeys API

The Hotkeys API is accessible through the [`glue.hotkeys`](../../reference/glue/latest/hotkeys/index.html) object. To register a hotkey, your application must be using a Glue42 JavaScript version newer than 4.3.5.

To register a hotkey, use the [`register()`](../../reference/glue/latest/hotkeys/index.html#API-register) method:

```javascript
// Define a hotkey object.
const hotkeyDefinition = {
    hotkey: "shift+alt+c",
    description: "Open Client Details"
};

// This function will be invoked when the hotkey is pressed.
const hotkeyHandler = () => {
    glue.appManager.application("Client Details").start();
};

// Register the hotkey.
await glue.hotkeys.register(hotkeyDefinition, hotkeyHandler);
```

To remove a hotkey, use the [`unregister()`](../../reference/glue/latest/hotkeys/index.html#API-unregister) and pass the value of the hotkey combination as an argument:

```javascript
await glue.hotkeys.unregister("shift+alt+c");
```

To remove all hotkeys registered by your app, use the [`unregisterAll()`](../../reference/glue/latest/hotkeys/index.html#API-unregisterAll) method:

```javascript
await glue.hotkeys.unregisterAll();
```

To check if your app has registered a specific hotkey, use the [`isRegistered()`](../../reference/glue/latest/hotkeys/index.html#API-isRegistered) method:

```javascript
// Returns a boolean value.
const isRegistered = glue.hotkeys.isRegistered("shift+alt+c");
```

### Hotkeys View

There is a utility view that allows you to see all hotkeys registered by different applications. You can open it from the [**Glue42 Enterprise**](https://glue42.com/enterprise/) tray icon menu - right-click on the tray icon to display the menu. When you click on the Hotkeys item you will see a list of the hotkeys registered by your app:

![Hotkeys](../../images/platform-features/hotkeys.gif)

For a complete list of the available Hotkeys API methods and properties, see the [Hotkeys API Reference Documentation](../../reference/glue/latest/hotkeys/index.html).

## Zooming

[**Glue42 Enterprise**](https://glue42.com/enterprise/) supports zooming in and out of windows of JavaScript applications. Zooming can be controlled via configuration (system-wide or per application) or programmatically via the available methods/properties of a window instance.

You can zoom in and out of windows in several ways:

- `CTRL` + `=/-`;
- `CTRL` + mouse scroll;
- `CTRL` + `0` - resets to the default zoom factor;
- mouse pad gestures;
- using the right-click context menu (if enabled);

*Note that zooming is based on domain - i.e., if you open two applications with the same domain and change the zoom factor of one of them, the zoom factor of the other will change accordingly.*

![Zooming](../../images/platform-features/zooming.gif)

### Configuration

You can configure window zooming system-wide from the `system.json` file in the `%LocalAppData%\Tick42\GlueDesktop\config` folder. Use the `"zoom"` property under the `"windows"` top-level key:

```json
"windows": {
    "zoom": {
        "enabled": true,
        "mouseWheelZoom": true,
        "factors": [25, 33, 50, 67, 75, 80, 90, 100, 110, 125, 150, 175, 200, 250, 300, 400, 500],
        "defaultFactor": 100
    }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `"enabled"` | `boolean` | If `true`, zooming will be enabled. |
| `"mouseWheelZoom"` | `boolean` | If `true`, will enable zooming with `CTRL` + `mouse scroll`. |
| `"factors"` | `number[]` | List of zoom factors to be used when the user zooms in or out of the window. The factors must be in ascending order and may have integer or floating point values. Zooming will only work with factor values within the range of 25 to 500. Avoid passing negative values when setting the zoom factor (via configuration or programmatically), as this will cause unexpected behavior.|
| `"defaultFactor"` | `number` | Default zoom factor within the range of 25 to 500. Avoid negative values. |

You can also enable zooming per application which will override the system-wide zoom configuration. Use the `"zoom"` property under the `"details"` top-level key of the application configuration file:

```json
[
    {
        "title": "MyApp",
        "type": "window",
        "name": "myApp",
        "details": {
            "url": "http://localhost:22080/my-app/index.html",
            "zoom": {
                "enabled": true,
                "mouseWheelZoom": true,
                "factors": [25, 33, 50, 67, 75, 80, 90, 100, 110, 125, 150, 175, 200, 250, 300, 400, 500],
                "defaultFactor": 100
            }
        }
    }
]
```

### Using Zoom Programmatically

There are several methods and properties exposed on the window instance, which you can use to control zoom behavior.

Get the current zoom factor:

```javascript
const win = glue.windows.my();

console.log(win.zoomFactor);
```

Zoom in:

```javascript
// Will zoom in the window to the next factor in the "factors" array.
await win.zoomIn(); 
```

Zoom out:

```javascript
// Will zoom out the window to the previous factor in the "factors" array
await win.zoomOut();  
```

Set a desired zoom factor:

```javascript
await win.setZoomFactor(number); 
```

Carefully consider all cases if you intend to pass a zoom factor value based on a logic in your app. Negative values will cause unexpected behavior. Passing positive values lower than 25 will cause zoom out with a factor of 25, positive values higher than 500 will cause zoom in with a factor of 500 and passing zero as a factor will preserve the previous zoom factor. 

Listening for zoom factor changes:

```javascript
// Returns a function which you can use to unsubscribe from events for zoom factor changes.
const unsubscribe = win.onZoomFactorChanged(w => {
    console.log(`Zoom factor changed to ${w.zoomFactor}`)
});

unsubscribe();
```

## Displays

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

[**Glue42 Enterprise**](https://glue42.com/enterprise/) provides a way for applications to programmatically capture screenshots of the available monitors. Based on custom logic you can capture one or all monitors in order to save a snapshot of the visual state at a given moment.

The Displays API is accessible through the [`glue.displays`](../../reference/glue/latest/displays/index.html) object.

### Configuration

To enable display capturing you must add the `"allowCapture"` property to your application configuration file and set it to `true`.

```json
{
    "name": "MyApp",
    "type": "window",
    ...
    "allowCapture": true,
    "details": {
        "url": "http://localhost:3000"
        ...
    }
}
```

### Displays API

**All displays**

To get all displays, use the [`all()`](../../reference/glue/latest/displays/index.html#API-all) method. It returns an array of all available [`Display`](../../reference/glue/latest/displays/index.html#Display) objects:

```javascript
const allDsiplays = await glue.displays.all();
```

**Primary display**

You can get the primary display with the [`getPrimary()`](../../reference/glue/latest/displays/index.html#API-getPrimary) method:

```javascript
// Returns the primary display.
const primaryDisplay =  await glue.displays.getPrimary(); 
```

Example of finding and capturing the primary display:

```javascript
const display = await glue.displays.getPrimary();
const screenshot = await display.capture({ scale:0.5 });
```

**Specific display**

To get a specific display, use the [`get()`](../../reference/glue/latest/displays/index.html#API-get) method. It accepts a display ID as an argument and resolves with a [`Display`](../../reference/glue/latest/displays/index.html#Display) object:

```javascript
const displayID = 2528732444;

// returns a display by ID
const display = await glue.displays.get(displayID); 
```

**The Display object**

The [`Display`](../../reference/glue/latest/displays/index.html#Display) object has the following properties:

| Property | Description |
|----------|-------------|
| `id` | Unique identifier associated with the display. |
| `bounds` | A [`Bounds`](../../reference/glue/latest/displays/index.html#Bounds) object with `height`, `width`, `left` and `top` properties, describing the bounds of the display. |
| `workArea` | A [`Bounds`](../../reference/glue/latest/displays/index.html#Bounds) object describing the working area of the display (the desktop area, excluding taskbars, docked windows and toolbars). |
| `dpi` | Dots per inch resolution of the display. |
| `isPrimary` | A `boolean` value specifying whether this is the primary display. |
| `index` | Index assigned to the display by the operating system. |
| `name` | Name assigned to the display by the operating system. |
| `aspectRatio` | Display aspect ratio (e.g., 16:9). |
| `scaleFactor` | The scale factor of the returned display (e.g., 1.25 = 125%). |

**Capturing all displays**

To capture all displays, use the [`captureAll()`](../../reference/glue/latest/displays/index.html#API-captureAll) method. It accepts a [`CaptureAllOptions`](../../reference/glue/latest/displays/index.html#CaptureAllOptions) object and returns a `base64` encoded string or an array of `base64` encoded strings depending on the specified [`combined`](../../reference/glue/latest/displays/index.html#CaptureAllOptions-combined) option.

The following example demonstrates how to capture all available displays and combine the screenshots into a single image with `width` set to 2000 pixels. The aspect ratio of the combined images will be preserved (the omitted [`keepAspectRatio`](../../reference/glue/latest/displays/index.html#AbsoluteSizeOptions-keepAspectRatio) property in the `size` object defaults to `true`) and the images will be arranged the way you have arranged your displays from your operating system settings:

```javascript
const screenshot = await glue.displays.captureAll({ combined: true, size: { width: 2000 } });
```

The [`CaptureAllOptions`](../../reference/glue/latest/displays/index.html#CaptureAllOptions) object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `combined` | `boolean` | **Required**. If `true`, will return a single image of all captured displays. If `false`, will return separate images for all captured displays. |
| `size` | `object` | *Optional*. Accepts either a [`ScaleOptions`](../../reference/glue/latest/displays/index.html#ScaleOptions) or an [`AbsoluteSizeOptions`](../../reference/glue/latest/displays/index.html#AbsoluteSizeOptions) object, specifying the size of the output image. |

**Capturing a single display**

To capture a single display, use the `capture()` method at top level of the API or on a [`Display`](../../reference/glue/latest/displays/index.html#Display) instance. 

When you use the [`capture()`](../../reference/glue/latest/displays/index.html#API-capture) method at top level of the API, pass a [`CaptureOptions`](../../reference/glue/latest/displays/index.html#CaptureOptions) object. The following example demonstrates how to use the display ID to find and capture the desired display and also how to specify capturing options. The `width` and `height` of the output image will be half the width and height of the captured monitor. The captured image is returned as a `base64` encoded string: 

```javascript
const displayID = 2528732444;
const captureOptions = {
    id: displayID,
    size: { scale: 0.5 }
}

const screenshot = await glue.displays.capture(captureOptions);
```

The [`CaptureOptions`](../../reference/glue/latest/displays/index.html#CaptureOptions) object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `id` | `number` | **Required**. ID of the targeted display. |
| `size` | `object` | *Optional*. Accepts either a [`ScaleOptions`](../../reference/glue/latest/displays/index.html#ScaleOptions) or an [`AbsoluteSizeOptions`](../../reference/glue/latest/displays/index.html#AbsoluteSizeOptions) object, specifying the size of the output image. |

The [`ScaleOptions`](../../reference/glue/latest/displays/index.html#ScaleOptions) object has only one property - `scale`, which accepts a number. The value you pass to it specifies the size of the output image relative to the actual screen size. For instance, if you use `scale: 0.5` the height and width of the output image will be half the height and width of the captured screen. 

The [`AbsoluteSizeOptions`](../../reference/glue/latest/displays/index.html#AbsoluteSizeOptions) object has the following properties, all of which are optional:

| Property | Type | Description |
|----------|------|-------------|
| `width` | `number` | Specifies the width of the output image. Defaults to the captured display width. |
| `height` | `number` | Specifies the height of the output image. Defaults to the captured display height. |
| `keepAspectRatio` | `boolean` | Whether to keep the aspect ratio of the output image when you specify `width` and/or `height` of the output image. If `true` and both `width` and `height` are set, then the specified `width` will be used as a basis for the output image aspect ratio. |

When you use the [`capture()`](../../reference/glue/latest/displays/index.html#Display-capture) method of a [`Display`](../../reference/glue/latest/displays/index.html#Display) instance, pass either a [`ScaleOptions`](../../reference/glue/latest/displays/index.html#ScaleOptions) or an [`AbsoluteSizeOptions`](../../reference/glue/latest/displays/index.html#AbsoluteSizeOptions) object, specifying the size of the output image. The following example demonstrates how to find and capture all non-primary displays:

```javascript
const screenshots = await Promise.all(
            // Get all displays.
            (await glue.displays.all())
            // Filter out the primary display.
            .filter(display => !display.isPrimary)
            .map(display => display.capture({ scale: 0.5 })));
```

**Capturing windows and window groups**

You can use the [`capture()`](../../reference/glue/latest/windows/index.html#GDWindow-capture) method of a Glue42 Window instance or a Glue42 Window group to capture the respective window or window group. This method works also for minimized windows and window groups but doesn't work for hidden windows.

```javascript
// Capture the current window.
const windowScreenshot = await glue.windows.my().capture();

// Capture the current group.
const groupScreenshot = await glue.windows.groups.my.capture();
```

For a complete list of the available Displays API methods and properties, see the [Displays API Reference Documentation](../../reference/glue/latest/displays/index.html).

## Logging

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

[**Glue42 Enterprise**](https://glue42.com/enterprise/) offers a [Logger](../../reference/glue/latest/logger/index.html) API which enables JavaScript applications to create a hierarchy of sub-loggers mapped to application components where you can control the level of logging for each component. You can also route the output of log messages (depending on the logging level) to a variety of targets - the developer console or an external output (usually a rolling file on the desktop, but actually any target the `log4net` library supports).

### Logging to Files from Your JavaScript Application

Adding logging to files to your JavaScript apps can be helpful in a variety of ways. Having a well-designed and meaningful logging structure in your apps and their components can save a lot of time when debugging an app during development or troubleshooting problems with an app in production. 

*Logging to files for JavaScript applications is available from Glue42 JavaScript version 4.8.0 or later and [**Glue42 Enterprise**](https://glue42.com/enterprise/) 3.9 or later.*

### Logging Configuration

Logging for applications in [**Glue42 Enterprise**](https://glue42.com/enterprise/) is disabled by default. To allow it, add an `"allowLogging"` key to your application configuration file and set it to `true`:

```json
{
    "name": "my-app",
    ...
    "allowLogging": true,
    "details": {
        ...
    }
}
```

### Using a Logger

The Logger API is accessible through the [`glue.logger`](../../reference/glue/latest/logger/index.html) object.

Logger instances have a [`subLogger()`](../../reference/glue/latest/logger/index.html#API-subLogger) method that creates a new sub-logger of the current logger. The name of each logger instance is a dot delimited string containing all names of the loggers constituting an hierarchy line from the base logger (the application name) down to the current logger. This allows an effective and intuitive logging structure which can be easily adjusted to the component hierarchy in your app. For instance, a structure like `app-name.main-component.sub-component` gives you a clear idea from where the respective log entry originates and helps you find the necessary information much faster in a log file that may (and usually does) contain thousands of entries.

To use a logger in your Glue42 enabled applications, create a logger instance with the `subLogger()` method and assign the logger a name:

```javascript
const logger = glue.logger.subLogger("main-component");
```

Next, set the logging level at which to publish log entries to the file. Use the [`publishLevel()`](../../reference/glue/latest/logger/index.html#API-publishLevel) method of the logger instance:

```javascript
logger.publishLevel("info");
```

Everything at and above the specified logging level will be logged, all else will be skipped. The available logging levels are `"trace"`, `"debug"`, `"info"`, `"warn"` and `"error"`.

To log messages, either use the [`log()`](../../reference/glue/latest/logger/index.html#API-log) method of the logger instance, or the respective logging level methods - [`error()`](../../reference/glue/latest/logger/index.html#API-error), `trace()`, etc.

```javascript
// The log() method accepts a message and logging level as arguments.
logger.log("Could not load component!", "error");

// or

// Each logging level method accepts only a message as an argument.
logger.error("Could not load component!");
```

### Location and Output

User application log files are located in the `%LocalAppData%\Tick42\UserData\<ENV>-<REG>\logs\applications` folder, where `<ENV-REG>` should be replaced by the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). A separate log file is created for each application that has logging enabled. The file is named after the application and is created after the app starts to output log entries. All instances of an application log to the same file.

The log file entries are in the following format:

```cmd
[<DATE>T<TIME>] [<LOGGING_LEVEL>] [<INSTANCE_ID>] [<LOGGER_NAME>] - <LOG_MESAGE>
```

Here is how an actual entry in a log file looks like:

```cmd
[2020-03-11T14:27:58.087] [ERROR] [30760_11] [client-list.test-logger] - test-error
    at t.error (http://localhost:22080/client-list/desktop-4.8.0.js:1:39269)
    at <anonymous>:1:8
```

For a complete list of the available Logger API methods and properties, see the [Logger API Reference Documentation](../../reference/glue/latest/logger/index.html).s

## Process Reuse

[**Glue42 Enterprise**](https://glue42.com/enterprise/) is an Electron based application and as such uses the built-in [process management](https://electronjs.org/docs/tutorial/application-architecture#main-and-renderer-processes) mechanisms of Electron - each window has a separate renderer process. This, however, may lead to a large memory footprint. The concept of process reuse helps you achieve balance between performance and consumption of system resources.

[**Glue42 Enterprise**](https://glue42.com/enterprise/) offers configurable grouping of application instances in a single process. Process reuse can be configured both on a system from the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) or on an application level from the respective application configuration file. 

Below are described some basic use-cases and the configurations needed for them. Use the [Applications View](../../getting-started/how-to/use-glue42/index.html#applications_view) to monitor how applications are grouped in the processes:

- If you want every instance of your app to go into a dedicated process, go to the `"details"` top-level key of your application configuration and set the `"dedicatedProcess"` property of the `"processAffinity"` key to `true`:

```json
"details": {
    ...
    "processAffinity": {
        "dedicatedProcess": true
    }
}
```

![Process Reuse](../../images/platform-features/dedicated-process.png)

- If you want to group all instances of your application into a single process, assign a unique string value to the `"affinity"` property of the `"processAffinity"` key in your application configuration file:

```json
"details": {
    ...
    "processAffinity": {
        "affinity": "XYZ"
    }
}
```

*Note that the number of instances that can be grouped into a single process is restricted by the value of the `"maxAppsInProcess"` property in the global process reuse configuration in the `system.json` file. Set it to a high number (e.g. `"maxAppsInProcess": 1000`) in order to be able to group all instances of your app in a single process with the `"affinity"` property.*

![Process Reuse](../../images/platform-features/same-app-affinity.png)

- If you want to group several apps into a single process, assign the same string value to the `"affinity"` property of the `"processAffinity"` key in the application configuration file of each app:

Client List app:

```json
"name": "clientlist",
...
"details": {
    ...
    "processAffinity": {
        "affinity": "XYZ"
    }
}
```

Portfolio app:

```json
"name": "portfolio",
...
"details": {
    ...
    "processAffinity": {
        "affinity": "XYZ"
    }
}
```

*Again, consider setting the global process reuse configuration in the `system.json` file in order to achieve the desired results when grouping applications and application instances in a process. The `"maxAppsInProcess"` defines the maximum number of (any) instances that can go in the process and the `appSlotSize` property defines how many slots in that process are reserved for instances of the same application.*

![Process Reuse](../../images/platform-features/two-apps-affinity.png)

- If you want to increase the number of apps in a single process, use the `"maxAppsInPocess"` property of the global process reuse configuration in the `system.json` file:

```json
"processReuse": {
        "enabled": true,
        "visibleApps": {
            "maxAppsInProcess": 50,
            "appSlotSize": 2
        },
        "hiddenApps": {
            "maxAppsInProcess": 20,
            "appSlotSize": 1
        }
    }
```

### System Configuration

Configure process reuse on a system level for both visible and hidden apps using the `"processReuse"` top-level key of the `system.json` file:

```json
"processReuse": {
    "enabled": true,
    "visibleApps": {
        "maxAppsInProcess": 12,
        "appSlotSize": 3
    },
    "hiddenApps": {
        "maxAppsInProcess": 20,
        "appSlotSize": 1
    }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `"enabled"` | `boolean` | If `true`, will enable process reuse. |
| `"maxAppsInProcess"` | `number` | The maximum number of application instances in a process (maximum slots in a process). |
| `"appSlotSize"` | `number` | Sets the number of slots reserved for instances of the same application within a process. The total number of slots in the process is defined by the `"maxAppsInProcess"`. |

If you want more apps to use the same process, increase the `"maxAppsInProcess"` value or decrease the `"appSlotSize"` value. If you set `"maxAppsInProcess"` to 1, then all apps and their instances will be in separate processes.

The settings for visible apps in the example above (`{ "maxAppsInProcess": 12, "appSlotSize": 3 }`) show that a maximum of 12 slots per process will be available and that 3 slots will be reserved for instances of the same app in that process. This translates to the following:

- If you start four instances of different apps and then start more instances of any of these apps, the fourth instance of any of the apps will be in another process, because there are only three slots per process available for instances of the same app:

![Process Reuse](../../images/platform-features/process-slots.png)

- If you start two different app instances and then start only instances of any of these two apps, their instances will run in the same process until the remaining two sets of three slots in the process are filled too:

![Process Reuse](../../images/platform-features/slots-fill.png)

### Application Configuration

To set the process reuse behavior on an application level, use the `"processAffinity"` property under the `"details"` top-level key in the application configuration file: 

```json
"details": {
    ...
    "processAffinity": {
        "dedicatedProcess": false,
        "affinity": "XYZ"
    }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `"dedicatedProcess"` | `boolean` | If `true`, each instance of the app will be started in a separate dedicated process. This property is useful if you have an important application and you want each instance of it to start in a dedicated process. |
| `"affinity"` | `string` | All apps with the same affinity values will be grouped in the same process until the maximum number of slots (defined globally in the `"maxAppsInProcess"` property) in the process is reached. When `"affinity"` is set, the global `"appSlotSize"` property is ignored. This means that you can group different app instances with the same `"affinity"` however you like within the limits of the `"maxAppsInProcess"` property. Use this property if you want to group several apps in a process. |

## Flash Plugin

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

By default, Flash plugins are disabled in Glue42 Windows. To enable Flash plugins, add the respective configuration in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) under the `"flash"` top-level key.

To enable the default Flash plugin that comes with [**Glue42 Enterprise**](https://glue42.com/enterprise/):

```json
{
    ...
    "flash": true,
    ...
}
```

To enable a specific Flash plugin, provide its path:

```json
{
    ...
    "flash": "C:\\Users\\user\\AppData\\Local\\Tick42\\GlueDesktop\\assets\\flash\\32.0.0.330\\pepflashplayer.dll",
    ...
}
```

## Issue Reporting

[**Glue42 Enterprise**](https://glue42.com/enterprise/) has a built-in feedback form that allows users to send feedback with improvement suggestions or bug reports. The user can describe the problem/suggestion in the "Description" field and can optionally attach logs/configs to the report. The form can be configured to send an email with the report to our team and/or to automatically create a Jira ticket with the issue reported by the user. Both on-premise and cloud based Jira solutions are supported.

The feedback form is an HTML app, which can be re-designed and re-configured to suit specific client needs and requirements. For more details, see the [system configuration schema](../../assets/configuration/system.json).

![Feedback form](../../images/platform-features/feedback-form.png)

## Adding DevTools Extensions

You can extend the Chrome DevTools in [**Glue42 Enterprise**](https://glue42.com/enterprise/) with additional extensions. To add a [DevTools Extension supported by Electron](https://electronjs.org/docs/tutorial/devtools-extension#supported-devtools-extensions), you need to have the extension installed and add a configuration for it in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) and in the configuration file of your application. The example below demonstrates adding the React DevTools Extension to [**Glue42 Enterprise**](https://glue42.com/enterprise/):

1. Install the [React DevTools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi) Chrome extension.

2. Locate the extension on your local machine - the default location for the React DevTools Extension is `%LocalAppData%\Google\Chrome\User Data\Default\Extensions\fmkadmapgofadopljbjfkapdkoienihi`. (You can move the extension installation folder wherever you like.)

3. Open the `system.json` configuration file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config` and add the path to the React DevTools Extension under the `"devToolsExtensions"` top-level array:

```json
{
    "devToolsExtensions": [
        "C:\\Users\\<username>\\AppData\\Local\\Google\\Chrome\\User Data\\Default\\Extensions\\fmkadmapgofadopljbjfkapdkoienihi"
    ]
}
```

*Replace `<username>` with your local username. Remember to escape the backslash characters.*

4. Open the JSON configuration file of your application and add the following configuration under the `"details"` top-level key:

```json
"security": {
    "sandbox": false
}
```

For instance:

```json
{
    "name": "My App",
    ...
    "details": {
        "url": "http://localhost:3000",
        ...
        "security": {
            "sandbox": false
        }
    }
}
```

5. Start [**Glue42 Enterprise**](https://glue42.com/enterprise/), open your [Glue42 enabled](../../getting-started/how-to/glue42-enable-your-app/javascript/index.html) app and you should be able to see the added extension to the Chrome DevTools when you open the developer console. In this case, you will need a React app in order to be able to see the React DevTools Extension.

## Extensible Installer Guide

The [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer supports extensibility mode, meaning you can repackage the installer with custom resources and installation settings. Some of the customizable features include replacing the icons, the installation screen, adding/removing or setting default install options or features of Glue42 Enterprise, adding other programs to be installed together with [**Glue42 Enterprise**](https://glue42.com/enterprise/).

### Extensibility Mode

The [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer can be run with an argument `/ext:path\to\extensibility-file.json`, which defines the path to a file that allows the operation of the installer to be controlled.

The installer defines a number of extensibility points, representing stages of the installation flow, each of which can be populated with one or more extensibility items, representing tasks to perform or settings to change. These items are listed in the extensibility file with the following structure:

```json
{
    "<extensibility point 1>": [
      { "type": "<extensibility item type>", "args": { "<arg>": <value>, ... }},
      { "type": "<extensibility item type>", "args": { "<arg>": <value>, ... }},
    ],
    "<extensibility point 2>": [
      { "type": "<extensibility item type>", "args": { "<arg>": <value>, ... }},
      { "type": "<extensibility item type>", "args": { "<arg>": <value>, ... }},
    ],
}
```
In case of no arguments, an extensibility item can be shortened to just a string with its type: `{ "type": "unattended", "args": {} }` is shortened to `"unattended"`.

Here is an example extensibility file content, listing most of the available extensibility points and items:

*Note that this is just for illustrative purposes. Some of the following settings don't make sense together.*

```json
{
    "startup": [
        // uninstall Glue42 Enterprise
        "uninstall",

        // unattended installation
        "unattended",

        // NB: the installation cannot run while certain applications are running,
        // e.g. a previous installation of Glue42 Enterprise, or Excel/Word/Outlook
        // (unless the corresponding plugin is disabled in the artifacts extensibility point)

        // By default, an unattended installer will exit with a non-zero exit code,
        // but you can make it retry by adding the following arguments:

        // - pop a message box for the user to dismiss (NB: this might cause
        // the installation to stall if there is no user present)
        // "args": { "conflictHandling": "ask" }

        // - retry 10 times, with an interval of 1 second
        // "args": { "conflictHandling": "retry", "waitMs": 1000, "retries": 10 }

        // hidden installation: similar to "unattended", but without showing a window
        "hidden",

        // use a predefined license file
        {
            "type": "license",
            "args": {
                // either path or url
                "file": "license.json",
                "url": "https://example.com/license.json"
            }
        },

        // logo to display in top-left corner
        {
            "type": "logo",
            "args": {
                // either path or url
                "file": "logo.png",
                "url": "https://example.com/logo.png",
                "onClick": "https://example.com/example"
            }
        },

        // large banner during installation
        {
            "type": "banner",
            "args": {
                // either path or url
                "file": "banner.png",
                "url": "https://example.com/banner.png",
                "onClick": "https://example.com/example"
            }
        }
    ],
    "artifacts": [

        // disable GlueXL artifact
        {
            "type": "GlueXL",
            "args": {
                "remove": true
            }
        },

        // make GlueOutlook not selected by default;
        // in unattended installations, this is the same as "remove"
        {
            "type": "GlueOutlook",
            "args": {
                "checkedByDefault": false
            }
        },

        // make GlueWord required
        {
            "type": "GlueWord",
            "args": {
                "required": true
            }
        },
    ],

    // welcome screen
    "welcome": [
        {
            "type": "run",
            "args": {
                "filename": "cmd.exe",
                "args": "/c somebatchfile.bat",
                // by default, exit code 0 is success,
                // while any other means error message, followed by installer exiting
                // you can override with "success" for success, any other string for error message
                // (error messages are not shown in unattended installation to avoid stalling)
                "exitCode1": "There was an error validating your installation",
                "exitCode2": "There was an error contacting server",
                "exitCode3": "success",
                //other exit codes

                // whether to hide the started process
                "hide": true,

                // whether to hide the installer while the process is running
                "hideInstaller": false
            }
        }
    ],

    // ... other screens: downloadProgress, packages, previewAndConfirm, uninstall, ...

    // package the Glue42 Enterprise installer with custom configuration files
    "finalizing": [ 
        {
            "type": "gdConfig",
            "args": {
                // archive with custom config files for Glue42 Enterprise
                "file": "custom-config.zip",
                // If `false` (default), will merge the custom config files with the default ones from the installer
                // by replacing any default file with the respective custom config file with the same name.
                // If `true`, the default config files will be deleted and replaced by the specified custom config files.
                // This means that you must provide all required configuration files for Glue42 Enterprise to function properly.
                "wipe": false
            }
        }
    ],

    // final screen
    "done": [
        // show Glue42 Enterprise shortcut location - like the current "Launch" button behavior
        "showGD",

        // launch the Glue42 Enterprise executable - like the old "Launch" button behavior
        "launchGD",

        // shows a message box
        {
            "type": "message",
            "args": {
                "text": "Don't forget to be awesome!",
                "title": "Reminder"
            }
        },

        // this is a good place to use a "run" item if something else needs to
        // happen after the installer is finished, e.g., run another installer

        // exit from final screen without user having to click "Done"
        {
            "type": "exit",
            "args": {
                "exitCode": 0
            }
        }
    ]
}
```

Some types of extensibility items are only applicable to some extensibility points, e.g., `"unattended"` (which enables an unattended installation) is only applicable to the `"startup"` extensibility point. Others can be used anywhere - such as `"run"`, which executes an external program and waits for it to finish; or `"message"`, which shows a message to the user.

Extensibility items are executed in the order they are specified. Multiple items of the same type can be present in the same extensibility point.

### Step-By-Step Example

Download an [extensible installer example](../../assets/sfx-installer-example.zip) to get you started. Use it to test how the extensible installer works, to tweak the setting, add your own files and, ultimately, create your own extensible installer.

In this step-by-step guide you will be creating a custom [**Glue42 Enterprise**](https://glue42.com/enterprise/) extensible installer with the following additions and changes:

- adding custom logos;

- installing [**Glue42 Enterprise**](https://glue42.com/enterprise/) with custom `system.json` properties, in order to setup [**Glue42 Enterprise**](https://glue42.com/enterprise/) to retrieve application configuration settings from a REST service;

It is assumed that you have your icon and install screen banner files ready, and also - you have set up your REST service and have it up and running.

*Note that if you have your own [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer file, then rename your installer to `GlueInstaller.exe` and in the `installer-and-other-files` folder replace the existing `GlueInstaller.exe` file with your own installer file.*

1. Navigate to the directory where you have downloaded the `sfx-installer-example.zip` and extract the files in it.

2. Open the `installer-and-other-files` folder and replace the provided `logo.png` and `banner.png` example files with your own logo and banner files.

3. Open the `example-extensibility.json` in your preferred text editor and under the `startup` top key array edit the `arguments` object of the respective extensibility items for the installation banner and logo:

- replace the `"file"` property values with the respective names of your logo and banner files;
- if you want clicking on the logo or the banner during installation to open your site, replace the URL in the `"onClick"` property with a link to your site. Otherwise, remove it altogether;

```json
"startup": [
    {
        "type": "logo",
        "args": {
            "file": "MyCustomLogo.png",
            "onClick": "https://www.my-site.com"
        }
    },
    {
        "type": "banner",
        "args": {
            "file": "MyCustomBanner.png",
            "onClick": "https://my-site.com"
        }
    }
]
```

4. The provided `system.json` file in `sfx-installer-example.zip` may not be the latest one, so you have to replace it with the `system.json` file from your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy located in `%LocalAppData%\Tick42\GlueDesktop\config`. Open the `system.json` file in your text editor and add the following configuration in order to enable [**Glue42 Enterprise**](https://glue42.com/enterprise/) to retrieve the application configuration settings directly from a REST service:

```json
"appStores": [
    {
        "type": "rest",
        "details": {
            // URL to your remote application store
            "url": "http://localhost:3000/appd/v1/apps/search",
            "auth": "no-auth",
            "pollInterval": 30000,
            "enablePersistentCache": true,
            "cacheFolder": "%LocalAppData%/Tick42/UserData/%GLUE-ENV%-%GLUE-REGION%/gcsCache/"
        }
    }
]
```

5. Go back to the folder where you extracted the files and double click on the `produce-sfx-installer.bat` file to produce the installer file. The output installer file is named `sfx-installer.exe`.

6. Run `sfx-installer.exe` to install the product.

*Note that the SFX installer should be Authenticode signed to avoid antivirus software raising false alerts. Also, to change its icon or file properties, you can use a resource editor, e.g. Resource Hacker. It is best to use a multi-size icon to support various resolutions and Windows Explorer view modes. The example used the Greenfish icon editor for this purpose.*