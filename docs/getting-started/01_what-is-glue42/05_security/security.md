## Authentication

There are several aspects to authentication:

1. Authentication for applications running within [**Glue42 Enterprise**](https://glue42.com/enterprise/) for the purpose of SSO

In most cases, [**Glue42 Enterprise**](https://glue42.com/enterprise/) is transparent with regard to authentication and just serves as an augmented container for the applications to run in. For example:
- Cookie based SSO - cookies can be shared between applications;
- Token based SSO - applications can use shared contexts to share the tokens; 
- NTLM/Kerberos - supported directly by the underlying Chromium engine;
                
[**Glue42 Enterprise**](https://glue42.com/enterprise/) can also provide a custom login screen where more customization is needed, or when the SSO flow needs to be “forced” before the first application is loaded.

2. Authentication for remote application stores    
 
In enterprise scenarios, the [application store](../../../glue42-concepts/application-management/overview/index.html#application_stores) would be deployed on the same infrastructure as the applications themselves and protected in the same manner. [**Glue42 Enterprise**](https://glue42.com/enterprise/) can use the same authentication mechanisms as in the previous case, or mix and match - i.e., applications can use one type of authentication and application stores (for example when deployed in a public cloud)
can use different ones. 
 
3. Glue42 Gateway authentication

If configured, each Glue42 Gateway connection can require transport level authentication as well. When a connection is established, the client will need to provide authentication details to the Glue42 Gateway which will be verified against an authentication provider and, as a result, the identity of the client will be established. The Glue42 Gateway will inject the identity of every client when passing information about it to other clients.
The Glue42 Gateway itself acts as an Identity Provider - once a client is authenticated, it can request a crypto token to pass around to other clients that will assume the identity of the originator. This allows for SSO in cases where the authentication provider requires interactive authentication - when [**Glue42 Enterprise**](https://glue42.com/enterprise/) is started, it will show a login dialog, establish a session with the Glue42 Gateway and from that point on, use its IdP API to generate tokens for each application that is started by the launcher.
The authentication provider support for the Glue42 Gateway is extensible and comes with the following implementations out of the box:
- No authentication
- Auth0/OpenID Connect
- Windows Integrated Authentication (NTLM/Kerberos)
                
In addition, if required, the Glue42 Gateway can provide connection filtering based on IP address and origin headers.

The authentication is configured via the `authentication` key under `gw\configuration` in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/), which is located in `%LocalAppData%\Tick42\GlueDesktop\config`:

```json
"gw":{
    "configuration": {
        ...
        "authentication": {
            "available": ["basic", "win"], 
            "default": "win"
        }
    }
}
```

The available authentication types are `basic` and `win`. If a type is invalid or not specified, the authentication will fall back to `basic`.

When using `win` authentication, you also have to configure the `auth` top-level key in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) in the following way:

```json
"auth": {
    "authController": "sspi"
}
```

## WebSocket Connection Filtering

The Glue42 Gateway supports connection filtering based on the origin header of the connection. The configuration resides under the `gw` top-level key of the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/), which is located in `%LocalAppData%\Tick42\GlueDesktop\config`. The following properties are available under `gw\configuration\security\origin_filters`:

| key        | type    | default | description                   |
|------------|---------|---------|-------------------------------|
| `whitelist` | array of strings  | empty | List of strings or regex patterns (strings starting with `#`) that allow an origin |
| `blacklist`   | array of strings | empty | List of strings or regex patterns (strings starting with `#`) that block an origin |
| `non_matched` | `"whitelist"` or `"blacklist"` | `"whitelist"` | Action to take if the origin does not match the white or the black list. |
| `missing` | `"whitelist"` or `"blacklist"` | `"whitelist"` | Action to take if the origin header is missing |

Example:

```json
"gw": {
    "configuration": {
        ...
        "security": {
            "origin_filters": {
                "missing": "whitelist", // native connections missing an origin header are allowed
                "whitelist": ["#https://.*\\.websocket.org", "#chrome-extension://.*"], // only accept connections from the test site and the chrome extensions
                "non_matched": "blacklist"
            }
        }
    }
}
``` 