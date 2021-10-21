## Overview

[**Glue42 Enterprise**](https://glue42.com/enterprise/) is highly configurable. You can apply custom system-wide, as well as application-specific settings. Among the configurable features are system settings, application settings, Glue42 Windows, Glue42 Themes and more.

### Schemas

The following JSON schemas describe the available options for configuring [**Glue42 Enterprise**](https://glue42.com/enterprise/):

- [System configuration schema](../../../assets/configuration/system.json)
- [Application configuration schema](../../../assets/configuration/application.json)
- [Glue42 Windows configuration schema](../../../assets/configuration/stickywindows.json)
- [Themes configuration schema](../../../assets/configuration/themes.json)

## Location

All configuration files for [**Glue42 Enterprise**](https://glue42.com/enterprise/) can be found in `%LocalAppData%\Tick42\GlueDesktop\config` (mainly in the form of JSON files). There you can find a `system.json` file for system-wide configurations, `stickywindows.json` for configuring the default behavior of [Glue42 windows](../../../glue42-concepts/windows/window-management/overview/index.html), `themes.json` for configuring the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [Themes](../../../glue42-concepts/windows/themes/index.html) and `channels.json` for configuring the Glue42 [Channels](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html).

In the `\apps` folder of the same directory, you can find the configuration files for the apps bundled with the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer.   

You can add an application configuration file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder to publish your own application locally. The `<ENV-REG>` placeholder should be replaced with the region and environment folder name used for the deployment of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) - e.g., `T42-DEMO`. This way, your files will not be erased or overwritten, in case you decide to upgrade or change your [**Glue42 Enterprise**](https://glue42.com/enterprise/) version.  

Note that the application configurations are monitored at runtime, so you don't need to restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) when you change something. The location of the application configuration folder can be changed from the `system.json`. For more information, see the `system.json` [configuration schema](../../../assets/configuration/system.json).

## Extending Configurations

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

All [**Glue42 Enterprise**](https://glue42.com/enterprise/) configuration files located in `%LocalAppData%\Tick42\GlueDesktop\config` can accept an `"extends"` top-level property in which you can specify a configuration store or configuration file locations. The current configuration file will extend the respective configuration from the specified local or remote file, or from the REST store providing the configuration files. You can define any number and type of configuration endpoints - the configuration settings will be merged in the order they are specified in the `"extends"` array.

Example configuration for extending a `system.json` file by combining settings from remote, local and REST store configuration files:

```json
{
    "env": "DEV",
    "region": "US",
    "extends": [
        { 
            "type": "remote", 
            "source": "https://something.com/glue42/%GLUE-REGION%-%GLUE-ENV%.system.json"
        },
        { 
            "type": "file", 
            "source": "./local.system.json" 
        },
        { 
            "type": "rest", 
            "source": "https://something.com/glue42/config",
            "auth": "negotiate"
        }             
    ]
}
```

| Property | Description |
|----------|-------------|
| `"type"` | The type of configuration file - `"local"`, `"remote"` or `"rest"`. |
| `"source"` | The path to a local file, the URL to a remote file or the endpoint of the REST store providing the configuration. |
| `"auth"` | Only applicable to `"rest"` type items. Can be `"no-auth"` (default), `"negotiate"` or `"kerberos"`. |

As you can see from the example, you can use Glue42 environment variables for region and environment in the paths or URLs to the configuration files. The values for the variables can be set in the `system.json` file or as OS environment variables. The values from the `system.json` will override the ones from the OS.

*Note that due to limitations regarding the Electron startup, the `"splash"`, `"cache"` and `"flash"` properties must be set locally in the `system.json` file.*

*See the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example to explore and test extending configurations from a REST store.*