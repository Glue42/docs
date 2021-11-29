## Overview 

A mock implementation of the Glue42 library is useful when you want to test your Glue42 enabled apps outside the Glue42 environment (e.g., in a browser). The [glue-js-mock](https://github.com/Tick42/glue-js-mock) repository offers an implementation of a Glue42 mock, a sample Glue42 enabled app for testing purposes and test implementations. The repo has the following structure:

- `/mock` - Glue42 mock that is used to mimic the [Glue42 JavaScript library](../../../reference/glue/latest/glue/index.html) in a testing environment;
- `/src` - the source of a sample Glue42 enabled app;
- `/test` - [Puppeteer](https://developers.google.com/web/tools/puppeteer) tests of the sample app;

*[Puppeteer](https://developers.google.com/web/tools/puppeteer) is used to test the sample Glue42 enabled app. However, the Glue42 mock is not tied up to Puppeteer and can be used with other tools as well.*

## Glue42 Mock

The Glue42 mock is a partial implementation of the [Glue42 JavaScript API](../../../reference/glue/latest/glue/index.html) used in testing to mimic the Glue42 APIs.

Currently, the Glue42 mock has partial support for:

- [Interop](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html)
- [Shared Contexts](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/javascript/index.html)
- [Channels](../../../glue42-concepts/data-sharing-between-apps/channels/javascript/index.html)
- [Glue42 Window context](../../../glue42-concepts/windows/window-management/javascript/index.html#context)

Move the Glue42 mock code in your application and extend/modify it to support your app in a better way.

## Detecting the Environment

To detect whether your app is running in Glue42, check for the `glue42gd` object attached to the global `window` object:

```javascript
// main.ts
import GlueFactory, { Glue42 } from "@glue42/desktop";
import { MockGlueJS } from "../mock/mock";

...

if (window.glue42gd) {
    // Running in Glue42.
    glue = await GlueFactory();
} else {
    // Not running in Glue42.
    glue = new MockGlueJS() as unknown as Glue42.Glue;
};
```

## Testing

The [glue-js-mock](https://github.com/Tick42/glue-js-mock) repository provides a [sample Glue42 enabled app](#testing-sample_glue42_enabled_app) and tests for it. For examples on how to create tests for your Glue42 enabled apps using the Glue42 mock, see the sample test files and the step-by-step [test example](#testing-test_example) below.

To execute the included tests, run the following commands:

```cmd
npm install
npm test
```

*The provided tests are executed by launching a "headless" browser (no browser UI). If you want to see the results from the tests in the app, use the `headless: false` setting when launching a browser instance and comment out the `afterAll()` and `afterEach()` statements in the test files to avoid closing the browser pages and the browser itself.*

### Sample Glue42 Enabled App

The provided sample Glue42 enabled application has the following functionalities:

- Interop:
    - [registers](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_registration) a Glue42 Interop method which when [invoked](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_invocation), updates a `<div>` with the argument of the Interop call;
    - when the "Get External Data" button is clicked, an external Interop method is invoked and the result is displayed in a `<div>`;

- Channels:
    - displays the context of the currently selected color Channel in a `<div>`;
    - when the "Update Channel" button is clicked, the context of the currently selected color Channel is updated;

These functionalities showcase a real (although simplified) usage of Glue42 in web apps.

### Test Example

This example uses one of the tests for the sample Glue42 enabled app from the [glue-js-mock](https://github.com/Tick42/glue-js-mock) repo. The test verifies that clicking a button on the page updates a `<div>` with the result from invoking an external Interop method.

1. [Register](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_registration) a new Interop method that will be invoked by the web. Access the mocked Glue42 attached to the `window` object to do so:

```javascript
await page.evaluate((method, result) => {
    (window as any).glue.interop.register(method, () => {
        return result;
    });
}, INTEROP_GET_TEXT_METHOD_NAME, result);
```

2. Simulate a button click that will [invoke](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_invocation) the Interop method and update the `<div>` in the page:

```javascript
// Click the button.
await page.click("#interop-get-external-data-button");
```

3. To verify the result from the operation, extract the text from the `<div>` and compare it to the result returned by the Interop method:

```javascript
// Get the `<div>` text and compare it to the result returned by the Interop method.
const element = await page.$(`#${INTEROP_GET_TEXT_DIV_NAME}`);
const innerText = await page.evaluate(element => element.innerText, element);

expect(innerText).toEqual(JSON.stringify(result));
``` 