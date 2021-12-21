## Guide

To start debugging your [**Glue42 Enterprise**](https://glue42.com/enterprise/) app in Visual Studio Code, execute the following steps:

1. Start [**Glue42 Enterprise**](https://glue42.com/enterprise/) with the `--remote-debugging-port` command line argument and specify a debugging port:

```cmd
tick42-glue-desktop.exe --remote-debugging-port=9222
```

Using the `--remote-debugging-port` command line argument instructs [**Glue42 Enterprise**](https://glue42.com/enterprise/) to listen on the specified port for a debug connection.

2. Add a new section to the `launch.json` file of Visual Studio Code with the following configuration:

```json
{
    "name": "Glue42: Renderer",
    "type": "chrome",
    "request": "attach",
    "targetSelection": "pick",
    "port": 9222,
    "webRoot": "${workspaceFolder}",
    "timeout": 30000
}
```

*Note that the port number in the `launch.json` configuration must be the same as the one specified in the `--remote-debugging-port` command line argument for starting the [**Glue42 Enterprise**](https://glue42.com/enterprise/) executable file.*

3. Start your application. To debug your application locally, start your app on `localhost` or other development server and update the `"url"` property in your [application definition](../configuration/application/index.html) with the respective `localhost` or development server URL.

4. Select the `Glue42: Renderer` profile from "Run and Debug" in Visual Studio Code.

5. Select your application tab and start debugging.