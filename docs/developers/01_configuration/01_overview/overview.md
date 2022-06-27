## Overview

[**Glue42 Enterprise**](https://glue42.com/enterprise/) is highly configurable. You can apply custom system-wide, as well as app-specific settings. Among the configurable features are system settings, app settings, Glue42 Windows, Glue42 Themes, logging and more.

### Schemas

The following JSON schemas describe the available options for configuring [**Glue42 Enterprise**](https://glue42.com/enterprise/):

- [System configuration schema](../../../assets/configuration/system.json)
- [App configuration schema](../../../assets/configuration/application.json)
- [Glue42 Windows configuration schema](../../../assets/configuration/stickywindows.json)
- [Themes configuration schema](../../../assets/configuration/themes.json)

*Note that a JSON schema for the [logging configuration](../system/index.html#logging) found in the `logger.json` file isn't available, as the [**Glue42 Enterprise**](https://glue42.com/enterprise/) logging mechanism is based on [`log4js-node`](https://github.com/log4js-node/log4js-node). The logging configuration provided in the `logger.json` file is identical to the `log4js-node` configuration as described in the `log4js-node` [API documentation](https://log4js-node.github.io/log4js-node/api.html).*

## Location

All configuration files for [**Glue42 Enterprise**](https://glue42.com/enterprise/) can be found in `%LocalAppData%\Tick42\GlueDesktop\config` (mainly in the form of JSON files). There you can find the `system.json` file for system-wide configurations, the `stickywindows.json` file for configuring the default behavior of [Glue42 windows](../../../glue42-concepts/windows/window-management/overview/index.html), the `themes.json` file for configuring the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [Themes](../../../glue42-concepts/windows/themes/index.html), the `channels.json` file for configuring the Glue42 [Channels](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html), and the `logger.json` file for configuring the [logging](../system/index.html#logging) for [**Glue42 Enterprise**](https://glue42.com/enterprise/).

In the `\apps` folder of the same directory, you can find the configuration files for the apps bundled with the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer.

You can add an app configuration file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder to publish your own app locally. The `<ENV-REG>` placeholder should be replaced with the region and environment folder name used for the deployment of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy - e.g., `T42-DEMO`. This way, your files won't be erased or overwritten, in case you decide to upgrade or change your [**Glue42 Enterprise**](https://glue42.com/enterprise/) version.

Note that the app configurations are monitored at runtime, so you don't need to restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) when you change something. The location of the app configuration folder can be changed from the `system.json`. For more information, see the `system.json` [configuration schema](../../../assets/configuration/system.json).