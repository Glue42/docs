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

You can add an application configuration file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder to publish your own application locally. The `<ENV-REG>` placeholder should be replaced with the region and environment folder name used for the deployment of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) - e.g., `T42-DEMO`. This way, your files won't be erased or overwritten, in case you decide to upgrade or change your [**Glue42 Enterprise**](https://glue42.com/enterprise/) version.  

Note that the application configurations are monitored at runtime, so you don't need to restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) when you change something. The location of the application configuration folder can be changed from the `system.json`. For more information, see the `system.json` [configuration schema](../../../assets/configuration/system.json).