## Overview

In this quick tutorial you will learn how to:

- Install and use **Spectron**.
- Use **Mocha** and **Spectron** to write end-to-end tests to test your app in [**Glue42 Enterprise**](https://glue42.com/enterprise/).

*You can get the example app and test configuration used in this guide from [GitHub](https://github.com/Glue42/glue42-spectron-example).*

## Installing Mocha and Spectron

### Mocha

To install the Mocha testing framework:

```cmd
npm install mocha
```

*For more info on how to setup and use Mocha, see the [official Mocha documentation](https://mochajs.org/)*

### Spectron

**Spectron** is a testing tool that connects to [WebdriverIO v4](http://v4.webdriver.io/api.html) in order for you to be able to access the DOM elements. Install **Spectron** version 13.0.0 by executing the following command from the command prompt:

```cmd
npm install spectron@13.0.0
```

## Test Guide

*See the full example below for reference to the test steps described here.*

Open the `test.js` file with your favorite editor to start writing your test:

1. First, you need to import the `Application` module from **Spectron**:

```javascript
const Application = require("spectron").Application;
```

2. Then, import the `assert()` function from Node.js:

```javascript
const assert = require("assert");
```

3. The main block of your tests is the `describe()` block, which contains the actual test cases. The `describe()` in **Mocha** is used to unify common test cases. You can have nested `describes()`, which is convenient for organizing complex test scenarios. The `describe()` should contain a general description of the test case group and a `function()` in which you write your tests:

```javascript
describe("My Test App", function() {
    // test block 1
    // test block 2
});
```

Define an `app` variable and set a timeout for the tests (the default Mocha timeout is 2000 ms, which is insufficient for starting Glue42 Enterprise, starting your app and executing the tests):

```javascript
describe("My Test App", function() {

    let app;
    this.timeout(20000)

    // test block 1
    // test block 2
});
```

4. Use a `before()` and an `after()` blocks to instruct Mocha what to do before and after executing all tests:

- In the `before()` block, use the imported `Application` module to create a new `Application` instance object and assign it to the `app` variable. You have to pass some necessary options to the newly created `Application` object:
    - `path` - the path to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) EXE file on your machine;
    - `cwd` - the current working directory to be used for the launched application;
    - `args` - in this property you must pass all the overrides required for [**Glue42 Enterprise**](https://glue42.com/enterprise/) to start only one app (besides the splash screen window);

*There are other `Application` options you can pass upon creating a new application instance - for more info, see the relevant [Spectron documentation](https://github.com/electron-userland/spectron#application-api).*

The function in the `before()` should return the started app instance:

```javascript
before(function () {
        app = new Application({
            path: `${process.env.LOCALAPPDATA}/Tick42/GlueDesktop/tick42-glue-desktop.exe`,
            cwd: `${process.env.LOCALAPPDATA}/Tick42/GlueDesktop`,
            args: [
                "-- config=%LOCALAPPDATA%/Tick42/GlueDesktop/config/system.json",
                "--noSplash",
                "--singleApp=%LOCALAPPDATA%/Tick42/UserData/T42-DEMO/apps/js-test-app.json"
            ],
        });

        return app.start()
            .catch(console.error)
    });
```

- In the `after()` block, you need to close the app:

```javascript
after(function() {
    if (app && app.isRunning()) {
        return app.stop();
    }
});
```

5. Now, use the `it()` block to write your test. The `it()` block should have a description of the specific test case and, again, a `function()` which contains the test logic. The basic skeleton of your test should now look like this:

```javascript
describe("My Test App", function() {
    let app;
    this.timeout(20000)

    before(function() {
        // start your app and get the correct window reference
    });

    after(function() {
        // close your app
    });

    it("Should have the correct title", function() {
        // actual test
    });
});
```

The `app` object has a `client` property which is exposed from the WebdriverIO v4 API. Use `app.client.getTitle()` to get the title of your app and assert whether it is the correct one:

```js
it("Should have the correct title", async function() {
    const title = await app.client.getTitle();

    assert.equal(title, "My Test App");
});
```

6. Run the `npm test` command to execute the `mocha` script command, which in turn will execute the test and visualize the results:

```cmd
npm test
```

### Test Example

```js
const Application = require("spectron").Application
const assert = require("assert");

describe("My Test App", function () {

    // SETUP section

    let app;

    this.timeout(20000)

    before(function () {

        // get reference to Glue42 Enterprise and configure it to start a single app (your app)
        app = new Application({
            path: `${process.env.LOCALAPPDATA}/Tick42/GlueDesktop/tick42-glue-desktop.exe`,
            cwd: `${process.env.LOCALAPPDATA}/Tick42/GlueDesktop`,
            args: [
                "-- config=%LOCALAPPDATA%/Tick42/GlueDesktop/config/system.json",
                "--noSplash",
                "--singleApp=%LOCALAPPDATA%/Tick42/UserData/T42-DEMO/apps/js-test-app.json"
            ],
        });

        // start Glue42 Enterprise and your app before all tests
        return app.start()
            .catch(console.error)
    });

    // shutdown Glue42 Enterprise after all tests
    after(function () {
        if (app && app.isRunning()) {
            return app.stop();
        }
    });

    // TESTS section

    // test 1
    it("Should have the correct title", async function () {
        const title = await app.client.getTitle();

        assert.equal(title, "My Test App");
    });

    // test 2
    it("Should get a URL", async () => {
        const title = await app.client.url("https://docs.glue42.com/")
            .getTitle();

        assert.equal(title, "Glue42 Documentation");
    });
});
```

<!-- TODO test to replace the second test when the new docs are deployed

it("Should get the correct URL and element", async () => {
    const title = await app.client.url("https://docs.glue42.com/")

    app.client.click(".grid-item a");

    assert.equal(title, "Glue42 Documentation");
}) -->