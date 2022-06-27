## Introduction

This tutorial is a collection of hands-on exercises, in which we will be building a web app taking advantage of many of the Glue42 features available today. As our fictitious users start using our app, we will be getting a lot of change requests to improve the app in various ways and we will be using Glue42 to accomplish this. We will start by creating an app called **Client Portfolio** which will (initially) consist of two windows:

- a **Clients** window - showing a list of clients (**ID** and **Name**);
- a **Portfolio** window - showing the list of equities the selected client owns (**Symbol** and **Quantity** but also the **Best Bid** and **Offer** prices, if available);

When the user clicks on a client in the **Clients** window, the selected client's portfolio will be loaded in the **Portfolio** window.
As we progress through the tutorial, we will be prompted to add many more cool features like support for tab windows, handling and triggering window events, custom search and notifications, dynamic method discovery and many more.

## Prerequisites

This tutorial assumes you are a JavaScript developer or a full stack web developer with sufficient knowledge of core JavaScript (ECMAScript 5 is sufficient). No knowledge of modern web frameworks such as React or Angular is assumed or required. No prior experience with Glue42 is expected but we advise you to take a look at the following resources, which we consider useful to developers:

[Glue42 Documentation](../../../getting-started/what-is-glue42/general-overview/index.html)

[Glue42 Reference](../../../reference/glue/latest/glue/index.html)

And a few resources related to core JavaScript, if not already familiar with these:

[Array.filter](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

[Array.map](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

[Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)

We will be working with our latest development drop at the time of writing this tutorial, but the information here will be updated as new features are implemented.

*There is also a .NET version of the tutorial available [here](../net/index.html). A Java tutorial can be produced on demand.*

## Tutorial Structure

The tutorial code is available for download at [GitHub](https://github.com/Glue42/js-tutorial). Clone the repo and you will get the latest version of the tutorial code. Checkout the branch named **GD3-Start** to begin.

The tutorial directory is organized as follows:
- **app** - put the files you will be working on here. More on that in **Chapter 1**;
- **lib** - all of the browserified [`glue.js`](../../../reference/glue/latest/glue/index.html) files;
- **resources** - a sleek Tick42 icon;
- **solution** - the full tutorial solution;
- **start** - the startup app skeleton;
- **support** - all of the support apps you will need to complete the tutorial, bundled with the corresponding configuration files;
- **util** - a simple image to base64 converter;

Glue42 for JavaScript is also available as versioned NPM packages (and also runs in Node.js), but to keep things simple, in this tutorial we are going to use the latest development version, browserified and ready to be included in your HTML pages. The tutorial is broken into several parts, each demonstrating different Glue42 capabilities, and each part will depend on the completion of the previous ones. We will try to minimize the amount of boilerplate code not related to Glue42, so we will have all HTML, CSS and some JavaScript files already coded and ready to use. Plain HTML and JavaScript are used throughout the tutorial to avoid confusion with other technologies. The coding you are required to do is marked with `TUTOR_TODO` comments, so make sure you search for all of these in all parts of the tutorial. Even though each chapter depends on the previous one, we have provided all the necessary support files, apps and code from the beginning, so you can start and complete the tutorial with your own additions to the starter skeleton, without losing progress.

Each part will try to introduce you to just one Glue42 concept, in as little detail as necessary to complete it. You can get a lot more detailed information about the Glue42 APIs in the [API guides](../../../glue42-concepts/glue42-toolbar/index.html) and [references](../../../reference/glue/latest/glue/index.html).

Naturally, we have also included a full tutorial solution, which follows the app structure set in the startup skeleton. Also included are partial tutorial solutions at the start of each task to get you going. If you get stuck and wish to start with certain tasks completed, simply checkout the branch with name corresponding to the chapter and task number you want to start from.

## 1. Setting Up

### 1.1. Getting Started with the Tutorial Files

Getting started with the Glue42 for Javascript tutorial is very easy. The first thing you need to do is get all of the tutorial code. The latest tutorial version is available at [GitHub](https://github.com/Glue42/js-tutorial), so just clone the repo and you will get everything you need to get started. Next, run `npm install` to get all the tutorial dependencies:

```bash
cd /*path-to-tutorial*
npm install
```

As we will be building a web app, we will need a server. Actually, we will need two servers - one will host our apps and one will supply us with fake data, so we can have something to work with. We have set this up for you, so just run `npm start`. This will launch a simple **HTTP server**, which will host the tutorial directory at `http://localhost:22909/`. You will also get a simple **REST server** at port **22910**. After that, you need to start up Glue42 Enterprise. The [**Glue42 Enterprise**](https://glue42.com/enterprise/) setup is outside the scope of this tutorial, so we assume you already have it installed and configured. If you don't have it yet, download and install it from [here](https://glue42.com).

### 1.2. Deploying and Publishing Your App

In the `start` folder you will find all the files (HTML and JavaScript) you will be working on. Together they form the tutorial startup skeleton. Begin by copying these files into the `app` directory.

It is a good idea to read more about [Glue42 initialization and configuration](../../../getting-started/how-to/glue42-enable-your-app/javascript/index.html) before you start with this part of the tutorial.

Now, we need to initialize Glue42. **It is very important how you configure Glue42.** The defaults (e.g. passing an empty configuration object) should be sufficient for most apps. However, we have prepared a Glue42 configuration object in `lib/tick42-glue/tick42-glue-config.js`. You will find `TUTOR_TODO Chapter 1.2 Task 1` in `portfolio.html` and `TUTOR_TODO Chapter 1.2 Task 2` in `clients.html`, where you will be prompted to include the Glue42 configuration object and the Glue42 library. Once you have included the configuration and the library in both HTML files, you will be able to initialize Glue42 and add the variable `glue` to the global namespace. To do this, go to `clients.js`, find the `TUTOR_TODO Chapter 1.2 Task 3` and:

- call the Glue42 factory function
- pass the config object from `lib/tick42-glue/tick42-glue-config.js`
- when the `Promise` is resolved, assign the received `glue` instance to the window object (global namespace)
- call these five functions, which will bootstrap the app and define the skeleton of the app:
	- `checkGlueConnection()`;
	- `setUpUi()`;
	- `setupClients()`;
	- `registerGlueMethods()`;
	- `trackTheme()`;
- Finally, don't forget to catch and log any errors.

Now find `TUTOR_TODO Chapter 1.2 Task 4` in `portfolio.js` and do the same. It is pretty much identical, apart from the functions you need to call in the `then()` statement.

**Publishing the App**

The app has initialized Glue42 and is deployed (`http-server`), now you should publish it, so it is available to your users in the Glue42 Toolbar. In this tutorial, we will explain and work with the **file configuration** mode.

*We have a session dedicated to on-boarding your apps, where we will show you how to prepare and manage configurations in the Glue42 Configuration Manager using an utility and a set of use case oriented batch files.*

You can read more about App Configuration in the [Configuration documentation](../../../developers/configuration/application/index.html) which explains in detail how the [**Glue42 Enterprise**](https://glue42.com/enterprise/) configurations work. For this part of the tutorial, all you need to know is that:

- [**Glue42 Enterprise**](https://glue42.com/enterprise/) components (windows and native executables) are configured in JSON files, where the most important part is the URL (or path) of the component.
- The configuration files are stored in `%LocalAppData%\Tick42\GlueDesktop\config`.

**Note:**  Configuration changes are detected in real-time, so once you change a configuration file, you don't need to restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) to detect the changes.

To publish the app:

- Copy the `tutorial-*-applications.json` files (from `/start/configs/chapter-1`) to the app configuration folder (`%LocalAppData%\Tick42\GlueDesktop\config\apps`).
- If you have followed the above instructions and copied the files to the `/app` directory, you wouldn't need to make any changes. If, on the other hand, you decided not to place them in `/app`, then you need to modify the `tutorial-*-applications.json` files so that they contain the correct **Clients** and **Portfolio** windows URLs.

Once you publish your apps to Glue42 Enterprise, you should be able to launch any of them. Open the Developer Console (`F12`) and type `glue` or `glue.info` to verify Glue42 has been loaded.

You should verify that everything works:

- Make sure both **Clients** and **Portfolio** appear in the Glue42 Toolbar.
- Click on both apps to launch the windows.
- Verify that **Clients** loads a sample list of clients.
- Both windows should have a green field in the top left corner saying **Glue is available**.

![Glue42 JavaScript Tutorial Chapter 1](../../../images/tutorials/enterprise-js/1.2.gif "Glue42 JavaScript Tutorial Chapter 1")

Here are a few things that might have gone wrong:

1. "I don't see the apps": make sure you have copied the JSON files to the correct configuration folder and the files from `/start` to `/app`.
2. "I see the apps, but when I click them, I see a 'Page Not Found' message": make sure you have run `npm install` and `npm start`, which will fire up the server you need. Also, double check the URLs in the configuration files - the apps are trying to reach the files in the `/app` directory of the tutorial. Make sure to either put your files there or change the URL to look in `/start`.
3. "I have the apps and both windows are loading, but **Clients** doesn't display any clients": you need to run `npm start`, which will start the REST server that provides the fake data.
4. "I don't have an Glue42 Toolbar": you need to run the [**Glue42 Enterprise**](https://glue42.com/enterprise/) app.
5. "Pages load but nothing works": hit `F12` and have a look at the console output for any errors.
6. "Something else happened": let us know, and we will update this tutorial section as we resolve it.

## 2. Interop Methods

### Overview
Now that we have the apps defined, it is time to "glue" them together before we show them to our users. In this part, we will "listen" for party changes in the **Portfolio** window by **registering an Interop method** and will trigger such a change from the **Clients** window by **invoking an Interop method**. In order to complete this part, you should have the **Interop** [documentation](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html) and [reference](../../../reference/glue/latest/interop/index.html) at hand.

### 2.1. Registering Methods

We will start with the **Portfolio** window, where we will demonstrate **registering an Interop method** (`SetParty()`), which, when called, will trigger the window to re-load the portfolio for the selected **party** (client).

Since different apps might be using different party identifiers, when we define a party, we will define it as a `Composite` and use the party ID's as members of this composite parameter. We also need to check if this hasn't been already defined, and either re-use it directly, if it contains all we need, or consult with the other developer teams to include our party ID.

Assuming we are the first to propose the **party** object, our method registration will have:

- name: `"SetParty"`;
- display name: `"Set Party"`;
- description: `"Switches the app window to work with the specified party"`;
- will accept a `Composite` parameter called `party` with 2 optional `party` identifiers (strings) - `pId` and `salesForceId`, of which we will only use the `pId` in this tutorial;

**Note:** Only the **name** property and the **method handler** are required during registration but the best practice is to always populate the **display name** and **description**, because as we will see later, the display name can actually be shown in a context menu, and the description can be used both as a tool tip, and as documentation for our colleagues.

Your task is to register `SetParty()` - in `portfolio.js`, look for `TUTOR_TODO Chapter 2.1` and do that. Once your method is invoked, call `loadPortfolio()` and pass the `pId` of `party`. You may want to read about [Interop Method Definition](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_registration-method_definition) before you start implementing this part.

**Note**: You don't need to move to the **Clients** window and code the method invocation there in order to test the method. Verify that it works by invoking `SetParty()` from the **Developer Console**. Simply typing `SetParty()` in the console won't work, because `SetParty()` isn't a method that is defined in the global namespace. `SetParty()` is a special Interop method and should be invoked via the API:

```javascript
const invokeArgs = {
    party: client
};
glue.interop.invoke("SetParty", invokeArgs);
```
This means that once you have agreed upon the Glue42 interface with your colleagues, you can test your windows without having to wait for the rest of the windows to be developed.

### 2.2. Invoking Methods

Now that our **Portfolio** window responds correctly to party changes, let's switch to the **Clients** window and trigger these party changes by **invoking** the Interop method `SetParty()` and passing the selected **client** as a `party` object.

Open `clients.js` and find `TUTOR_TODO Chapter 2.2` to add the code which performs the method invocation. You should pass the `client` object for the `party` argument. You may need to check the [Interop Method Invocation](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_invocation) for more information.
A common mistake developers make when trying to invoke a method with a `Composite` parameter typically looks like this:

```javascript
const party = { pId: 123, salesForceId: 456 };
glue.interop.invoke("SetParty", party);
```

The problem is that the invocation expects an object containing the method parameters, so passing the `party` object really instructs Interop to send `pId` and `salesForceId` as the two arguments of the invocation. You need to pass the `party` parameter wrapped in the request object, like so:

```javascript
glue.interop.invoke("SetParty",
    {                       // start of parameters
        party: {            // just one parameter - party, which is a composite
            pId: 123,
            salesForceId: 456
        }
    });
```

You should test whether the method invocation works. Again, you don't need to run the **Portfolio** window to do so. All you need is this method to be available (but don't care which app offers it), so you can **register a dummy method implementation** in the Developer Console (`F12`). Try that, run the **Clients** window, register the method in the Developer Console and check if the invocation works.

Now that we have tested whether `SetParty()` is correctly registered in the **Portfolio** window, and whether we are invoking it correctly from the **Clients** window, we can expect that the windows can interoperate. Launch both windows and verify that clicking a **client** from the **Clients** window loads and displays the client's **portfolio** in the **Portfolio** window.

![Glue42 JavaScript Tutorial Chapter 2.2](../../../images/tutorials/enterprise-js/2.2.gif "Glue42 JavaScript Tutorial Chapter 2.2")

By default, [**Glue42 Enterprise**](https://glue42.com/enterprise/) windows are **sticky** (which can be controlled by the configuration setting or at runtime, when opening a window), so try sticking the two windows together.

### 2.3. Object Types and Method Discovery

Some of our users have [Thomson Reuters Eikon](https://eikon.thomsonreuters.com/index.html)  installed, while others have [Bloomberg Terminal](https://www.bloomberg.com/professional/solution/bloomberg-terminal/). Both are software products providing a set of financial tools used to see news and various charts about certain **instruments**. Users say they have to constantly switch between apps, copying and pasting ticker symbols from our app to Eikon or Bloomberg. Both terminal apps have a Glue42 add-in which registers methods like **Show News** and **Show Chart** and we can easily offer these in our **Portfolio** window, but we would rather not hardcode the logic to show these actions in the **Portfolio** window of our app, especially since some users may not have any of these apps installed.

In Glue42, the concept of context-sensitive methods works by setting up a context (which we call **Object Type**) when you register a method. The idea is that one app registers a method(s) tagged with one or more object types, and then another app(s) can dynamically discover (at runtime) the method(s) for a particular **object type**.

We will extend the **Portfolio** window to use **object type** method discovery to figure out which methods are available for a particular instrument (the **object type** will be "Instrument").

Search for the `TUTOR_TODO Chapter 2.3 Task 1` in `portfolio.js`. Your first task is to extend the logic of the `row.onclick` event. When a **row** (**instrument**) is clicked, you should find all registered methods (`glue.interop.methods().filter()`) with an "Instrument" object type and then invoke `addAvailableMethods(*found methods*, rowData.RIC, rowData.BPOD)`. Now when an **instrument** is clicked, a modal window will appear listing the currently registered methods.

Your second task is to locate `TUTOR_TODO Chapter 2.3 Task 2` in the `invokeAgmMethodByName()` and implement the logic for invoking an Interop method with the passed `methodName` and `params`. You may need the documentation for [Object Type Targeted Method Invocation](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#object_types).

To verify that everything works, we have registered two methods with an `Instrument` object type in the provided code. You should be able to see them and call them when clicked.

Now, when users click on an **instrument** in the **Portfolio** window, we are going to show them the **display names** of the available methods, and when they click on a method, we will invoke the method, passing the selected **instrument**.

![Glue42 JavaScript Tutorial Chapter 2.3](../../../images/tutorials/enterprise-js/2.3.gif "Glue42 JavaScript Tutorial Chapter 2.3")

We are excited we have a decent proof of concept, so we are going to show it to our clients and see what they have to say.

## 3. Interop Streams

In this part we will use the [Interop Streaming](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#streaming) API to add real-time market data to our app.

Our users say they mostly use Eikon to see real-time market data. They say that if we can bring the **best bid and offer** (BBO) prices in our app, they will have a lot more screen real estate.

For the purpose of this tutorial, we have provided a Sample Price Publisher (`/support/sample-price-publisher`), which streams fake **bid and offer prices**. Before you proceed, make sure you copy the config file (`/support/tutorial-sample-price-publisher-applications.json`) to the app configuration folder and launch **SamplePricePublisher** from the Glue42 Toolbar, just like you did with **Portfolio** and **Clients**.

Your task is to implement the `subscribeBySymbol()` and the `unsubscribeSymbolPrices()` functions. Open `portfolio.js` and find `TUTOR_TODO Chapter 3 Task 1`. In `subscribeBySymbol()` you need to subscribe to a stream named `T42.MarketStream.Subscribe` (created by the Sample Price Publisher), which expects a single subscription argument called `Symbol` (string).

When the `Promise` returned by [`interop.subscribe()`](../../../reference/glue/latest/interop/index.html#API-subscribe)  is resolved, get the subscription and subscribe to its [`onData`](../../../reference/glue/latest/interop/index.html#Subscription-onData) event, and upon receiving the data, simply invoke the callback passed to `subscribeBySymbol()`, which will do the rest.

Next, in `portfolio.js` find `TUTOR_TODO Chapter 3 Task 2`. In `unsubscribeSymbolPrices()` you need to traverse the currently active subscriptions and close each one. This is necessary, because when we load a new portfolio, we want to clear the existing symbol subscriptions and subscribe to the new ones.

As a result, the **Bid** and **Ask** cells should be updated every second with a new value.

You may need the [Interop Stream Subscription](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#streaming-subscribing_to_a_stream) documentation. You can also have a look at the [Stream Publishing](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#streaming-publishing_streams) API and our implementation of the Sample Price Publisher.

![Glue42 JavaScript Tutorial Chapter 3](../../../images/tutorials/enterprise-js/3.gif "Glue42 JavaScript Tutorial Chapter 3")

## 4. Window Management

### Overview

Glue42 windows can be extensively customized via:

- [Glue42 Window configuration](../../../reference/glue/latest/windows/index.html#WindowSettings) - at [**Glue42 Enterprise**](https://glue42.com/enterprise/) deployment time;
- [Glue42 Window Groups configuration](../../../reference/glue/latest/windows/index.html#Group) - at [**Glue42 Enterprise**](https://glue42.com/enterprise/) deployment time;
- [App configuration](../../../developers/configuration/application/index.html) - at app deployment time;

Once created, the developer is in full control of the windows using the [Window Management](../../../glue42-concepts/windows/window-management/javascript/index.html) API.

### 4.1. Opening Windows at Runtime

The users are starting to like our app, but they are saying it is inconvenient having to push two buttons in the Glue42 Toolbar, then grab the **Portfolio** window and stick it to the **Clients** one. They also say that they always put the **Portfolio** window below the **Clients** one.

In this chapter we will first remove **Portfolio** from the Glue42 Toolbar, then add a button on the **Clients** window, and finally use the **Window Management** API to open the **Portfolio** window and stick it below the **Clients** window. You may need the [Windows Management API guide](../../../glue42-concepts/windows/window-management/javascript/index.html) and [reference](../../../reference/glue/latest/windows/index.html) in order to implement this part.

Your task is to remove the **Portfolio** app from the Glue42 Toolbar and use the **Window Management** API to open it directly from the **Clients** window, instructing [**Glue42 Enterprise**](https://glue42.com/enterprise/) to position and stick it below the **Clients** window.

Begin by modifying the **Portfolio** JSON file in the [**Glue42 Enterprise**](https://glue42.com/enterprise/) configuration folder:

```json
{
    "hidden": true  // this hides the Portfolio app from the Glue42 Toolbar
}
```

Next, in `clients.html` find `TUTOR_TODO Chapter 4.1 Task 1` and uncomment the button tag. In `clients.js`, find `TUTOR_TODO Chapter 4.1 Task 2` in the `setUpUi()` function. Invoke the `openWindow()`, pass as arguments a name for the window ("Portfolio", for example), reference to the current window ([`glue.windows.my()`](../../../reference/glue/latest/windows/index.html#API-my)) and a direction (`"bottom"`). It is best to use this function, because we will be calling it from multiple places throughout this tutorial. Then, in `openWindow()`, find `TUTOR_TODO Chapter 4.1 Task 3` and create an `options` object, which you will use to set three important settings:

```javascript
{
    mode: "flat"
    relativeTo: myWin.id
    relativeDirection: direction
}
```

The combination of [`relativeTo`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeTo) and [`relativeDirection`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeDirection) allows us to stick the newly created **Portfolio** window to the bottom of the current window.

**Note:** We will use the window [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id) as the value for the [`relativeTo`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeTo) property, not the entire window reference.

Finally, use the [**Window Management**](../../../reference/glue/latest/windows/index.html#API) API to open a new **Portfolio** window:

```javascript
glue.windows.open(
    windowName,
    window.location.href.replace("clients.html", "portfolio.html"),
    options);
```
![Glue42 JavaScript Tutorial Chapter 4.1](../../../images/tutorials/enterprise-js/4.1.gif "Glue42 JavaScript Tutorial Chapter 4.1")

**Note:** Opening windows using the **Window Management** API is OK, but, ideally, you would want to use the [App Management](../../../glue42-concepts/application-management/javascript/index.html) API to manage your apps and subscribe to app/instance related events. We will take a deeper look at the [App Management](../../../glue42-concepts/application-management/javascript/index.html) API in **Chapter 5**.

### 4.2. Window Styles

The users loved that but said they still see two separate windows on the screen and when they close the **Clients** window, the **Portfolio** one will still hang there. They asked us if we could make it close together with the **Clients** window. Also, they complain that they can shrink the windows as much as they want and would prefer, if we restricted both windows to a certain minimum size. Finally, they don't like the "white spot" on the window which stays until the data is loaded.

In this chapter we will use **Window Styles** to convert the windows to borderless HTML windows, and we will remove the window controls (minimize, restore and close) from the **Portfolio** window. We will set sizing restrictions on both windows, and we will also start listening for Glue42 lifecycle events ([`glue.windows.windowRemoved()`](../../../reference/glue/latest/windows/index.html#API-windowRemoved)) in the **Portfolio** window so that when the **Clients** window is closed, we will close the **Portfolio** one as well.

Finally, we will use loader animations to show the default loading animation (feel free to change it) while we are loading data.

The first thing you need to do, is to go to the **Clients** app JSON file (in the [**Glue42 Enterprise**](https://glue42.com/enterprise/) configuration folder) and add or modify:

```json
{
    "details": {
        "minHeight": 400,
        "minWidth": 600,
        "mode": "html",
    }
}
```

Then open the **Portfolio** app JSON file and add or modify the following:

```json
{
    "details": {
        "allowClose": false,
        "allowCollapse": false,
        "allowMaximize": false,
        "allowMinimize": false,
        "minHeight": 400,
        "minWidth": 600,
        "mode": "html",
    }
}
```

These settings will make our app feel more like a single app, instead of two separate apps and will also set the window restrictions we were asked to do.

 In `clients.js` find `TUTOR_TODO Chapter 4.2 Task 1` - you will need to tweak the `options` object to set various `allow` attributes and dimensions. Your task is to make sure that the newly created **Portfolio** window:
- is in HTML mode;
- can't be minimized, maximized, collapsed or closed;
- has minimum height of 400 and minimum width of 600 pixels;

The [window settings documentation](../../../glue42-concepts/windows/window-management/javascript/index.html#window_settings) may come in handy here.

**Important note!**
*You may have noticed that we are configuring the **Portfolio** window twice (once in the config file and once from the `options` object in our code). The reason is that for the time being we are using the **Window Management** API to open a new window/app. When we do this, the app config file isn't taken into account. The proper way to open an app is by using the **App Management** API, which we will take a look at in **Chapter 5**. When we start using the **App Management** API, our settings in the config file will be taken into account, but we could still override them with our `options` object.*

Finally, in order to detect when the **Clients** window is closed, you will use a **Window Management** API event ([`windowRemoved()`](../../../reference/glue/latest/windows/index.html#API-windowRemoved)), so that when **Clients** is closed, you will close the **Portfolio** window as well.
But how would you know which window is the **Clients** window? The user could be running multiple instances, so you can't simply test whether the name of the window starts with `clients_window`. So, we need to pass the instance (window [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id)) of the **Clients** window to the **Portfolio**. One way would be to pass it in the URL, but what if we want to pass more complex data (objects, arrays or a mix of them)?
Glue42 allows one window to pass a context (JSON-serializable object, a non-cyclic directed graph) to another window, via configuration (in the JSON file) and programmatically (in the [`glue.windows.open()`](../../../reference/glue/latest/windows/index.html#API-open) call). So, you will need to pass your window [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id) as the initial context to the **Portfolio** window you are opening, so that in the **Portfolio** window you can subscribe for [`glue.windows.windowRemoved()`](../../../reference/glue/latest/windows/index.html#API-windowRemoved) and close the **Portfolio** window when the **Clients** window is closed. Create a `context` attribute (object) in the `options` object, and set any properties you want, e.g. `parentWindowId: yourWindowId`.
After that, in `portfolio.js` find `TUTOR_TODO Chapter 4.2 Task 2` in the `setUpWindowEventsListeners()` function. You have to subscribe for [`glue.windows.onWindowRemoved()`](../../../reference/glue/latest/windows/index.html#API-onWindowRemoved) and in the callback, check whether the [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id) attribute of the passed window object matches the [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id) which you passed in the initial window context ([`glue.windows.my().context`](../../../reference/glue/latest/windows/index.html#GDWindow-context)), and if they do, close the **Portfolio** window ([`glue.windows.my().close()`](../../../reference/glue/latest/windows/index.html#GDWindow-close)).
As a last touch, go to `clients.js` and add the loader animations in the `getClients()` function, where you will find `TUTOR_TODO chapter 4.2 Task 3` -  for showing the loader, and `TUTOR_TODO chapter 4.2 Task 4` - for hiding it. Show the loader before you initiate the request ([`glue.windows.my().showLoader()`](../../../reference/glue/latest/windows/index.html#GDWindow-showLoader)), and hide it ([`glue.windows.my().hideLoader()`](../../../reference/glue/latest/windows/index.html#GDWindow-hideLoader)) in the [**always**](http://api.jquery.com/jquery.ajax/) event of the `$.ajax` `Promise`.

![Glue42 JavaScript Tutorial Chapter 4.2](../../../images/tutorials/enterprise-js/4.2.gif "Glue42 JavaScript Tutorial Chapter 4.2")

### 4.3. Tabbed Windows

The users say it would be great if they can see multiple client portfolios at once. They aren't sure how this will work, because they worry about screen real estate.

In this chapter we will use the **Tabbed Windows** feature to extend our app. We will keep the option to open a generic **Portfolio** window below the **Clients** window, but we will add the option to stack individual **Portfolio** tabs to the right of the **Clients** window. The idea here is to create a tab frame the first time we select a **client**, put the **Portfolio** window in it and every time the user clicks on a different **client**, we will add another window as a tab in the same tab frame. We are also going to name them after the selected **client** by using the `setTitle()` method on the window object, and also demonstrate passing context from the **Clients** to the **Portfolio** window, something which is unique to Glue42 Enterprise.

Now, find the `TUTOR_TODO Chapter 4.3 Task 1` in `clients.js`, inside the `openTabWindow()` function. We call this function every time a **client (row)** is clicked (the same way we invoke an Interop method) and pass the selected **client**. You should also check if the **client**'s tab is already opened - if so, you should [`activate()`](../../../reference/glue/latest/windows/index.html#GDWindow-activate) it.

Opening a tab window means that you will have to use a different window mode in the window style attributes - `tab`. In order to put all tabs in a single tab frame, you will also need to set the [`tabGroupId`](../../../reference/glue/latest/windows/index.html#WindowSettings-tabGroupId) to the same value for all tab windows - it could be any string, e.g. `PortfolioTabs`.

```javascript
{
    mode: "tab",
    tabGroupId: "PortfolioTabs"
}
```

To put the **Portfolio** windows on the right of the **Clients** window, make sure you pass the **Clients** window [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id) in the [`relativeTo`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeTo) attribute, and set [`relativeDirection`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeDirection) to `"right"`. Note that only the first **Portfolio** tab needs the [`relativeTo`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeTo) and [`relativeDirection`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeDirection) attributes. Provided that consequent tabs specify the same [`tabGroupId`](../../../reference/glue/latest/windows/index.html#GDWindow-tabGroupId), they will be grouped together. The documentation for [finding windows](../../../glue42-concepts/windows/window-management/javascript/index.html#finding_windows) may help you here.
Finally, in order to pass the selected `party` to the newly created **Portfolio** tab, use the [`context`](../../../reference/glue/latest/windows/index.html#GDWindow-context) object to pass the `party` object.

Next, in `portfolio.js`, navigate to `TUTOR_TODO chapter 4.3 Task 2` in the `setUpAppContent()` method. Up until now we registered the Interop methods, but now we may not need to do that. If the current **Portfolio** window is created as a tab, we shouldn't register the method, because this will break our idea of individual portfolio windows. Achieving this effect is pretty easy:

- Get the window context ([`glue.windows.my().context`](../../../reference/glue/latest/windows/index.html#GDWindow-context)).
- Check if it contains the passed `party` object.
- If it doesn't, this means that the **Portfolio** window was opened by clicking the button at the bottom of the **Clients** window, in which case let the code register the Interop method for handling `party` changes.
- If it does have a `party` object, this means that this **Portfolio** window is for a specific **client**, and in that case you don't want to listen for changes, because, if you do, all tabs will display the same data. Instead:
	- get the `party` object `preferredName` attribute and use that to set the title of the tab (using [`glue.windows.my().setTitle`](../../../reference/glue/latest/windows/index.html#GDWindow-setTitle));
	- assign the same title using `document.getElementById("title").textContent()`;
	- call `loadPortfolio()` passing the `pId` attribute of the `party` object;

![Glue42 JavaScript Tutorial Chapter 4.3](../../../images/tutorials/enterprise-js/4.3.gif "Glue42 JavaScript Tutorial Chapter 4.3")

### 4.4. Custom Window Frame Buttons

Our users loved the tabs so much that they want us to extend this functionality. First of all, they would like a bit more flexibility when positioning the tabs and the **Portfolio** window. Preferably, as windows open, the app should "just stack them to the bottom" of the **Clients** window, but if there is no space, it should "just stack them to the right" of the window.
Also, our users hate the big **Portfolio** button and want something more subtle which doesn't take that much space.
Furthermore, they said that it would be great if we could somehow allow them to scatter the tabs as individual windows so that they can move them around freely, but also have a convenient way to gather them back in a frame and stick this frame back to **Clients**.

Yes, that is too much. But Glue42 can enable your app to do all of the above (and more), so let's tackle this problem one piece at a time.

**Adding a "Subtle" Button to Open the Selected Client's Portfolio**

The **Window Management** API supports adding what we call [frame buttons](../../../glue42-concepts/windows/window-management/javascript/index.html#frame_buttons). You can add, remove and handle clicks on any number of buttons. A frame button is called so, because it is added to the **frame** (top-most, non-content portion) of the current window.

In `clients.js`, inside the `setUpUi()` function, find `TUTOR_TODO Chapter 4.4 Task 1` and create a frame button. Then subscribe to frame button click events and open a **Portfolio** window when your newly created button is clicked. Note that the image for the button must be in **base64** and if you are wondering how to convert an image to **base64**, you can use the `util\convert-image-to-base64.html` "utility" page to create one. If you don't feel like creating your own button image, you can just uncomment and use the provided `buttonOptions` object.

Now would be a good time to delete or comment-out the big blue **Portfolio** button from `clients.html`, because we have a much nicer-looking frame button.

**Stacking Portfolio Windows**

The trickier part now is to start stacking **Portfolio** windows either at the bottom or at the right of the current window. Stacking is easy, you could use [`snap`](../../../reference/glue/latest/windows/index.html#GDWindow-snap) or [`relativeTo`](../../../reference/glue/latest/windows/index.html#WindowSettings-relativeTo), but first you have to check whether the window will be off the bounds of the screen.
The idea here is to snap the **Portfolio** window or tab group to the bottom of the **Clients** window, if there is enough space on the screen, otherwise stick it to the right of the **Clients** window.
The Glue42 Browser injects several useful objects into each window. One of them is `glue42gd.monitors`. This returns an array of objects describing each physical monitor on the user's desktop. For the tutorial, assume the user is running on the primary one (the `isPrimary` property of the `monitor` object is set to `true`). Once you have identified the primary monitor, you can use the `workingAreaHeight` property, which gives you the usable height of the monitor (minus the task bar).
Now, you can check if the bottom of your window (top + height) + the height of the **Portfolio** window (the default height is `400` but we will expose an `originalHeight` on the window object) is less than the working area height.

The **Window Management** API offers a property called [`bottomNeighbours`](../../../reference/glue/latest/windows/index.html#GDWindow-bottomNeighbours) (returns an array of all windows attached to the bottom of the current window) for each window object, so for the purpose of this tutorial, simply check if the **Clients** window has any bottom neighbors. If it doesn't **and** if it passes the above check, snap the **Portfolio** window or tab group (depending on what is clicked) to the bottom (you can re-use the `openWindow()` function and pass `"bottom"`), otherwise pass `"right"`.

Locate `TUTOR_TODO Chapter 4.4 Task 2` in `getWindowDirection()` in `clients.js` to get the primary monitor and return a `direction`. Then, in `TUTOR_TODO Chapter 4.4 Task 3` pass the `direction` to `openTabWindow()` as a second argument, instead of the hardcoded `"right"`.
In `TUTOR_TODO Chapter 4.4 Task 4` use the **Window Management** API for handling frame button clicks to handle a frame button click, check the `id` and open a **Portfolio** window. Get the direction and pass it as a third argument to `openWindow()`.

![Glue42 JavaScript Tutorial Chapter 4.4](../../../images/tutorials/enterprise-js/4.4.4.gif "Glue42 JavaScript Tutorial Chapter 4.4")

**Gather and Scatter Windows**

This is a bit more involved, but cool as well. Check out the GIF at the end of this section to get a better idea of the task at hand.

The idea here is to add a frame button to the **Portfolio** tab window, which will either scatter the grouped tabs into separate windows, or will gather them into a tab frame, if they are scattered. So this actually requires two frame buttons, and you will be showing only one at a time, depending on whether you have more than one **Portfolio** tab in the tab frame. If there are two or more tabs in the group, you need to show the **Extract** button, otherwise show the **Gather** button.

In `portfolio.js`, in the `setUpTabControls()` function, find `TUTOR_TODO Chapter 4.4 Task 5`. You should first check if your window is tabbed ([`glue.windows.my().mode`](../../../reference/glue/latest/windows/index.html#GDWindow-mode) is `tab`) and if not, you shouldn't do anything else.
In `TUTOR_TODO Chapter 4.4 Task 6`, if the window is a tab, you should create the **Gather** and **Extract** buttons, similarly to the way we created the frame button in the **Clients** app. Simply uncomment the buttons. You should also subscribe to the following events:

| Event | Description |
|-------|-------------|
|[`onWindowAttached()`](../../../reference/glue/latest/windows/index.html#GDWindow-onWindowAttached)|A window (not the current one) has been attached to the current window tab frame.|
|[`onWindowDetached()`](../../../reference/glue/latest/windows/index.html#GDWindow-onWindowDetached)|A window (not the current one) has been detached from the current window tab frame.|
|[`onAttached()`](../../../reference/glue/latest/windows/index.html#GDWindow-onAttached)|The current window has been "tabbed"|
|[`onDetached()`](../../../reference/glue/latest/windows/index.html#GDWindow-onDetached)|The current window has been "untabbed"|

You should handle all these events and decide which button (**Scatter** or **Gather**) to create or remove in each scenario. Also, before adding or removing a button check if it is present or not.
In `TUTOR_TODO Chapter 4.4 Task 7` you should implement logic to decide which button was clicked and find a way to keep track of detached tabs.
In `TUTOR_TODO Chapter 4.4 Task 8` you should use the **Window Management** API and decide which button should be rendered - **Gather** or **Scatter**.

Now, for the actual implementation of scattering/gathering windows. The idea is simple:
- When the **Scatter** button is clicked, we should go through all tabbed windows and call [`detachTab()`](../../../reference/glue/latest/windows/index.html#GDWindow-detachTab) on each of them, and we also need to keep track of all windows that were detached (so we can gather them later).
- When the **Gather** button is clicked, we should go through all detached windows and gather them in a tab frame - technically, this means snapping the first window to the **Clients** window, then attaching the rest of the detached windows to this first tab.

So, where do we keep the state - the windows that were detached and that we will later need to be re-attached? This needs to happen at a "central" place, but until you have learned about [Shared Contexts](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/javascript/index.html), we can keep them in the **Clients** window itself, or more specifically - in its context.
Yes, in addition to passing a context to a window during creation, you can also update a window context at runtime by calling [`window.updateContext()`](../../../reference/glue/latest/windows/index.html#GDWindow-updateContext). So, you can keep the detached windows (only their [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id)'s) in an object with a single property `detachedTabs`, or whatever you like, which will be an array holding the window [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id)'s of the detached tabs.
You already have the **Clients** window [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id), because in the previous chapters you passed it upon creating the **Portfolio** window. Now, the [`glue.windows.findById()`](../../../reference/glue/latest/windows/index.html#API-findById) method will come in handy for getting a reference to the window itself by its [`id`](../../../reference/glue/latest/windows/index.html#GDWindow-id).

So, **Scatter** is clicked, go over all tabs ([`glue.windows.my().tabs()`](../../../reference/glue/latest/windows/index.html#GDWindow-tabs)), add them to an array and detach them. Then store the array in the **Clients** window context.
Finally, upon clicking **Gather**, get the detached windows from the **Clients** window context (`someWindow.context`), find the first window, snap it to the right of the **Clients** window, and then go over the rest of the detached windows and call `firstWindow.attachTab(detachedWindowId)`.

![Glue42 JavaScript Tutorial Chapter 4.4](../../../images/tutorials/enterprise-js/4.4.8.gif "Glue42 JavaScript Tutorial Chapter 4.4")

## 5. App Management

We are making a good progress with our app, but now is a good moment to take a step back and improve on something. Up until this moment we have been using the **Window Management** API to open apps. This works fine, but it isn't the proper way to manage apps in Glue42 Enterprise. Now, we are going to take a look at the **App Management** API and see how and why we should use it to open our apps.

As we previously discussed, opening apps via the **Window Management** API completely bypasses the settings in the JSON file. This leads to some confusion and possibly mistakes. Opening an app via the **App Management** API ensures that our settings will be taken into account and we can still override them in our code (by providing a [`windowSettings`](../../../reference/glue/latest/windows/index.html#GDWindow-tabs) object).

By using the **App Management** API you don't need to know the URL of the app, which, in cases where there are multiple environments and regions, is very helpful and ensures you will launch the correct app.

Furthermore, the user might not be entitled to open a given app and using the **Window Management** API bypasses the entitlement checks done by Glue42, only to fail later when the newly opened window loads.

Modifying your code to use the **App Management** API is quite easy. In `clients.js` locate the `TUTOR_TODO Chapter 5 Task 1` inside `openWindow()`. Here you will need to make two objects: one is the `context` and the other one is the `windowSettings` object. Basically, the second one should contain everything we currently have in the `options` object, apart from the `context` itself. After that, call the start of an app using the proper API: `glue.appManager.application("Portfolios").start(context, windowSettings)`.

You can also remove the window style properties like [`minHeight`](../../../reference/glue/latest/windows/index.html#WindowSettings-minHeight) and [`allowMinimize`](../../../reference/glue/latest/windows/index.html#WindowSettings-allowMinimize) and set them in the configuration file.

Next, in the `openTabWindow()` function find `TUTOR_TODO Chapter 5 Task 2` and go through the same process as in `Task 1`.
Note that you can't re-use the filtering you (likely) did in `openTabWindow()`, because you can't set the window name. So, you need to get creative. What else is unique to the tabs that we can use as a filtering condition?

## 6. Glue42 Search Service

Our users say some **instruments** don't have a ticker symbol, but either CUSIP, or ISIN, or SEDOL, and they would be very interested in a solution which lets them search by **any** instrument identification code when they are managing a client's portfolio.

In this chapter we will use the [Glue42 Search Service (GSS)](../../../reference/gss/latest/gss/index.html) and [Client Search APIs](../../../reference/gss/latest/gss/index.html#API) to offer this functionality. We don't have the actual search back-end yet, but that is fine because we have sample Java and JavaScript **GSS publishers** which we can feed with sample data. Switching search providers is a matter of configuration, so our app is actually oblivious of what the real data source is.

Your task here is to allow the user to search for **instruments** and add them to the **portfolio**. For this, you will be using GSS.

In this tutorial we will be using the [GSS simple wrapper](../../../reference/gss/latest/gss/index.html) API.

You will need a supporting app which is the sample JavaScript Instrument GSS provider. The configuration file is named `tutorial-instrument-provider-applications`, it is located in the `support` directory and you should copy it to `%LocalAppData%\Tick42\GlueDesktop\config\apps`.

The Instrument GSS Provider will start automatically when you launch [**Glue42 Enterprise**](https://glue42.com/enterprise/) and it should just display "Sample GSS Publisher".

In order to implement the search you will need the documentation for the [GSS Client Search API](../../../reference/gss/latest/gss/index.html#API), which is a wrapper on top of the GSS API. It is named `ClientSearch` for historical reasons but can in fact work with any entity type, as you will see.

Now, switch to your code and add the `gss-client-search.js` (located in `lib\tick42-gss`) to your `portfolio.html` in `TUTOR_TODO Chapter 6 Task 1 `.

Find `TUTOR_TODO Chapter 6 Task 2` in the `initInstrumentSearch()` function. You need to:
- create a search client - `new gssClientSearch.create(gssOptions)`;
- create a query and save it - `createdClient.createQuery("Instrument")`;
- subscribe to the query [`onData`](../../../reference/gss/latest/gss/index.html#GssQuery-onData) event and pass the received data to `displayResult()`;

Finally, in the `search()` function you need to find `TUTOR_TODO Chapter 6 Task 3` and use the created query, call its search method and pass the `searchValue`.

When you are ready, your apps should look like the example below. Type part of a symbol which you know exists (e.g., we are using "B" in the example, because we know there is "Barclays" and "BMW") and hit the **Search** button (no auto-completion is required for the tutorial). You should see a pop up with the GSS results displayed. Click on a search result **symbol** and it will be added to the portfolio.

![Glue42 JavaScript Tutorial Chapter 6](../../../images/tutorials/enterprise-js/6.gif "Glue42 JavaScript Tutorial Chapter 6")

You are ready. Try playing with the `ClientSearch` options to see how data comes in and out.

## 7. Glue42 Notification Service

The market data team is now sending GNS notifications when a major financial event occurs, which requires advisors to call clients who can lose (or gain) a lot of money, because of a sudden market or instrument price change (e.g., Apple buys Microsoft, or a Grexit/Brexit). Our users are very happy with the notifications, but say that as we own the client portfolios, we surely know who the affected clients are, and wonder if we can do something about it.

We ask the market data team to add [Glue42 Routing](../../../glue42-concepts/notifications/javascript/index.html#glue42_routing) to their [GNS](../../../glue42-concepts/notifications/javascript/index.html) notifications so we can trap these on the desktop, instead of in the generic GNS notifications UI. They will route the notification so that when it arrives on the user's desktop, the user will still be able to see the notification toast, but when they click it, it will take them to our new "who-to-call" window instead of the generic notification details window.

In this chapter, your task is to handle notifications from your app by registering a Glue42 Routing handler for GNS notifications in your app.

In order to complete this task, you will need a supporting app (GNS publisher) which is called **Market Data Monitor**. From it you can manually raise notifications which you will see in the GNS UI. The configuration file `tutorial-market-data-monitor-applications.json` can be found in the `support` directory. You should copy it to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) app config folder (`%LocalAppData%\Tick42\GlueDesktop\config\apps`). Then, in order to register **Market Data Monitor** as a GNS publisher, you should copy this configuration to `%LocalAppData%\Tick42\GnsDesktopManager\Config\overrides.gnsDesktopManager.LOCAL-GD3-GLUE4OFFICE.properties`, but be careful not to replace any of its previous contents.

```properties
gns.providersRepository.MarketData.type=AgmServer
gns.providersRepository.MarketData.enabled=true
gns.providersRepository.MarketData.agmServer.serverIdentity.machineName=
gns.providersRepository.MarketData.agmServer.serverIdentity.applicationName=MdMonitorJsAgmGnsServer
gns.providersRepository.MarketData.agmServer.serverIdentity.user=
gns.providersRepository.MarketData.agmServer.serverIdentity.environment=
gns.providersRepository.MarketData.agmServer.serverIdentity.region=
gns.providersRepository.MarketData.query.types=
gns.providersRepository.MarketData.query.attributes=
gns.providersRepository.MarketData.query.states=
gns.providersRepository.MarketData.query.limit=
gns.providersRepository.MarketData.query.lastSequenceId=
gns.providersRepository.MarketData.query.lastModifiedSince=
gns.providersRepository.MarketData.query.includePayload=
```

*You should check out the source code of this app, after you are done with the demo, to understand how you can create a GNS provider on the desktop.*

Note that the **Market Data Monitor** will start automatically. That is one of the features of Glue42 Enterprise.

You should now implement the `TUTOR_TODO Chapter 7` in `clients.js`. In the handler of `g42.FindWhoToCall()`, open the provided `symbolPopup.html` using [`glue.windows.open()`](../../../reference/glue/latest/windows/index.html#API-open), passing the `symbol` attribute from the invocation arguments in the context.

When you are done, launch the GNS UI and raise a notification from the **Market Data Monitor**. You should see a toast at the bottom right corner of your screen, and also see the notification in the GNS UI.
Double-click the notification. Instead of showing the details of the notification, the GNS UI will follow the routing and call your `g42.FindWhoToCall()` method. When this happens, your task is done and your app will show a popup with the affected clients (hardcoded), which looks like this:

![Glue42 JavaScript Tutorial Chapter 7](../../../images/tutorials/enterprise-js/7.gif "Glue42 JavaScript Tutorial Chapter 7")

## 8. Glue42 Outlook Connector

This tutorial assumes that you already have [**Glue42 Enterprise**](https://glue42.com/enterprise/) Office Connectors installed, but you can find the documentation [here.](../../../connectors/general-overview/index.html)

Knowing our users spend a considerable amount of time working with Outlook, where they usually have to copy and paste data from our apps, we will try to surprise them by adding the ability to compose an e-mail at the click of a button, right from our web app.

First, you should include `glue4office-dev.js` (located in `lib/tick42-g4oe`) in `portfolio.html` with `TUTOR_TODO Chapter 8 Task 1`. Then you need to uncomment the `Send as e-mail` button from `TUTOR_TODO Chapter 8 Task 2`, which, when clicked, will send the data the user is looking at in a new message, along with an attachment of the raw data, ready to be imported and analyzed into other apps, such as Excel.

Next, you will need to enable the Glue42 Outlook Connector by running `_EnableGlueOutlook.cmd` in `%LocalAppData%\Tick42\GlueOutlook`. Exit Outlook before attempting to install the connector.

You should then implement the `TUTOR_TODO`'s in `portfolio.js`. There are some instructions and you can also view the [Glue4Office Outlook API](../../../connectors/ms-office/outlook-connector/javascript/index.html).

Begin by initializing [`Glue4Office`](../../../reference/glue4office/latest/glue4office/index.html) in `TUTOR_TODO Chapter 8 Task 3` and `TUTOR_TODO Chapter 8 Task 4`. This is done pretty much the same way as when initializing Glue42 itself, but with additional options, which you will find in `portfolio.js`. The important part here is to pass the current Glue42 instance to `Glue4Office`, because otherwise it will create its own which will cause conflicts. The resolved `Promise` will return a [`Glue4Office`](../../../reference/glue4office/latest/glue4office/index.html) instance, which you should assign to the window object, just like you did with Glue42.

You don’t need to actually build a portfolio table using HTML, just prepare some HTML, and some CSV data for the attachment. We have prepared all the email content for you, but feel free to explore it.

Finally, in `TUTOR_TODO Chapter 8 Task 5`, compose a new email ([`g4o.outlook.newEmail()`](../../../reference/glue4office/latest/outlook/index.html#API-newEmail)) passing the `content` object.

Now run Outlook. The connector runs within Outlook, and it will start it for you. You can read about how to configure an app to start automatically with [**Glue42 Enterprise**](https://glue42.com/enterprise/) in the [App Configuration Guide](../../../developers/configuration/application/index.html).

When you are ready, you should be able to compose a new email from Glue42 within Outlook.

![Glue42 JavaScript Tutorial Chapter 8](../../../images/tutorials/enterprise-js/8.gif "Glue42 JavaScript Tutorial Chapter 8")

## 9. Glue42 Excel Connector

Some of our users are power Excel users and they have asked us if we could somehow let them export the data from our app into Excel for some last minute modifications and then get it back into our app to be submitted to the back-end.
We can use the Glue42 Excel Connector which gives us exactly that functionality.

Your first task, `TUTOR_TODO Chapter 9 Task 1`, is to uncomment the `Send to Excel` button in `portfolio.html`. Next, in `portfolio.js` you need to enable Excel in the [`Glue4Office`](../../../reference/glue4office/latest/glue4office/index.html) config object in `TUTOR_TODO Chapter 9 Task 2`. Then in `TUTOR_TODO Chapter 9 Task 3` create an Excel spreadsheet ([`g4o.excel.openSheet()`](../../../reference/glue4office/latest/excel/index.html#API-openSheet)), passing the supplied config object.

As a finishing touch, when the `Promise` is resolved, you will receive a reference to the new sheet so you can subscribe to its [`onChanged()`](../../../reference/glue4office/latest/excel/index.html#Sheet-onChanged) event. When that is invoked, pass the new data to the `loadPortfolioFromExcel()` method.

As a result, your app will be able to send the current portfolio to Excel when the button is clicked and will update the portfolio when the user modifies and saves the data in Excel.

![Glue42 JavaScript Tutorial Chapter 9](../../../images/tutorials/enterprise-js/9.gif "Glue42 JavaScript Tutorial Chapter 9")

## 10. Shared Contexts

Some of our users are looking at the Reuters Eikon all day and it has a dark background. They are complaining that our app, and apps from other teams, are giving them a headache because they are using a light theme. They would be very thankful if we did something about it.

After some brainstorming with developers from the other teams, we have decided to implement a light and a dark theme. However, users want to switch to either the dark or the light theme in all apps at the same time and would hate to do that in each app.

Using Glue42 [Shared Contexts](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/javascript/index.html), we will broadcast the theme change to all apps on the user's desktop. When our apps start, they will query the theme name by reading the shared context and will automatically switch to the currently used theme.

In this assignment, you will need to add theme support in your app. We have already developed the theme switching app for you. It isn't a big deal, just a couple of calls, but feel free to check out its code.

- Deploy the theme switching app in [**Glue42 Enterprise**](https://glue42.com/enterprise/) (copy the `tutorial-theme-chooser-applications.json` file from the `support` directory) to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) `config` folder.
- Switch to your code and find `TUTOR_TODO Chapter 10 Task 1` in `clients.js` and `TUTOR_TODO Chapter 10 Task 2` in `portfolio.js`. Subscribe for context changes (the context event is called `theme`) in both tasks. You need to check the `name` property in the context change you receive - if it is `dark`, you will need to call `setTheme("bootstrap-dark.min.css")`, and if it is `light`, call `setTheme("bootstrap.min.css")`.
- When you are ready, you will be able to switch the theme from the theme switching app (so make sure you run that first).

Here is what your app will look like after you have implemented the functionality and have pressed the “Dark” button in the theme switching app.

![Glue42 JavaScript Tutorial Chapter 10](../../../images/tutorials/enterprise-js/10.gif "Glue42 JavaScript Tutorial Chapter 10")

## 11. Metrics and Logging

An operative just emailed us about a pilot user claiming that when he selects a **client**, either the client's **portfolio** won't load at all or will take too long to load. He can't remember which **client** that was.

In this chapter, using the [Metrics](../../../glue42-concepts/metrics/javascript/index.html) API and [reference](../../../reference/glue/latest/metrics/index.html), we will make our app:

- add an **Error Count** metric to record the number of times our AJAX requests have failed;
- add a **Time Span** metric to capture the latency of the requests;
- create a **Composite** metric to record the client ID, the time and error message of the last service call exception, if any. Here, the initial value defines the shape (template) of the metric;
- put the above metrics under a metrics sub-system and set its state to **RED** if service goes down, **AMBER** if service is taking too long, and **GREEN** if everything is well;

We will also put some [logging](../../../reference/glue/latest/logger/index.html) around the service call, which (for the purposes of this tutorial) will log to the developer console (`F12`). However, you can easily log to a file on the user's desktop so that we don't need to run analysis on the recorded metrics, if we happen to reproduce this on our end.

In order to complete this chapter you need to create a sub-logger instance and use it in the `loadPortfolio()` method - write to the console (info) every time the request is successful. Find `TUTOR_TODO Chapter 11 Task 1` in `portfolio.js` and create the sub-logger. Then in `TUTOR_TODO Chapter 11 Task 2` log with `logger.info` and the provided `logMessage`.

Create a metrics instance, a sub-system, and set the state to `GREEN` in `TUTOR_TODO Chapter 11 Task 3`. Assign it to `serviceMetricsSystem`.

Prepare the error count metric in `TUTOR_TODO Chapter 11 Task 4` and assign it to `serviceErrorCount`. Check out the [Metrics](../../../reference/glue/latest/metrics/index.html) API and see what `countMetric()` does.

In `TUTOR_TODO Chapter 11 Task 5` assign a new [`objectMetric`](../../../reference/glue/latest/metrics/index.html#ObjectMetric) to `lastServiceError`.

Lastly, assign to `serviceLatency` a new [`timespanMetric`](../../../reference/glue/latest/metrics/index.html#TimespanMetric) in `TUTOR_TODO Chapter 11 Task 6`.

Now that our metrics are set up, we should use them to change system state, if needed, increment error counts and so on:
- As the portfolio starts loading, we should start the latency metric. Do this in `TUTOR_TODO Chapter 11 Task 7` by calling its [`start()`](../../../reference/glue/latest/metrics/index.html#TimespanMetric-start) method.
- The latency metric should be stopped as soon as the load succeeds (`TUTOR_TODO Chapter 11 Task 8`) or fails (`TUTOR_TODO Chapter 11 Task 9`).
- Now, assuming the load was successful, we need to alert if the load was too slow. If so, set the system state to **AMBER** in `TUTOR_TODO Chapter 11 Task 10`. Otherwise, set the system state to **GREEN** in `TUTOR_TODO Chapter 11 Task 11`.
- Assuming the loading failed - we have already stopped the latency metric, but we have yet to increment the error count. Do this in `TUTOR_TODO Chapter 11 Task 12`. If the loading failed, we should also update the `lastServiceError` in `TUTOR_TODO Chapter 11 Task 13` and set the state of the system to **RED** in `TUTOR_TODO Chapter 11 Task 14`.

## Congratulations!

You should now have the foundation to start working on Glue42 enabling your real apps.

Next step?

Build awesome Glue42 enabled apps!!