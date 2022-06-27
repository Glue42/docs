## Introduction

This guide explains how to reference and initialize the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) library in your web app so it can interoperate with any of the MS Office products supported by Glue42 Enterprise.

After completing the steps in this guide, you can read the respective development guides to learn how your app can interact with specific MS Office products.

## Prerequisites

### Programming Experience with JavaScript

We assume you are a front-end JavaScript developer, or a full-stack developer with sufficient JavaScript knowledge. This guide uses ECMAScript 6 (ES6) in the examples, but ES6 knowledge isn't required to use the API.

All APIs return [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). If you aren't comfortable using `Promise`s and prefer Node.js-style callbacks, you can use [nodeify](https://github.com/then/nodeify) to convert `Promise`s to Node.js-style callbacks.

### Glue42 Enterprise Installation

You can download the [**Glue42 Enterprise**](https://glue42.com/enterprise/) trial version from [here](https://glue42.com/free-trial/).

## Referencing

The [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) library is available both as an NPM module and as a standalone transpiled JavaScript file in `%LocalAppData%\Tick42\GlueSDK\Glue4OfficeJS\js\web-bundle`.

You can reference the library in a `<script>` tag:

```html
<script type="text/javascript" src="office.min.js"></script>
```

The browserified JavaScript file is also a CommonJS module, which you can `require`/`import`.

CommonJS:

```javascript
const Glue4Office = require("glue4office");
```

ES6:

```javascript
import Glue4Office from "glue4office";
```

## Initialization

When you reference the standalone JavaScript file, it will expose a global factory function called [`Glue4Office`](../../../../reference/glue4office/latest/glue4office/index.html#Glue4Office).

If you have imported or required the library, the function will be available with the name you specified in the `import` or `require` statement.

Assuming you stick to the default `Glue4Office` name, to initialize the library, you need to call the function [Glue4Office](../../../../reference/glue4office/latest/glue4office/index.html#Glue4Office) and specify which MS Office apps you want to interoperate with. The function returns a `Promise`, which will resolve with an initialized instance of the library.

Example:

```javascript
const g4oConfig = {
    application: "MS Office Interop",
    excel: true,        // enable Excel interop
    word: true,         // enable Word interop
    outlook: false      // disable Outlook interop
}
Glue4Office(g4oConfig)
    .then(g4o => {
        // g4o is a reference to the Glue4Office API
        window.g4o = g4o;    // expose g4o as a global variable
        // use g4o
    })
    .catch(console.error)
```

If your app is already loading Glue42, you can initialize [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) by passing an [initialized Glue42 library instance](../../../../getting-started/how-to/glue42-enable-your-app/javascript/index.html) in the configuration object:

```javascript
const glueConfig = { application: "MS Office Interop" }
Glue(glueConfig)
    .then(glue => {
        window.glue = glue;          // expose glue as a global variable
        const g4oConfig = {
            glue,
            excel: true,            // enable Excel interop
            word: true,             // enable Word interop
            outlook: false          // disable Outlook interop
        }
        return Glue4Office(g4oConfig)
    })
    .then(g4o => {
        // g4o is a reference to the Glue4Office API
        window.g4o = g4o;    // expose g4o as a global variable
        // use g4o
    })
    .catch(console.error)
```

Once you have obtained an initialized reference, you can start interacting with the MS Office connectors:

```javascript
g4o.excel.openSheet(...);

g4o.word.openDocument(...);

g4o.outlook.newEmail(...);
```

The best place to render your app is when the `Promise` is resolved (in the `then` clause above). You would typically expose a global reference to the library and then render your app.

## React Example

```javascript
Glue4Office(g4oConfig)
    .then(g4o => {
        window.g4o = g4o;        // expose g4o as a global var
        ReactDOM.render(
            appElement,
            document.getElementById("root"));
    });
```

## Angular Example

```javascript
Glue4Office(g4oConfig)
    .then(g4o => {
        window.g4o = g4o;        // expose g4o as a global var
        platformBrowserDynamic().bootstrapModule(AppModule);
        // or
        platformBrowser().bootstrapModuleFactory(AppModuleNgFactory);
    });
```