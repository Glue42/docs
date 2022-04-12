## Floating Toolbar

The source code for the Glue42 [Floating Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html#floating_toolbar) is [available on GitHub](https://github.com/Glue42/toolbar). You can use it as a template for creating your own custom toolbar.

## Launchpad

The Glue42 [Launchpad](../../../../glue42-concepts/glue42-toolbar/index.html#launchpad) is available as a React Component on NPM - [`@glue42/launchpad-ui-react`](https://www.npmjs.com/package/@glue42/launchpad-ui-react). You can use the component in your own apps and also modify it by passing different options. For more details, see the README file of the [`@glue42/launchpad-ui-react`](https://www.npmjs.com/package/@glue42/launchpad-ui-react) package.

## Custom Toolbar

Once you have implemented a toolbar, you must host it, create an [application configuration](../../../../developers/configuration/application/index.html#application_configuration) file for it and add it to your application store. Make sure to set the `"shell"` top-level key to `true`:

```json
{
    "shell": true
}
```

Modify the [system configuration](../../../../developers/configuration/system/index.html) of [**Glue42 Enterprise**](https://glue42.com/enterprise/) from the `system.json` file located in `%LocalAppData%\Tick42\GlueDesktop\config`. Set the `"useEmbeddedShell"` property to `false`:

```json
{
    "useEmbeddedShell": false
}
```

Restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) for the changes to take effect.