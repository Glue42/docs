## Login Screen

[**Glue42 Enterprise**](https://glue42.com/enterprise/) allows showing a login screen before the first application is loaded. This is useful if you have shared authentication between your apps (SSO) and you want the user to log in just once.

To enable using a login screen, you have to modify the system configuration of [**Glue42 Enterprise**](https://glue42.com/enterprise/). To complete the authentication process and allow the user access, you have to signal Glue42 that the user has logged in successfully.

### Configuration

To enable the login screen, use the `"ssoAuth"` top-level key of the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config` and set its `"authController"` property to `"sso"`. Use the `"options"` property to provide the location of the login screen and settings for the Glue42 Window in which it will be loaded:

```json
{
    "ssoAuth": {
        "authController": "sso",
        "options": {
            "url": "http://localhost:3000/",
            "window": {
                "width": 500,
                "height": 730,
                "mode": "flat"
            }
        }
    }
}
```

The `"options"` object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"url"` | `string` | **Required.** Location of the login screen. |
| `"window"` | `object` | Settings for the Glue42 Window in which the login screen will be loaded. |
| `"keepAlive"` | `boolean` | If `true`, [**Glue42 Enterprise**](https://glue42.com/enterprise/) won't close the login window. This way, you can hide it yourself and use it to refresh the authentication parameters (user, token and headers) when necessary. |

The only required property of the `"options"` object is `"url"`, which must point to the location of the login screen.

The `"window"` object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"width"` | `integer` | Width of the login window. |
| `"height"` | `integer` | Height of the login window. |
| `"mode"` | `string` | Glue42 Window [mode](../../../../glue42-concepts/windows/window-management/overview/index.html#window_modes). Possible values are `"html"` (default), `"flat"` and `"tab"`. |

*See also the [authentication controller schema](../../../../assets/configuration/authController.json).*

### Authentication

To allow the user access after authenticating, you must signal [**Glue42 Enterprise**](https://glue42.com/enterprise/) that the authentication process is complete. Use the `authDone()` method of the `glue42gd` object which is injected in the global `window` object. It accepts an *optional* object as a parameter in which you can specify the name of the authenticated user, а token and headers:

```javascript
const options = {
    user: "john.doe@org.com",
    token: "token",
    headers: {
        "name": "value"
    }
};

glue42gd.authDone(options);
```

The optional object passed as an argument to `authDone()` has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `user` | `string` | The user ID will be set as a value of the `sid` property of `glue42gd`. Can be used for visualization purposes. |
| `token` | `string` | The token will be applied to each request to the remote stores or the [Glue42 Server](../../../../glue42-concepts/glue42-server/index.html). |
| `headers` | `JSON object` | Extra headers that will be passed to the remote stores or the [Glue42 Server](../../../../glue42-concepts/glue42-server/index.html). |

## Remote Applications & Layouts

In the standard [**Glue42 Enterprise**](https://glue42.com/enterprise/) deployment model, application definitions aren't stored locally on the user machine, but are served remotely. Remote application and Layout stores can be hosted and retrieved using the [Glue42 Server](#remote_applications__layouts-glue42_server) and/or [REST services](#remote_applications__layouts-rest_stores).

The settings for the application and Layout stores are defined in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config`. The app store settings are under the `"appStores"` top-level key, the Layout store settings are under the `"layouts"` top-level key.

*Note that [**Glue42 Enterprise**](https://glue42.com/enterprise/) respects the FDC3 standards and can retrieve standard Glue42, as well as FDC3-compliant application definitions. For more details on working with FDC3-compliant apps, see the [FDC3 Compliance](../../../fdc3-compliance/index.html) section, the [FDC3 App Directory documentation](https://fdc3.finos.org/docs/app-directory/overview) and the [FDC3 Application](https://fdc3.finos.org/schemas/1.2/app-directory#tag/Application) schema.*

### Glue42 Server

#### Applications

To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to fetch application configurations from a [Glue42 Server](../../../../glue42-concepts/glue42-server/index.html), set the `"type"` property of the app store configuration object to `"server"`:

```json
{
    "appStores": [
        {
            "type": "server"
        }
    ]
}
```

#### Layouts

To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to fetch application configurations from a [Glue42 Server](../../../../glue42-concepts/glue42-server/index.html), set the `"type"` property of the `"store"` object to `"server"`:

```json
{
    "layouts" : {
        "store": {
            "type": "server"
        }
    }
}
```

*Note that when using the [Glue42 Server](../../../../glue42-concepts/glue42-server/index.html) as a Layout store, Layout files aren't only fetched from the server, but are also saved on the server (e.g., when the user edits and saves an existing Layout).*

### REST Stores

#### Applications

To configure a connection to the REST service providing the application store, add a new entry to the `"appStores"` top-level key of the `system.json` file:

```json
{
    "appStores": [
        {
            "type": "rest",
            "details": {
                "url": "http://localhost:3000/appd/v1/apps/search",
                "auth": "no-auth",
                "pollInterval": 30000,
                "enablePersistentCache": true,
                "cacheFolder": "%LocalAppData%/Tick42/UserData/%GLUE-ENV%-%GLUE-REGION%/gcsCache/"
            }
        }
    ]
}
```

The only required properties for each app store configuration object are `"type"`, which should be set to `"rest"`, and `"url"`, which is the address of the remote application store. You can also set the authentication, polling interval, cache persistence and cache folder.

| Property | Description |
|----------|-------------|
| `"auth"` | Authentication configuration. Can be one of `"no-auth"`, `"negotiate"` or `"kerberos"`. |
| `"pollInterval"` | Interval in milliseconds at which to poll the REST service for updates. |
| `"enablePersistentCache"` | Whether to cache and persist the configuration files locally (e.g., in case of connection interruptions). |
| `"cacheFolder"` | Where to keep the persisted configuration files. |

The remote store must return application definitions in the following response shape:

```json
{
    "applications": [
        // List of application definition objects.
        {...}, {...}
    ]
}
```

#### Layouts

To configure a connection to the REST service providing the Layout store, edit the `"layouts"` top-level key of the `system.json` file:

```json
{
    "layouts": {
        "store": {
            "type": "rest",
            "restURL": "http://localhost:8004/",
            "restFetchInterval": 20,
            "restClientAuth": "no-auth"
        }
    }
}
```

| Property | Description |
|----------|-------------|
| `"type"` | Can be `"file"`, `"rest"` or `"server"`, depending on the type of Layout store. |
| `"restURL"` | The URL address of the Layouts REST service. |
| `"restFetchInterval"` | Interval in seconds at which to fetch Layouts from the REST service. |
| `"restClientAuth"` | Authentication configuration. Can be one of `"no-auth"`, `"negotiate"` or `"kerberos"`. |

*The `"restURL"`, `"restFetchInterval"` and `"restClientAuth"` properties are valid only when `"type"` is set to `"rest"`. Otherwise, they are ignored.*

The remote store must return Layout definitions in the following response shape:

```json
{
    "layouts": [
        // List of Layout definition objects.
        {...}, {...}
    ]
}
```

### Example Store Implementations

#### Applications

For a reference implementation of a remote application configurations store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example that implements the [FDC3 App Directory](https://fdc3.finos.org/docs/1.0/appd-intro) and is compatible with [**Glue42 Enterprise**](https://glue42.com/enterprise/). This basic implementation doesn't take the user into account and returns the same set of data for all requests. For instructions on running the sample server on your machine, see the README file in the repository.

For a .NET implementation of a remote application configurations store, see the [.NET REST Config](https://github.com/Tick42/rest-config-example-net) example.

#### Layouts

For a reference implementation of a remote Layout definitions store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example. The user Layouts are stored in files with the same structure as local Layout files. This basic implementation doesn't take the user into account and returns the same set of data for all users. New Layouts are stored in files using the name of the Layout and there isn't validation for the name. The operation for removing a Layout isn't implemented and just logs to the console. For instructions on running the sample server on your machine, see the README file in the repository.

For a .NET implementation of a remote Layout definitions store, see the [.NET REST Config](https://github.com/Tick42/rest-config-example-net) example.

## Environments & Regions

[**Glue42 Enterprise**](https://glue42.com/enterprise/) allows running it in different environments and regions. Environments usually include development, testing, quality assurance, production environments, etc., in which [**Glue42 Enterprise**](https://glue42.com/enterprise/) is tested or integrated. Regions can be any semantic groups - geographic regions, user groups, product categories, etc., defined by the client adopting [**Glue42 Enterprise**](https://glue42.com/enterprise/).

To enable [**Glue42 Enterprise**](https://glue42.com/enterprise/) to run in different environments/regions, you must:

- create a separate [system configuration file](#environments__regions-system_configuration_files) for each environment/region combination. You can use the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config` as a base system configuration template, common for all environments/regions;

- create different [shortcuts](#environments__regions-shortcuts) pointing to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file for each environment/region, and set command line arguments specifying the respective environment/region configuration file that will be merged with the base system configuration file;

### System Configuration Files

To set different environments/regions in which to run [**Glue42 Enterprise**](https://glue42.com/enterprise/), use the `system.json` configuration file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) as a common configuration. Create separate system configuration files for all environments/regions and in each one define only the settings specific for the environment/region. When you start [**Glue42 Enterprise**](https://glue42.com/enterprise/) in a certain environment/region, the respective configuration file will be merged with the base `system.json` file, overriding the settings in it.

It is important to either specify a different port for the Glue42 Gateway for each environment/region, or to use a [Dynamic Gateway Port](../../../../developers/configuration/system/index.html#dynamic_gateway_port) by setting the Glue42 Gateway port to `0` in the base system configuration file. Otherwise, you won't be able to run multiple instances of [**Glue42 Enterprise**](https://glue42.com/enterprise/) simultaneously in different environments/regions, as the first started instance will occupy the specified port and the other instances won't be able to connect.

Use the `"gw"` top-level key in the `system.json` file to set the port number. The following example demonstrates how to configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to use a random free port for each of its instances by setting the Glue42 Gateway port to `0`:

```json
{
    "gw": {
        "configuration": {
            "port": 0
        }
    }
}
```

The following example demonstrates creating a system configuration file for a `"PROD"` environment and an `"EMEA"` region, named `system-PROD-EMEA.json`. The port for the Glue42 Gateway isn't specified, as it is set to `0` in the common system configuration file. The only setting that will be overridden is for the [application stores](#remote_applications__layouts) under the `"appStores"` top-level key:

```json
// system-PROD-EMEA.json
{
    "env": "PROD",
    "region": "EMEA",
    "appStores": [
        {
            "type": "rest",
            "details": {
                "url": "https://my-domain.com/my-app-store",
                "auth": "no-auth",
                "pollInterval": 30000,
                "enablePersistentCache": true,
                "cacheFolder": "%LocalAppData%/Tick42/UserData/%GLUE-ENV%-%GLUE-REGION%/gcsCache/"
            }
        }
    ]
}
```

Keep in mind that by default [**Glue42 Enterprise**](https://glue42.com/enterprise/) is configured to create sub-folders for each environment/region in the `UserData` and `Cache` folders by using the `%GLUE-ENV%` and `%GLUE-REGION%` environment variables:

```json
{
    "folders": {
        "userData": "%LocalAppData%/Tick42/UserData/%GLUE-ENV%-%GLUE-REGION%/",
        "cache": "%LocalAppData%/Tick42/Cache/%GLUE-ENV%-%GLUE-REGION%/"
    }
}
```

This means that the information stored in these folders by default will be separated per environment/region. To avoid overwriting data in case you decide to change the locations of `UserData` and `Cache` in your base configuration, either use the `%GLUE-ENV%` and `%GLUE-REGION%` environment variables to create sub-folders for each environment/region, or make sure that each environment/region configuration file specifies different locations for these folders.

### Shortcuts

It is possible to create any number of shortcuts to [**Glue42 Enterprise**](https://glue42.com/enterprise/), which will allow you to launch it with custom configuration according to your requirements. The entry point executable is found in `%LocalAppData%\Tick42\GlueDesktop\tick42-glue-desktop.exe`. Running it without arguments will use the default configuration located in `%LocalAppData%\Tick42\GlueDesktop\config\system.json`. To override the base configuration with the specific configuration for an environment/region, use the following template to provide command line arguments for each shortcut:

```cmd
tick42-glue-desktop.exe [optional Electron arguments] -- config=<base config file> configOverrides config0=<custom config override> config1=...
```

Substitute `<base config file>` with the location of your base configuration file (usually `config/system.json`), and `<custom config override>` with the location of your configuration override file. It's possible to supply more than one configuration override files.

The following example demonstrates how to use the `system.json` file as a base configuration for [**Glue42 Enterprise**](https://glue42.com/enterprise/) and override it with a file named `system-PROD-EMEA.json` and also placed in the `%LocalAppData%\Tick42\GlueDesktop\config` folder:

```cmd
tick42-glue-desktop.exe -- config=config/system.json configOverrides config0=config/system-PROD-EMEA.json
```

*Note that to specify command line arguments using a shortcut to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file, you must right click on the shortcut, select the "Properties" menu option, go to the "Target" field and add the arguments after the name of the executable file. Note the space before and after the double dash.*

*For details on how to automatically create shortcuts during installation when using the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer application for deployment, see the [Installer > Shortcuts](../installer/index.html#shortcuts) section. Otherwise, you can use PowerShell or any other deployment technology to create shortcuts.*

## Issue Reporting

[**Glue42 Enterprise**](https://glue42.com/enterprise/) has a built-in Feedback Form that allows users to send feedback with improvement suggestions or bug reports. To report a problem or submit a suggestion, describe it in the "Description" field and optionally attach logs and configs to the report. The form can be configured to send an email with the report to the Glue42 team and/or to automatically create a Jira ticket with the issue reported by the user. Both on-premise and cloud-based Jira solutions are supported. The Feedback Form is an HTML app and can be re-designed to suit specific client needs and requirements.

![Feedback Form](../../../../images/rebrand-glue42/feedback-form.png)

### System Configuration

To configure the Feedback Form, use the `"issueReporting"` top-level key of the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config`. The following is an example Feedback Form configuration:

```json
{
    "attachmentsViewMode": "category",
    "jira": {
        "enabled": true,
        "url": "https://jira.tick42.com/rpc/soap/jirasoapservice-v2",
        "user": "user",
        "password": "password",
        "project": "MY_PROJECT"
    },
    "mail": {
        "enabled": true,
        "zipAttachments": true,
        "bugSubjectPrefix": "Error:",
        "reportSubjectPrefix": "Feedback:",
        "recipients": ["support@example.com", "dev-support@example.com"],
        "allowEditRecipients": true,
        "maxAttachmentMB": "10"
    },
    "folderAttachments": [
        {
            "name": "G4E-Screenshots",
            "folderPath": "%GLUE-USER-DATA%/logs",
            "zipFolderPath": "GlueDesktop\\Screenshots",
            "filter": "*.png",
            "category": "Screenshots",
            "selected": false,
            "recursive": true
        },
        {
            "name": "G4E-AppLogs",
            "folderPath": "%GLUE-USER-DATA%/logs",
            "zipFolderPath": "GlueDesktop\\Logs",
            "filter": "*.log*",
            "category": "Logs",
            "selected": true,
            "recursive": true
        },
        {
            "name": "G4E-Crashes",
            "folderPath": "%GLUE-USER-DATA%/crashes/reports",
            "zipFolderPath": "GlueDesktop\\Crashes",
            "filter": "*.dmp",
            "category": "Dumps",
            "recursive": false
        }
    ]
}
```

The `"issueReporting"` top-level key has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"jira"` | `object` | **Required.** Jira configuration. For more details, see the [Jira](#issue_reporting-jira) section. |
| `"mail"` | `object` | **Required.** Mail configuration. For more details, see the [Mail](#issue_reporting-mail) section. |
| `"folderAttachments"` | `object[]` | **Required.** Attachments configuration. For more details, see the [Attachments](#issue_reporting-attachments) section. |
| `attachScreenShots` | `boolean` | Whether to attach screenshots of all monitors to the issue report. |
| `attachmentsViewMode` | `string` | Possible values are `"file"` or `"category"`. Defines how the attachments will be displayed in the Feedback Form. If set to `"file"`, all attachable files will be displayed as separate items, otherwise several main categories will be displayed and if the category is selected by the user, all files in it will be attached. |

### Jira

The configuration for automatically creating a Jira ticket when submitting a Feedback Form is found under the required `"jira"` property of the `"issueReporting"` top-level key. It accepts an object with the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"enabled"` | `boolean` | If `true`, will enable the option to create a Jira ticket when submitting the issue. |
| `"url"` | `string` | **Required.** Link to the Jira API. |
| `"user"` | `string` | **Required.** The username of the user that will create the ticket. |
| `"password"` | `string` | **Required.** The password of the user that will create the ticket. |
| `"project"` | `string` | **Required.** The name of the project in which the ticket will be created. |
| `"preferredRole"` | `string` | Preferred role in the Jira project. |
| `"preferredRoleDescriptor"` | `string` | Description of the preferred role in the Jira project. |
| `"useProjectLeadAsPreferredAssignee"` | `boolean` | Whether to assign the ticket to the project lead. |
| `"tlsVersion"` | `string` | Force TLS protocol version, e.g. `"TLS1.2"`. |
| `"noPriority"` | `boolean` | If `true`, the ticket "Priority" field won't be set. |
| `"noEnvironment"` | `boolean` | If `true`, the ticket "Environment" field won't be set. |

### Mail

The configuration for automatically sending an email when submitting a Feedback Form is found under the required `"mail"` property of the `"issueReporting"` top-level key. It accepts an object with the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"enabled"` | `boolean` | If `true`, will enable the option to send an email when submitting the issue. |
| `"zipAttachments"` | `boolean` | If `true`, the attachments will be archived. |
| `"bugSubjectPrefix"` | `string` | Prefix for the email subject when the issue is a bug. |
| `"reportSubjectPrefix"` | `string` | Prefix for the email subject when sending feedback. |
| `"recipients"` | `string[]` | A list of email addresses to which the issue report will be sent. |
| `"allowEditRecipients"` | `boolean` | If `true`, the user will be allowed to manually add more recipients. |
| `"maxAttachmentMB"` | `string` | The maximum size of the attachments in MB. |

### Attachments

The configuration for attaching files when submitting a Feedback Form is found under the required `"folderAttachments"` property of the `"issueReporting"` top-level key. It accepts an array of objects, each with the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"name"` | `string` | **Required.** Name for the attachment. |
| `"folderPath"` | `string` | **Required.** Path to the folder that will be attached. |
| `"zipFolderPath"` | `string` | **Required.** Path in the archived folder. |
| `"filter"` | `string` | **Required.** Regex filter that will be applied to each file in the folder - e.g., `"*.log"`. |
| `"category"` | `string` | **Required.** Category for the attached item. |
| `"selected"` | `boolean` | If `true`, the category (or all files under this category) in the Feedback Form will be selected by default. |
| `"recursive"` | `string` | If `true`, will collect and attach all files from all folders located inside the specified folder. Set to `false` to collect the files only in the specified folder. |

## Other System Configurations

### Multiple Instances

To control running multiple instances of [**Glue42 Enterprise**](https://glue42.com/enterprise/), use the `"allowMultipleInstances"` top-level key in the `system.json` file.

The following example demonstrates how to prevent users from running multiple instances of [**Glue42 Enterprise**](https://glue42.com/enterprise/):

```json
{
    "allowMultipleInstances": "never"
}
```

The `"allowMultipleInstances"` property accepts the following values:

| Value | Description |
|-------|-------------|
| `"inDifferentEnvRegion"` | Default. Allows running multiple instances only in different [environments/regions](#environments__regions). |
| `"never"` | Disables running multiple instances. |

### Multiple Versions

It's possible to install multiple versions of [**Glue42 Enterprise**](https://glue42.com/enterprise/) on the same machine. If you plan on supporting multiple installations, ensure that the different versions are installed in separate folders, and use relative paths in the `system.json` configuration file in order to avoid data overwriting.

*Note that the [MS Office Connectors](../../../../connectors/ms-office/overview/index.html) are integrated with the registered MS Office application at installation time. This means that only their last-installed version will be available.*

### Global Protocol Handler

The default name of the Glue42 [global protocol](../../../../glue42-concepts/glue42-platform-features/index.html#global_protocol_handler) is `glue42`, but you can change this prefix using the `"protocol"` property of the `"protocolHandler"` top-level key in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/):

```json
{
    "protocolHandler": {
        "protocol": "mycustomprotocol"
    }
}
```

The following example demonstrates how to start a Glue42 enabled application using the `app` protocol option after you have changed the default protocol name:

```cmd
mycustomprotocol://app/clientlist
```