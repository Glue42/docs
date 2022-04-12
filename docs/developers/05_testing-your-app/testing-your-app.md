## Overview

There are two general approaches for testing Glue42 enabled applications depending on whether you are an individual app owner or a team within your organization responsible for the Glue42 integration flow.

Application owners can test their Glue42 apps by mocking the Glue42 methods used in them with the help of any familiar testing framework.

For end-to-end automation tests, it is recommended to use the [Playwright](https://playwright.dev/) testing tool. The next section details a sample test case using Playwright.

## Testing with Playwright

[Playwright](https://playwright.dev/) allows you to create end-to-end automation tests for [**Glue42 Enterprise**](https://glue42.com/enterprise/) and your Glue42 enabled applications.

*For more in-depth information on using Playwright, see the [Playwright official documentation](https://playwright.dev/docs/intro).*

*See the full Glue42 [Playwright example](https://github.com/Glue42/playwright-example) on GitHub.*

#### Testing Example

The following test example demonstrates how to start [**Glue42 Enterprise**](https://glue42.com/enterprise/) and wait for it to load, then open the Applications view, search for the Glue42 Dev Tools application, start it and wait for it to load.

1. Import the required test objects and define the [**Glue42 Enterprise**](https://glue42.com/enterprise/) working directory and the path to the Glue42 executable file:

```javascript
const { _electron: electron } = require("playwright")
const path = require("path");
const { test, expect } = require("@playwright/test")

// Make sure these paths correspond to your specific Glue42 deployment.
const gdDir = `${process.env.LocalAppData}\\Tick42\\GlueDesktop\\`;
const gdExePath = path.join(gdDir, "tick42-glue-desktop.exe");
```

2. Create a helper function that will wait for a Glue42 app to load:

```javascript
// The function will receive the Glue42 app name and the Electron app object as arguments.
const waitForAppToLoad = (appName, electronApp) => {
    return new Promise((resolve, reject) => {
        electronApp.on("window", async (page) => {
            try {
                const glue42gd = await page.evaluate("glue42gd");

                // Compare the received app name with the actual app name and resolve on load if they match.
                if (appName === glue42gd.application) {
                    page.on("load", () => {
                        resolve({
                            app: glue42gd.application,
                            instance: glue42gd.instance,
                            glue42gd,
                            page
                        });
                    });
                };
            } catch (error) {
                // Do nothing.
            };
        });
    });
};
```

3. Create the test unit:

```javascript
// Timeout for the test.
test.setTimeout(60000);

test("Launch Dev Tools from the Glue42 Application Manager and wait for it to appear.", async () => {
    // Step 1: Start Glue42 Enterprise.
    const electronApp = await electron.launch({
        executablePath: gdExePath,
        cwd: gdDir
    });

    // Step 2: Wait for the Glue42 Application Manager to appear.
    const { page } = await waitForAppToLoad("glue42-application-manager", electronApp);

    // Step 3: Click on the element with an "apps" ID to expand the Applications view.
    await page.click("id=apps");

    // Step 4: Type "dev tools" in the app search field.
    await page.type("id=app-search", "dev tools");

    // Step 5: Click on the result.
    await page.click("id=search-results");

    // Step 6: Wait for the Dev Tools app to appear.
    await waitForAppToLoad("DevTools", electronApp);
});
```