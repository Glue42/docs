## Extensible Installer

Most tasks for customizing [**Glue42 Enterprise**](https://glue42.com/enterprise/) can be accomplished by using the extensibility features of the Glue42 installer application. Whether it's changing the interface and behavior of the installer, modifying Glue42 components, substituting default with custom configurations, creating shortcuts, or adding custom resources like icons, splash screen, loaders and more, the installer application offers ways for automating almost all customization processes.

The basic premise of the installer extensibility mode is to repackage the stock Glue42 installer application into a new executable file together with your custom configurations and resources. To accomplish this, you must use the provided [basic setup for creating a repackaged installer](https://enterprise.glue42.com/upload/docs/sfx-installer-example.zip). Some customizations can be achieved by modifying the [extensibility configuration file](#extensible_installer-extensibility_configuration_file) for the repackaged installer, while others require you to directly modify or replace resources in the provided basic setup.

*For descriptions of the files contained in the basic setup and details on how to use it, see the [Extensibility Installer Example](#extensible_installer_example) section.*

### Extensibility Configuration File

The extensibility features of the Glue42 installer are controlled by an extensibility configuration file. It defines a number of extensibility points, representing stages of the installation flow, each of which can be populated with one or more extensibility items, representing tasks to perform or settings to change. The extensibility points and items are listed in the extensibility file using the following structure:

```json
{
    "<extensibility point 1>": [
        {
            "type": "<extensibility item type>",
            "args": {
                "<arg>": <value>,
                ...
            }
        },
        ...
    ],
    "<extensibility point 2>": [
        {
            "type": "<extensibility item type>",
            "args": {
                "<arg>": <value>,
                ...
            }
        },
        ...
    ],
    ...
}
```

In case of no arguments, an extensibility item can be shortened to just a string with its type:

```json
{
    "startup": [
        {
            "type": "unattended",
            "args": {}
        }
    ]
}

// Can be shortened to:
{
    "startup": [
        "unattended"
    ]
}
```

All extensibility points are arrays. The table below lists all available extensibility points and the extensibility item types applicable to them:

| Extensibility Point | Extensibility Items |
|---------------------|---------------------|
| `"startup"` | Use the `"uninstall"`, `"unattended"` or `"hidden"` items to control the install process. Use the `"license"`, `"logo"` and `"banner"` items to specify a custom license file, custom logo and banner images. |
| `"styles"` | Use the `"color"` item to specify colors for various components of the installer UI. For more details, see the [Color Scheme](#installer_ui-color_scheme) section. |
| `"artifacts"` | Use the names of the Glue42 [Artifacts](#artifacts) as items to control which artifacts to remove or make required. |
| `"context"` | Use the `"setValue"` item to modify the installer screen titles, URLs, text, the product name, installation location and more. |
| `"shortcuts"` | This item allows you to modify or remove predefined shortcuts for Glue42 [Artifacts](#artifacts) and also create brand new ones. For more details, see the [Shortcuts](#shortcuts) section. |
| `"welcome"`, `"previewAndConfirm"`, `"downloadProgress"`, `"packages"`, `"uninstall"` | These extensibility points are screens of the installer which represent stages of the install or uninstall process. Use `"run"` items to run external programs, or `"message"` items to show messages to the user. |
| `"finalizing"` | Use the `"gdConfig"` item to supply custom configuration files for [**Glue42 Enterprise**](https://glue42.com/enterprise/). |
| `"done"` | This is the final installer screen. Use `"showGD"` and `"launchGD"` to open the folder containing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file or directly launch [**Glue42 Enterprise**](https://glue42.com/enterprise/). Use the `"exit"` item to exit the install process with a specified exit code. |

*For a basic extensibility file setup and an example on how to use it to modify the Glue42 installer, see the [Extensible Installer Example](#extensibile_installer_example) section.*

### Extensibility Points & Items

Each extensibility point represents a stage in the installation or the uninstallation process. Extensibility points are configured using the extensibility items applicable to them.

This section lists all available extensibility points and the extensibility items applicable to them.

*Note that the [`"run"`](#extensible_installer-extensibility_points__items-extensibility_item_run) and [`"message"`](#extensible_installer-extensibility_points__items-extensibility_item_message) extensibility items can be used in every extensibility point to run a custom script or show a message to the user.*

#### Extensibility Point: "startup"

The `"startup"` extensibility point accepts the following items:

- `"uninstall"`

Use to uninstall the product. The installer will start directly from the Uninstall screen.

- `"unattended"`

Provides a continuous, uninterrupted installation process from the moment the installer is started until the installation succeeds or fails. The user won't be asked to take any action. The installation process can't be completed while certain applications are running - e.g., [**Glue42 Enterprise**](https://glue42.com/enterprise/), or Excel/Word/Outlook, unless the corresponding [Glue42 Connector](../../../../connectors/general-overview/index.html) has been disabled in the `"artifacts"` extensibility point. By default, an unattended installation will exit with a non-zero exit code when it encounters conflicts, but you can specify options for conflict handling.

| Argument | Type | Description |
|----------|------|-------------|
| `"conflictHandling"` | `"ask"` \| `"retry"` | Specify how to handle conflicts during unattended or hidden installations. The possible values are `"retry"` - try to resume the installation automatically, and `"ask"` - display a message for the user to dismiss. Using `"ask"` may stall the installation if no user is present. |
| `"waitMs"` | `number` | Interval in milliseconds at which to attempt resuming the installation. |
| `"retries"` | `number` | The number of times to try to resume the installation. |

*See also the [Unattended Installation](#unattended_installation) section.*

- `"hidden"`

Provides the same functionality as `"unattended"`, except that the installer is hidden.

- `"license"`

Use this item to specify a custom license file.

| Argument | Type | Description |
|----------|------|-------------|
| `"file"` | `string` | Path to a license file. |
| `"url"` | `string` | URL pointing to a license file. |

- `"logo"`

Use this item to specify a custom logo for the installer.

| Argument | Type | Description |
|----------|------|-------------|
| `"file"` | `string` | Path to a file for the logo. |
| `"url"` | `string` | URL pointing to a file for the logo. |
| `"onClick"` | `string` | URL to open if the user clicks on the logo. Skip if you don't want to open a page when the user clicks on the logo. |

- `"banner"`

Use this item to specify a custom banner for the installer.

| Argument | Type | Description |
|----------|------|-------------|
| `"file"` | `string` | Path to a file for the banner. |
| `"url"` | `string` | URL pointing to a file for the banner. |
| `"onClick"` | `string` | URL to open if the user clicks on the banner. Skip if you don't want to open a page when the user clicks on the banner. |

The following is an example configuration for the `"startup"` extensibility point demonstrating how to supply custom license, logo and banner files:

```json
{
    "startup": [
        // Use a predefined license file.
        {
            "type": "license",
            "args": {
                // Either a path or a URL.
                "file": "license.json"
            }
        },

        // Logo to display in the top-left corner.
        {
            "type": "logo",
            "args": {
                // Either a path or a URL.
                "url": "https://example.com/logo.png",
                // URL to open when the user clicks on the logo.
                "onClick": "https://example.com"
            }
        },

        // Banner during installation.
        {
            "type": "banner",
            "args": {
                // Either a path or a URL.
                "file": "banner.png",
                // URL to open when the user clicks on the banner.
                "onClick": "https://example.com"
            }
        }
    ]
}
```

#### Extensibility Point: "styles"

The `"style"` extensibility point accepts the following items:

- `"color"`

Use this item to set the color for various components of the installer UI.

| Argument | Type | Description |
|----------|------|-------------|
| `"name"` | `string` | Name of the installer component. For a list of the available installer components, see the [Color Scheme](#installer_ui-color_scheme) section. |
| `"value"` | `string` | Color for the installer component as a hexadecimal number. |

*See also the [Color Scheme](#installer_ui-color_scheme) section.*

#### Extensibility Point: "context"

The `"context"` extensibility point accepts the following items:

- `"setValue"`

Use this item to set various miscellaneous values for installer screen titles, URLs, text, the product name, installation location and more.

| Argument | Type | Description |
|----------|------|-------------|
| `"name"` | `string` | Name of the installer component or setting. |
| `"value"` | `-` | Value for the specified installer component or setting. Can be any type. |

*For example usage of the `"context"` extensibility point, see the [Installer UI](#installer_ui) section.*

#### Extensibility Point: "artifacts"

The `"artifacts"` extensibility point accepts the names of the [**Glue42 Enterprise**](https://glue42.com/enterprise/) Artifacts as item types. Each item type can have the following arguments:

| Argument | Type | Description |
|----------|------|-------------|
| `"required"` | `boolean` | Whether to make the specified Glue42 Artifact required for the installation. |
| `"checkedByDefault"` | `boolean` | Whether to select the specified Glue42 Artifact for installation by default. |
| `"remove"` | `boolean` | Whether to remove the specified Glue42 Artifact from the installation. |

The following is an example configuration for the `"artifacts"` extensibility point demonstrating how to disable the installation of the Glue42 [Excel Connector](../../../../connectors/ms-office/excel-connector/javascript/index.html):

```json
{
    "artifacts": [
        {
            "type": "GlueXL",
            "args": {
                "remove": true
            }
        }
    ]
}
```

*For a list of the available Glue42 Artifacts, see the [Artifacts](#artifacts) section.*

#### Extensibility Point: "shortcuts"

The `"shortcuts"` extensibility point accepts the following items:

- `"shortcut"`

Use this item to modify or remove predefined shortcuts for Glue42 [Artifacts](#artifacts) and also create brand new ones.

| Argument | Type | Description |
|----------|------|-------------|
| `"artifact"` | `string` | The name of the Glue42 Artifact for which a shortcut will be created. If the installation of the respective artifact has been disabled, shortcuts to it won't be created. For a list of the available Glue42 Artifacts, see the [Artifacts](#artifacts) section. |
| `"icon"` | `string` | Path to an icon for the shortcut. This is usually a custom file packaged as a resource in the installer. To instruct the installer to copy the packaged icon file to the installation folder of the specified artifact, use `copy:` before the path to the file - e.g., `copy:artifact-icon.ico`. |
| `"iconIndex"` | `number` | Select an icon index in case a multi-icon package is specified in the `"icon"` argument. |
| `"filename"` | `string` | File name for the shortcut. |
| `"desktop"` | `boolean` | Whether to add a shortcut to the Windows Desktop folder. |
| `"startMenu"` | `boolean` | Whether to add a shortcut to the Windows Start Menu folder created during installation. See also the [Start Menu Settings](#start_menu_settings) section. |
| `"description"` | `string` | Description for the shortcut file. |
| `"target"` | `string` | URL or path to the file to which the shortcut refers. Some Glue42 Artifacts have default target files and it isn't necessary to specify them explicitly. |
| `"targetArguments"` | `string` | Optional custom command line arguments for the shortcut command. |
| `"workingDirectory"` | `string` | Optional working directory override for the shortcut command. |

*See also the [Shortcuts](#shortcuts) section.*

#### Extensibility Points: "welcome", "downloadProgress", "packages", "previewAndConfirm", "uninstall"

All these extensibility points represent screens of the installation process and you can use `"run"` items in them to execute external scripts, or `"message"` items to show messages to the user.

The following is an example configuration for the `"welcome"` extensibility point demonstrating how to run a script:

```json
{
    "welcome": [
        {
            // Depending on your purpose, you may run scripts here,
            // or after the installation, in the "done" extensibility point.
            "type": "run",
            "args": {
                // Name of the program to run.
                "filename": "cmd.exe",
                // Command line arguments for the program execution.
                "args": "/c copy-resources.bat",
                // By default, "exitCode0" is success, while any other is an error message, followed by the installer exiting.
                // You can override any code with "success" for success and any other string for an error message.
                // Note that error messages aren't shown during unattended or hidden installations to avoid stalling.
                "exitCode1": "Error validating the installation.",
                "exitCode2": "Error contacting server.",
                "exitCode3": "success",
                //Other exit codes.

                // Hide the started process.
                "hide": true,

                // Don't hide the installer while the process is running.
                "hideInstaller": false
            }
        }
    ]

    // Other screens: "downloadProgress", "packages", "previewAndConfirm", "uninstall".
}
```

#### Extensibility Point: "finalizing"

The `"finalizing"` extensibility point accepts the following items:

- `"gdConfig"`

Use this item to supply custom configuration files for [**Glue42 Enterprise**](https://glue42.com/enterprise/).

| Argument | Type | Description |
|----------|------|-------------|
| `"file"` | `string` | Path to an archive file with custom configuration files for [**Glue42 Enterprise**](https://glue42.com/enterprise/). |
| `"wipe"` | `boolean` | If `false` (default), will merge the custom configuration files with the default ones from the installer by replacing any default file with the respective custom file with the same name. If `true`, the default configuration files will be deleted and replaced with the custom ones. This means that you must provide all required configuration files for [**Glue42 Enterprise**](https://glue42.com/enterprise/) to function properly. |

*See also the [Overriding Configurations](#post_installation-overriding_configurations) section.*

#### Extensibility Point: "done"

The `"done"` extensibility point accepts the following items:

- `"launchGD"`

Use this item to automatically launch [**Glue42 Enterprise**](https://glue42.com/enterprise/) after the installation has completed.

*See also the [Launching](#post_installation-launching) section.*

- `"showGD"`

Use this item to automatically open the folder containing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file after the installation has completed.

*See also the [Launching](#post_installation-launching) section.*

- `"exit"`

Use this item to exit the installation process with a specified exit code.

| Argument | Type | Description |
|----------|------|-------------|
| `"exitCode"` | `number` | Exit code for the installation process. |

*See also the [Exiting the Installer](#post_installation-exiting_the_installer) section.*

#### Extensibility Item: "run"

Executes an external program and waits for it to finish.

| Argument | Type | Description |
|----------|------|-------------|
| `"filename"` | `string` | Path to the executable file of the program to run. |
| `"args"` | `string` | Command line arguments for the program. |
| `"exitCode<number>"` | `string` | Define custom messages for the exit codes returned from running the program. By default, `"exitCode0"` is success, while any other is an error message, followed by the installer exiting. You can override any code with "success" for success and any other string for an error message. Note that error messages aren't shown during unattended or hidden installations to avoid stalling. |
| `"hide"` | `boolean` | Whether to hide the started process. |
| `"hideInstaller"` | `boolean` | Whether to hide the installer while the process is running. |

#### Extensibility Item: "message"

Shows a message to the user.

| Argument | Type | Description |
|----------|------|-------------|
| `"text"` | `string` | Message text. |
| `"title"` | `string` | Title for the message window. |

### Online & Offline Installers

[**Glue42 Enterprise**](https://glue42.com/enterprise/) provides two types of installers: online and offline (or "embedded"). They are identical in functionality, but the online installer downloads the necessary Glue42 Artifacts from the Glue42 CDN, while the embedded installer contains them all as resources. The advantage of this is that the online installer is much smaller, but requires a connection to the Internet and permission to download files. If you encounter networking or permission issues, try the offline installer.

Both types of installer can be used interchangeably to create online and offline repackaged installers respectively. In fact, you could repackage both and let your deployment team or users decide which one to use.

## Installer UI

The following sections describe how to customize various elements of the installer UI. As explained in previous sections, some customizations can be achieved by modifying the [extensibility configuration file](#extensible_installer-extensibility_configuration_file) for the repackaged installer, while others require you to directly modify or replace resources in the provided [basic setup for creating a repackaged installer](https://enterprise.glue42.com/upload/docs/sfx-installer-example.zip).

*For descriptions of the files contained in the basic setup for creating a repackaged Glue42 installer and details on how to use it, see the [Extensible Installer Example](#extensibile_installer_example) section.*

### Icon

To change the icon of the repackaged installer, replace the provided icon file in the [extensible installer example](#extensible_installer_example) keeping the file name the same.

### Extraction Text

To change the title and text in the preliminary dialog of the repackaged installer, modify the `Title` and `BeginPrompt` arguments in the `config.txt` file of the [extensible installer example](#extensible_installer_example):

```cmd
Title="My Custom Installer"
BeginPrompt="Do you want to install My Custom Product?"
```

### Product Name

To change the product name displayed in the installer, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "ProductName",
                "value": "Custom Product"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

### Version

To add a custom suffix to the product version, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "VersionSuffix",
                "value": "custom-suffix"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

### Window Title

To change the title of the installer window, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "WindowName",
                "value": "My Custom Product Installer"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

### Screen Icon

To change the icon of the installer screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "Icon",
                "value": "icon.ico"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

### Logo

To change the logo in the top left of the installer screen, use the `"startup"` extensibility point:

```json
{
    "startup": [
        {
            "type": "logo",
            "args": {
                "file": "logo.png",
                "onClick": "https://example.com"
            }
        }
    ]
}
```

*See also [Extensibility Point: "startup"](#extensible_installer-extensibility_points__items-extensibility_point_startup).*

### Color Scheme

To change the color of an installer component, use the `"styles"` extensibility point. Pass the name of the component and the desired color a hexadecimal value. The supported installer components are:

- `"Background"`
- `"ButtonBackground"`
- `"Caret"`
- `"CheckboxDisabled"`
- `"ControlsDisabled"`
- `"DisabledColor"`
- `"Foreground"`
- `"Highlight"`
- `"Hyperlink"`
- `"ItemSelected"`
- `"LoginBoxBackground"`
- `"LoginBoxForeground"`
- `"TextBoxBackground"`
- `"TextBoxForeground"`
- `"TreeViewHover"`
- `"Warning"`

The following is an example configuration for the `"styles"` extensibility point demonstrating how to change the background and foreground colors for the installer screens:

```json
{
    "styles": [
        {
            "type": "color",
            "args": {
                "name": "Background",
                "value": "#000000"
            }
        },
        {
            "type": "color",
            "args": {
                "name": "Foreground",
                "value": "#ffffff"
            }
        }
    ]
}
```

*See also [Extensibility Point: "styles"](#extensible_installer-extensibility_points__items-extensibility_point_styles).*

### Welcome Screen Text

To change the text of the initial screen of the installer, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "WelcomeScreenParagraphs",
                "value": "My Custom Product"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

<!-- In the previous example, `$PRODUCT` will be replaced with the `ProductName` value as described in the [Product Name](#installer_ui-product_name) section. -->

<!-- To insert the [**Glue42 Enterprise**](https://glue42.com/enterprise/) version, set the following properties using the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "VersionReplacementPattern",
                // Regular expression to match in the Welcome screen text and replace with version information.
                "value": "(will install .*)"
            }
        },
        {
            "type": "setValue",
            "args": {
                "name": "VersionSubstitutionPattern",
                // Version information to put in the text. You can use capturing groups from "VersionReplacementPattern"
                // and the `$version` and `$versionShort` variables to create your own string.
                "value": "$1 version $version"
            }
        }
    ]
}
``` -->

### Custom Install Button

To disable the Custom Install button in the installer, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "AllowCustomInstall",
                "value": false
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

### Artifacts Screen

#### Title

To change the title displayed in the Artifacts screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "DownloadScreenTitle",
                "value": "Custom Screen Title"
            }
        }
    ]
}
```

#### Banner

To change the banner displayed in the Artifacts screen, use the `"startup"` extensibility point:

```json
{
    "startup": [
        {
            "type": "banner",
            "args": {
                "file": "banner.png",
                "onClick": "https://example.com"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context) and [Extensibility Point: "startup"](#extensible_installer-extensibility_points__items-extensibility_point_startup).*

### Final Screen

#### Title

To change the title displayed in the Final screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "DoneScreenTitle",
                "value": "Custom Screen Title"
            }
        }
    ]
}
```

#### Text

To add additional text under the buttons in the Final screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "DoneScreenDetails",
                "value": "Custom Text"
            }
        }
    ]
}
```

To change the text size, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "DoneScreenDetailsFontSize",
                "value": 15
            }
        }
    ]
}
```

#### Documentation Link

To change the documentation link in the Final screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "DocumentationUrl",
                "value": "https://example.com"
            }
        }
    ]
}
```

#### Support Link

To change the support link in the Final screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "SupportEmailOrUrl",
                "value": "https://example.com"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

### Uninstall Screen

To change the title displayed in the Uninstall screen, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "UninstallScreenTitle",
                "value": "Custom Screen Title"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

## Installation Location

To change the default installation location, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "Path",
                "value": "%ProgramFiles%/MyCustomProduct"
            }
        }
    ]
}
```

In an interactive installation, the user can select a custom path using the Custom Install button. To reference the final installation location in your custom scripts, use the `%INSTALL_PATH%` environment variable.

*Note that specific permission or filesystem virtualization setups may cause permission or performance issues if you try to install in a protected location, network drive, etc.*

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

## Unattended Installation

The installation process can be unattended or hidden altogether. In unattended mode, no user interaction is necessary, but the installer screens are still visible. The hidden mode provides the same functionality as the unattended mode, except that the installer screens aren't visible. To run the installer in unattended or hidden mode, either run the executable file with the `/unattended` or `/hidden` command line arguments respectively, or specify the desired mode in the extensibility configuration file for the installer:

```json
// Unattended installation.
{
    "startup": [
        "unattended"
    ]
}
```

```json
// Hidden installation.
{
    "startup": [
        "hidden"
    ]
}
```

By default, an unattended installation will exit with a non-zero exit code when it encounters conflicts, but you can specify options for conflict handling.

To show a message box for the user to dismiss, set the `"conflictHandling"` argument to `"ask"`:

```json
{
    "startup": [
        {
            "type": "hidden",
            "args": {
                // Show a message box to the user in case of conflicts.
                "conflictHandling": "ask"
            }
        }
    ]
}
```

*Note that using `"ask"` may stall the installation if no user is present.*

To try resuming the installation a specified number of times at a specified interval, set the `"conflictHandling"` argument to `"retry"`:

```json
{
    "startup": [
        {
            "type": "hidden",
            "args": {
                // Try resuming the installation in case of conflicts.
                "conflictHandling": "retry",
                // Retry at an interval of 1000 ms.
                "waitMs": 1000,
                // Retry 10 times.
                "retries": 10
            }
        }
    ]
}
```

*Note that the installation process can't be completed while certain applications are running - e.g., [**Glue42 Enterprise**](https://glue42.com/enterprise/), or Excel/Word/Outlook, unless the corresponding [Glue42 Connector](../../../../connectors/general-overview/index.html) has been disabled in the `"artifacts"` extensibility point.*

*See also [Extensibility Point: "startup"](#extensible_installer-extensibility_points__items-extensibility_point_startup).*

## Shortcuts

To create or modify shortcuts, use the `"shortcuts"` extensibility point. The following example demonstrates how to create three shortcuts: two shortcuts to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file with command line arguments specifying override configurations for running [**Glue42 Enterprise**](https://glue42.com/enterprise/) in different environments, and one shortcut to the official Glue42 site. The two shortcuts to [**Glue42 Enterprise**](https://glue42.com/enterprise/) will be created on the desktop and in the Windows Start Menu, and the icons for them will be copied from the repackaged installer to the folder containing the `"GlueDesktop"` artifact. The shortcut to the Glue42 site will be created only on the desktop and will use the `glue.ico` file located in `%LocalAppData%/Tick42/GlueDesktop/assets/images/` as its icon.

```json
{
    "shortcuts": [
        {
            "type": "shortcut",
            "args": {
                "artifact": "GlueDesktop",
                "icon": "copy:icon.ico",
                "filename": "My Custom Product DEV",
                "desktop": true,
                "startMenu": true,
                "description": "Start My Custom Product in DEV environment.",
                "targetArguments": "-- config=config/system.json configOverrides config0=config/system-override-DEV.json"
            }
        },
        {
            "type": "shortcut",
            "args": {
                "artifact": "GlueDesktop",
                "icon": "copy:icon.ico",
                "filename": "My Custom Product UAT",
                "desktop": true,
                "startMenu": true,
                "description": "Start My Custom Product in UAT environment.",
                "targetArguments": "-- config=config/system.json configOverrides config0=config/system-override-UAT.json"
            }
        },
        {
            "type": "shortcut",
            "args": {
                "target": "https://glue42.com",
                "filename": "Glue42 Official Site",
                "description": "Open the Glue42 Official Site",
                "icon": "GlueDesktop/assets/images/glue.ico",
                "startMenu": false,
                "desktop": true
            }
        }
    ]
}
```

*Note that Windows Explorer sometimes fails to show the proper icon after packaging due to icon caching. If the installer icon seems not to be updated, select a different Explorer view (e.g., switch to "Details" or "Tiles") to see the updated icon.*

*See also [Extensibility Point: "shortcuts"](#extensible_installer-extensibility_points__items-extensibility_point_shortcuts).*

## Start Menu Settings

To specify a custom name for the Windows Start Menu folder, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "StartMenuFolderName",
                "value": "Custom Folder Name"
            }
        }
    ]
}
```

To opt out of creating a Start Menu folder, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "CreateStartMenuEntries",
                "value": false
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

## Control Panel Settings

To change the publisher name displayed in the Control Panel "Uninstall or change a program" menu, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "UninstallPublisher",
                "value": "My Company Name"
            }
        }
    ]
}
```

To change the icon displayed in the Control Panel "Uninstall or change a program" menu, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "UninstallDisplayIcon",
                "value": "icon.ico"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

## Artifacts

The following is a list of the names of the available [**Glue42 Enterprise**](https://glue42.com/enterprise/) Artifacts:

Required:

- `"GlueDesktop"`

Glue42 SDKs:

- `"Glue42NET"`
- `"Glue42Java"`
- `"GlueJS"`
- `"Glue4OfficeJS"`

Glue42 Connectors:

- `"GlueXL"`
- `"GlueWord"`
- `"GlueOutlook"`
- `"FidessaConnector"`
- `"BloombergBridge"`

Glue42 Demos:

- `"EnterpriseDemos"`
- `"LocalDemos"`

Other:

- `"GnsDesktopManager"`

*Note that depending on your license for [**Glue42 Enterprise**](https://glue42.com/enterprise/), some of these artifacts may not be available in your installer.*

To make an artifact required, to select it by the default, or to remove it from the installation, use the `"artifacts"` extensibility point. The following example demonstrates how to make the [Excel Connector](../../../../connectors/ms-office/excel-connector/javascript/index.html) required, how to remove the [Word Connector](../../../../connectors/ms-office/word-connector/javascript/index.html) from the installation and how to select by default the [Outlook Connector](../../../../connectors/ms-office/outlook-connector/javascript/index.html):

```json
{
    "artifacts": [
        {
            "type": "GlueXL",
            "args": {
                "required": true
            }
        },
        {
            "type": "GlueWord",
            "args": {
                "remove": true
            }
        },
        {
            "type": "GlueWord",
            "args": {
                "checkedByDefault": true
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

## Custom Scripts

To run a custom script at any installation stage, use the `"run"` extensibility item. The following example demonstrates how to run a script for copying custom resources after the installation has completed:

```json
{
    "done": [
        {
            "type": "run",
            "args": {
                // Name of the program to run.
                "filename": "cmd.exe",
                // Command line arguments for the program execution.
                "args": "/c copy-resources.bat",
                // By default, "exitCode0" is success, while any other is an error message, followed by the installer exiting.
                // You can override any code with "success" for success and any other string for an error message.
                // Note that error messages aren't shown during unattended or hidden installations to avoid stalling.
                "exitCode1": "Error validating the installation.",
                "exitCode2": "Error contacting server.",
                "exitCode3": "success",
                //Other exit codes.

                // Hide the started process.
                "hide": true,

                // Don't hide the installer while the process is running.
                "hideInstaller": false
            }
        }
    ]
}
```

In an interactive installation, the user can select a custom path using the Custom Install button. To reference the final installation location in your custom scripts, use the `%INSTALL_PATH%` environment variable. The following example demonstrates how to copy an entire custom `\splash` directory to the respective `\splash` directory in the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installation folder (e.g., `%LocalAppData%\Tick42\GlueDesktop\assets\splash`) using the `xcopy` command:

```cmd
xcopy /E splash %INSTALL_PATH%\GlueDesktop\assets\splash
```

*See also [Extensibility Item: "run"](#extensible_installer-extensibility_points__items-extensibility_item_run).*

## Client Key

To specify the client key received from the Glue42 support team, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "ClientKey",
                "value": "my-custom-client-key"
            }
        }
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context).*

## Post Installation

### Overriding Configurations

To instruct the installer to replace or merge the [**Glue42 Enterprise**](https://glue42.com/enterprise/) configuration files with your custom ones, use the `"finalizing"` extensibility point. The custom configuration files must be in a ZIP archive (archive the files directly, don't place them in a containing folder). The following example demonstrates how to provide a ZIP file with custom configuration files and merge them with the default ones:

```json
{
    "finalizing": [
        {
            "type": "gdConfig",
            "args": {
                "file": "custom-config.zip",
                // If `false` (default), will merge the custom configuration files with the default ones from the installer
                // by replacing any default file with the respective custom file with the same name.
                // If `true`, the default configuration files will be deleted and replaced with the custom ones.
                // This means that you must provide all required configuration files for Glue42 Enterprise to function properly.
                "wipe": false
            }
        }
    ]
}
```

*Note that this can also be achieved by using the `"run"` extensibility item to execute your custom script for copying custom resources to the respective [**Glue42 Enterprise**](https://glue42.com/enterprise/) folders. For more details, see the [Custom Scripts](#custom_scripts) section.*

*See also [Extensibility Point: "finalizing"](#extensible_installer-extensibility_points__items-extensibility_point_finalizing).*

### Launching

After the installation has completed, you can instruct the installer to automatically launch [**Glue42 Enterprise**](https://glue42.com/enterprise/) or open the folder containing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file. You can also specify command line arguments when launching [**Glue42 Enterprise**](https://glue42.com/enterprise/).

The following example demonstrates how to automatically start [**Glue42 Enterprise**](https://glue42.com/enterprise/) with specified command line arguments after installation by using the `"context"` and `"done"` extensibility points:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "LaunchGDArgs",
                "value": "command-line-arguments"
            }
        }
    ],

    "done": [
        "launchGD"
    ]
}
```

*Note that the specified command line arguments will be used even if you let the user decide whether to launch [**Glue42 Enterprise**](https://glue42.com/enterprise/) after installation.*

The following example demonstrates how to open the folder containing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file after installation:

```json
{
    "done": [
        "showGD"
    ]
}
```

*See also [Extensibility Point: "context"](#extensible_installer-extensibility_points__items-extensibility_point_context) and [Extensibility Point: "done"](#extensible_installer-extensibility_points__items-extensibility_point_done).*

### Exiting the Installer

To instruct the installer to exit automatically after installation, use the `"done"` extensibility point:

```json
{
    "done": [
        // Exit from the Final screen without the user having to click a button.
        {
            "type": "exit",
            "args": {
                "exitCode": 0
            }
        }
    ]
}
```

*See also [Extensibility Point: "done"](#extensible_installer-extensibility_points__items-extensibility_point_done).*

## Signing

As a final step of preparing your custom installer, you have to sign the resulting executable with an Authenticode signing certificate. This is necessary to prevent Windows or antivirus software warnings.

## Extensible Installer Example

This section will provide you with a basic setup for creating a custom repackaged installer and will give you a step-by-step example of how to use an extensibility configuration file to package custom resources and control the installation process.

Download the [basic setup for a custom installer](https://enterprise.glue42.com/upload/docs/sfx-installer-example.zip). Use it to test how the custom installer works, to tweak the setting, add your own files and, ultimately, create your own custom installer.

### Basic Setup Files

The basic setup for creating a custom installer contains the following files and folders:

| Name | Description |
|------|-------------|
| `7-Zip` | File archiving program, necessary for the repackaging process. |
| `config.txt` | Contains settings for the repackaged installer initialization step. Change the values for the `Title` and `BeginPrompt` properties with your preferred name for the internal product deployment. See also the [Extraction Text](#installer_ui-extraction_text) section. |
| `info.rc` | Describes the Windows file properties applied to the repackaged installer. Change the values of the properties inside to your preference. |
| `installer-icon-composite.ico` | Contains the icon used for the repackaged installer executable. Note that Windows Explorer sometimes fails to show the proper icon after packaging due to icon caching. If the installer icon seems not to be updated, select a different Explorer view (e.g., switch to "Details" or "Tiles") to see the updated icon. |
| `produce-sfx-installer.bat` | Script for producing a repackaged installer. After your customizations are complete, run this script to repackage the installer. |
| `ResourceHacker.exe` | Tool for editing resources for Windows applications. |
| `\custom-installer-files` | Contains custom resources that will be included in the repackaged installer (`banner.png`, `icon.ico`, `logo.png`), custom system configuration for [**Glue42 Enterprise**](https://glue42.com/enterprise/) that will replace the default one (`system.json`), a custom script for copying resources after installation (`copy-resources.bat`) and a file for configuring the extensibility features of the installer (`extensibility.json`). Here you must place your [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer and rename it to `glue42-enterprise-installer.exe`. |
| `extensibility.json` | A JSON file located in `\custom-installer-files` containing configuration instructions for the repackaged installer. Use the different [Extensibility Points & Items](#extensible_installer-extensibility_points__items) to control the extensibility features of the installer. |

### Guide

In this step-by-step guide you will be creating a custom [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer with the following additions and changes:

- adding custom logo, banner and icon for the installer screens;

- setting which [Glue42 Artifacts](#artifacts) to be required or removed during the installation;

- customizing the title and text of the installer windows;

- setting the foreground and background colors for the installer screens;

- creating a custom shortcut;

- installing [**Glue42 Enterprise**](https://glue42.com/enterprise/) with a custom system configuration in order to setup [**Glue42 Enterprise**](https://glue42.com/enterprise/) to retrieve [application configurations from a REST service](../functionality/index.html#remote_apps__layouts-rest_stores-apps);

*You must have a running REST service from which to retrieve application configurations. For testing purposes, you can use the [Node.js REST Configuration Example](https://github.com/Tick42/rest-config-example-node-js).*

- running a custom script that will copy a resource to a [**Glue42 Enterprise**](https://glue42.com/enterprise/) folder after installation;

- showing a message to the user;

- opening the folder containing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file after the installation has completed;

Follow these steps to create the customized installer:

1. Navigate to the directory where you have downloaded the `sfx-installer-example.zip` and extract the files in it.

2. Copy your [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer to the `\custom-installer-files` directory and rename it to `"glue42-enterprise-installer.exe"`.

*You can download a trial version of [**Glue42 Enterprise**](https://glue42.com/enterprise/) from [here](https://glue42.com/free-trial/).*

3. **Custom logo, banner and icon**

For convenience, the `\custom-installer-files` folder already contains example custom files for the installer logo, banner and icon, but you can replace them with your own files.

Open the `extensibility.json` file and in the `"startup"` extensibility point add the following configuration for the logo and the banner:

```json
{
    "startup": [
        {
            "type": "logo",
            "args": {
                "file": "logo.png",
                "onClick": "https://example.com"
            }
        },
        {
            "type": "banner",
            "args": {
                "file": "banner.png",
                "onClick": "https://example.com"
            }
        }
    ]
}
```

Use the `"onClick"` property to specify a URL to open when the user clicks on the banner or the logo, or skip it if you don't want clicking on the banner or the logo to open a web page.

Go to the `"context"` extensibility point and add the following configuration for the icon:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "Icon",
                "value": "icon.ico"
            }
        }
    ]
}
```

4. **Requiring and removing Glue42 artifacts**

To make some of the [Glue42 Artifacts](#artifacts) required or to remove them from the installation, use the `"artifacts"` extensibility point. Add the following configuration to make the `"GlueXL"` artifact required and remove the `"GlueWord"` artifact from the installation:

```json
{
    "artifacts": [
        {
            "type": "GlueXL",
            "args": {
                "required": true
            }
        },
        {
            "type": "GlueWord",
            "args": {
                "remove": true
            }
        }
    ]
}
```

*To check that this configuration is in effect, click on the Custom Install button during the installation and look for the specified artifacts.*

5. **Changing the installation window title and text**

To change the default installer window name and the text that is shown when installing the Glue42 components, use the `"context"` extensibility point:

```json
{
    "context": [
        {
            "type": "setValue",
            "args": {
                "name": "WindowName",
                "value": "Custom Glue42 Installer"
            }
        },
        {
            "type": "setValue",
            "args": {
                "name": "DownloadScreenTitle",
                "value": "Installing components..."
            }
        }
    ]
}
```

6. **Changing the colors of the installer screens**

To change the foreground and background color of the installer screens, use the `"styles"` extensibility point. Add the following configuration to change the background color to black and the foreground color to white:

```json
{
    "styles": [
        {
            "type": "color",
            "args": {
                "name": "Background",
                "value": "#000000"
            }
        },
        {
            "type": "color",
            "args": {
                "name": "Foreground",
                "value": "#ffffff"
            }
        }
    ]
}
```

7. **Creating a custom shortcut**

To define a custom shortcut to be created during installation, use the `"shortcuts"` extensibility point. Add the following configuration to create a shortcut to the official Glue42 site on the desktop, but not in the Windows Start menu:

```json
{
    "shortcuts": [
        {
            "type": "shortcut",
            "args": {
                "target": "https://glue42.com",
                "filename": "Glue42 Official Site",
                "description": "Open the Glue42 Official Site",
                "icon": "GlueDesktop/assets/images/icon.ico",
                "startMenu": false,
                "desktop": true
            }
        }
    ]
}
```

The shortcut will use the provided custom icon file for its icon. This icon file will be copied in a later step to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) folder specified here.

8. **Custom system configuration file for Glue42 Enterprise**

Use the default `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in the `%LocalAppData%\Tick42\GlueDesktop\config` folder as a basis for your custom system configuration. Get the `system.json` from a previous installation of [**Glue42 Enterprise**](https://glue42.com/enterprise/) and modify the `"appStores"` top-level array to configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to [retrieve application configurations from a REST service](../functionality/index.html#remote_apps__layouts-rest_stores-apps):

```json
{
    "appStores": [
        {
            "type": "rest",
            "details": {
                // URL to your remote app store.
                "url": "http://localhost:8004/apps/",
            }
        }
    ]
}
```

Archive the customized `system.json` file as a ZIP file named `system.zip` - archive the file directly, don't place it in a containing folder. Place the `system.zip` file in the `\custom-installer-files` folder.

Use the `"finalizing"` extensibility point to instruct the installer to replace the default `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) with your custom one. The `"wipe"` property is set to `false` in order to prevent the installer from deleting all other necessary configuration files:

```json
{
    "finalizing": [
        {
            "type": "gdConfig",
            "args": {
                "file": "system.zip",
                "wipe": false
            }
        }
    ]
}
```

9. **Copying a custom resource**

The custom `icon.ico` file must be copied to the `%LocalAppData%\Tick42\GlueDesktop\assets\images\` directory so that the previously created custom shortcut can use it as its icon. Use the `"done"` extensibility point to run the custom `copy-resources.bat` script after the installation has completed:

```json
{
    "done": [
        {
            "type": "run",
            "args": {
                "filename": "cmd",
                "args": "/c copy-resources.bat"
            }
        }
    ]
}
```

10. **Showing a message to the user**

To show a custom message for completed installation, use the `"done"` extensibility point:

```json
{
    "done": [
        {
            "type": "message",
            "args": {
                "title": "Installation Completed",
                "text": "Glue42 Enterprise installed successfully!"
            }
        }
    ]
}
```

11. **Opening the folder containing the Glue42 Enterprise executable file**

To instruct the installer to open the folder containing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file, use the `"done"` extensibility point:

```json
{
    "done": [
        "showGD"
    ]
}
```

12. Go back to the root folder and start the `produce-sfx-installer.bat` script to produce the repackaged installer file. The output installer is named `sfx-installer.exe`.

13. Run `sfx-installer.exe` to install the product and test the customizations.

*Note that the SFX installer should be Authenticode signed to prevent false alerts from Windows or antivirus software.*