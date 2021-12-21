## Quick Guide

To Glue42 enable your app, you need to do 3 things:

- reference the respective Glue42 library (JavaScript, .NET, Java, etc.);
- initialize the Glue42 library;
- create an application definition and add it to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) configurations folder;

The following is an example of configuring a simple web app. Create your own basic web app or use this [JavaScript test app skeleton](../../../../assets/js-test-app.zip) and follow the steps in the guide below to test in practice how to Glue42 enable an application and add it to the Glue42 Application Manager. If you choose to use the provided test app skeleton, see the README file for instructions on how to run it.

*See also how to reference and initialize Glue42 in a [.NET](../net/index.html), [Java](../java/index.html), [React](../react/index.html), [Angular](../angular/index.html), [Node.js](../nodejs/index.html) or [VBA](../vba/index.html) environment.*

1. Reference the Glue42 library file in a `<script>` tag. If you are using the example app skeleton provided above, the Glue42 library file is is named `desktop.umd.js`:

```html
<script type="text/javascript" src="desktop.umd.js"></script>
```

2. Initialize the Glue42 library.

To initialize the Glue42 library, place this code snippet in a `<script>` tag directly in the `index.html` or in the `index.js` file (which you have to reference in your `index.html`):

```javascript
const init = async () => {
    window.glue = await Glue();
};

init().catch(console.error);
```

Your `index.html` should now look something like this:

```html
<body>
    <h1>My Test App</h1>
    <div>
        <p>
            Some Data
        </p>
    </div>
</body>
<script type="text/javascript" src="desktop.umd.js"></script>
<script>
    const init = async () => {
        window.glue = await Glue();
        console.log(`Glue42 initialized successfully!\nGlue42 version: ${glue.version}`);
    };

    init().catch(console.error);
</script>
```

3. Create an application definition.

To add your application to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) Application Manager, you need to create an application configuration JSON file. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` should be replaced by the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). If you are using the provided test app, you can simply copy the provided `js-test-app.json` configuration file to that location. Otherwise, you can see below how to create a quick and simple application definition:

```json
{
    "name": "test-app",
    "title": "My Test App",
    "type": "window",
    "details": {
        "url": "http://localhost:3333/index.html",
        "mode": "tab",
        "width": 500,
        "height": 400
    }
}
```

The `"name"`, `"type"`, `"details"` and `"url"` properties are required.

4. Start your application.

Now you can start your application from the [**Glue42 Enterprise**](https://glue42.com/enterprise/) Toolbar. You can open the console of the app window by pressing `F12` to see the printed successful initialization message. You can also start exploring the Glue42 APIs by accessing the `glue` object that was attached to the global `window` object during initialization.

That's it! You now have your first Glue42 enabled app!

For more advanced configuration settings, visit the [Configuration](../../../../developers/configuration/application/index.html) section.

See also:

- The full [Tutorials](../../../../tutorials/javascript/index.html) on integrating [**Glue42 Enterprise**](https://glue42.com/enterprise/) functionalities in your apps.
- The [Examples](../../../../developers/examples/index.html) section for example [**Glue42 Enterprise**](https://glue42.com/enterprise/) functionality implementations and example apps, developed for testing purposes.
- Get acquainted with the main [Glue42 Concepts](../../../../glue42-concepts/glue42-toolbar/index.html) and learn how to work programmatically with the [**Glue42 Enterprise**](https://glue42.com/enterprise/) platform.
- [Glue42 JavaScript Reference](../../../../reference/glue/latest/glue/index.html) 