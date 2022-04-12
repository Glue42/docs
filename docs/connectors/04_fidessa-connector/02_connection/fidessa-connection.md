## Connection

If for any reason the connection with the Fidessa WebSocket API is lost, the Fidessa Connector will follow the instructions set in the `reconnectPeriod` property in the custom configuration. By default, it will try to reconnect 5 times at a 2 second interval and after that, it will try to reconnect every 30 seconds. This can be changed from the configuration.

## Settings

### Default

Default configuration settings:

```json
{
    "wsUrl": "ws://localhost:80/ITP/",
    "throttleMs": 500,
    "serviceMethod": "Glue42.ContextMapper.Enhance",
    "fimPath": "id.fim",
    "instrumentIdPath": "id",
    "fidClientPath": "fidId",
    "clientIdPath": "",
    // By default, there is no limit to the reconnect attempts.
    // Set a number value to this property to define a limit.
    // "maxReconnectAttempts": <infinity>
    "reconnectPeriod": "2000(5), 30000"
}
```

- `wsUrl` - WebSocket URL to which to connect;
- `throttleMs` - the time to wait and aggregate Channel updates before sending them to Fidessa;
- `serviceMethod` - the name of the method to use as a mapping service;
- `fimPath` - path to the FIM representation of the instrument name inside the `instrument` key of the context object (see [Fidessa Tracking Groups](../fidessa-tracking-groups/index.html));
- `instrumentIdPath` - path to the `id` property (holding the instrument ID) inside the `instrument` key of the context object (see [Fidessa Tracking Groups](../fidessa-tracking-groups/index.html));
- `fidClientPath` - path to the Fidessa ID of the client inside the `client` key of the context object (see [Fidessa Tracking Groups](../fidessa-tracking-groups/index.html));
- `clientIdPath` - path to the `id` property (holding the client ID) inside the `client` key of the context object (see [Fidessa Tracking Groups](../fidessa-tracking-groups/index.html));
- `maxReconnectAttempts` - how many times to attempt to reconnect. By default, there is no set limit to the reconnect attempts;
- `reconnectPeriod` - accepts a string in the format `"<number>(<number>), <number>"` which defines how the Fidessa Connector should attempt reconnection. The first number in the string represents an interval (in ms) at which to attempt reconnection, followed by the number of initial reconnection attempts (the second number in the brackets). The third number is also a reconnection interval (in ms) which the Connector should use after the initial reconnection attempts have failed;

### Custom

To change the default connection settings, use the `"customProperties"` top-level key of your [application configuration](../../../developers/configuration/application/index.html) file (usually located in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy - e.g., `T42-DEMO`). Assign to it an object holding your custom connection properties:

```json
{
    "maxReconnectAttempts": 10,
    "reconnectPeriod": "1500(3), 15000"
}
```