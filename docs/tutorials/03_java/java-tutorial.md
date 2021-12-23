## Overview

The purpose of this tutorial is to introduce [**Glue42 Enterprise**](https://glue42.com/enterprise/) and some of its features to Java developers and to show the Glue42 way of developing applications. The tutorial starts with four applications working independently of each other and when completed, the same applications will be integrated with each other and will interoperate as a unified user-friendly system.

## Prerequisites

- [**Glue42 Enterprise**](https://glue42.com/enterprise/)
- Experience with Java.
- Basic understanding of asynchronous programming.

## Resources

- See the Java examples distributed with [**Glue42 Enterprise**](https://glue42.com/enterprise/) and located in the `%LocalAppData%/Tick42/Demos/Java` folder.

- See how to [Glue42 Enable Your Java App](../../getting-started/how-to/glue42-enable-your-app/java/index.html#installation) - here you will find information on how to install, reference and initialize the Glue42 Java library, as well as how to create application configuration files for your Java apps.

- Each chapter of the tutorial provides a link to the documentation for the respective Glue42 Concept it explores.

- The reference documentation for the Glue42 Java APIs is distributed with [**Glue42 Enterprise**](https://glue42.com/enterprise/) and is located in the `%LocalAppData%/Tick42/GlueSDK/Glue42Java/docs/apidocs/` folder.

## Setup

Clone the [Java tutorial repository](https://github.com/Glue42/java-tutorial) and start [**Glue42 Enterprise**](https://glue42.com/enterprise/).

Repository structure:
- `data` - contains an `exe` which hosts all client related data on port 8083;
- `config` - contains the configuration files for all applications;
- `skeleton` - contains the skeleton of the solution in which you will start;
- `solution` - contains a full solution to the tutorial, which you can use for reference how everything should work when completed;
- `start` - the folder in which your work will begin;

The tutorial contains the following applications:
- **Clients** - Java Swing application which has information about clients and visualizes a grid with very little details;
- **Contacts** - Java Swing application which has a grid with contacts;
- **Portfolio** - Java Swing application visualizing a client's profile in detail;
- **Notifications** - Java Swing application which can raise notifications by clicking a button;

So, now you have four applications that must be integrated with [**Glue42 Enterprise**](https://glue42.com/enterprise/) in order to maximize 
the working efficiency and to improve the user experience. We can start with copying the skeleton files from `/skeleton` to `/start`.
Then, you have to build executable `jar` files with the applications. Go to the `/start` folder and run following command: `mvnw.cmd clean package`

Then, in the configuration folder (`/config`), you will find JSON files which are the pre-made configuration files for your applications. 
To start your applications from Glue42 Enterprise, you should update the `"path"` properties in these JSON files (they must point to the directory 
with the `jar` file with application).
Then, copy the JSON files to the application configuration folder (`%LocalAppData%/Tick42/GlueDesktop/config/apps`). 
Note that the folder is monitored at runtime, so your applications will appear instantly. Now, you can start all applications 
through the App Manager. In the App Manager, all applications will have `Tutorial` prefixed to their titles to make them easier to find.

Here are some common issues you may encounter:
- An app is missing from the Glue42 Toolbar: check the [application configuration file](../../developers/configuration/application/index.html#application_configuration-exe), it must be a valid JSON.
- An app doesn't start after being clicked: make sure that the `"path"` property of the application configuration file points to the directory with `jar` which contains the application. More information can be found [here](../../../getting-started/how-to/glue42-enable-your-app/java/index.html#application_definition)
- The `Clients` application doesn't visualize any data - the data provider `exe` must be running, so make sure that the `"path"` in its configuration file is valid and that port 8083 is free (this is where the data is hosted).


## Initializing the Glue42 Library

To use Glue42 Java library in your application, you must first initialize it. Simply create new instance of `Glue` class using its builder:
```java
import com.tick42.glue.Glue; 

Glue glue = Glue.builder().build();
```

It is best that this happens at the first possible moment so that Glue42 is always available. In all applications, the initialization is done 
in `main` method, before `JFrame` is created.
In most cases, it will be enough, however if you want better performance or to tinker with the advanced options you can check out 
the [Glue42 Enable Your App](../../../getting-started/how-to/glue42-enable-your-app/java/index.html) documentation.

Remember to always close the `Glue` instance when you're done with it. In all applications it is done when the window is closed.


## 1. Window Management

Users of desktop applications start to face issues with window management when they have to work with many apps and their desktop 
gets cluttered with random windows. Glue42 has a solution for this - the StickyWindows API. With this API you can make your application 
sticky which gives your users the opportunity to organize their desktops. 

To make the app windows sticky, find 
all `registerWithGlue()` methods in your applications and implement them, so that all applications are flat windows. You can use
`getWindowHandle()` and `register()` methods from `glue.windows()` API. For more details on how to do this, 
see the Java [Window Management](../../../glue42-concepts/windows/window-management/java/index.html) documentation.

*For each task you will find hints and guidelines as comments in the code.*


## 2. Shared Contexts

Glue42 offers many ways to share data between applications, the easiest one being the [Shared Contexts](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/java/index.html) API. 
The Shared Contexts API can pass data globally to all currently running applications. This data can be updated by everyone and everyone 
can listen for the updates.

First, we will start from the `Clients` project. In the `ClientsTable` class you can find the `onClientSelected()` method. 
The method should update context named `selectedClient`, so that every time the user clicks on a row, the context is updated. 
The `Portfolio` application should listen for changes and display the data. You should now follow the instructions in the code 
and implement this functionality. More information about the Shared Contexts API can be found 
[here](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/java/index.html).


## 3. Interop - Declarative Model

The [Interop](../../../glue42-concepts/data-sharing-between-apps/interop/overview/index.html) API is another way through which
you can share data with other applications. In this case, it is a better form of communication, so you should comment out your context code.

### 3.1 Method Registration
 
The `Portfolio` app is responsible for visualizing data, so it should register a method `showPortfolio()` which accepts the needed data 
and updates the `Portfolio` UI. You can find more information on how to register a method 
[here](../../../glue42-concepts/data-sharing-between-apps/interop/java/index.html#method_registration-registering_methods).

### 3.2 Method Invocation

The `Clients` app has data for visualization, so it must invoke `showPortfolio()` on every row click. You should pass list of client details.
It is available through `client.getDetails()` method. When you implement this, you will see the same result as the one from the Shared Contexts. 
You can find more information on how to invoke a method [here](../../../glue42-concepts/data-sharing-between-apps/interop/java/index.html).


## 4. Notifications

### 4.1 Raising Notifications

Glue42 has a notification API which is accessible through `glue.notifications()`. It can publish notifications with information 
and possible actions on them in the form of toasts which appear in the bottom right corner of the screen. Your next task 
is to implement the `raiseNotification()` method in the `Notifications` project. When you are done, the raise button 
should raise a basic notification with the same title as the text in the text box of the app. 
For more information on notifications, [see here](../../../glue42-concepts/notifications/java/index.html).

### 4.2 Glue42 Routing

If you click on a notification, a default screen will be opened with further information. Nevertheless, Glue42 can be configured 
so that a custom Interop method will be executed when a notification is clicked. You can pass Interop method name 
in the builder `details()` method. You must register an Interop method `showContacts()` in the `Clients` application that opens 
the `Contacts` application. For more information on Glue42 notifications, [see here](../../../glue42-concepts/notifications/java/index.html). 
To see how to open an application, continue to the next chapter.


## 5. Application Management

Glue42 has an Application Management API through which you can execute all kinds of application management commands. 
For the current scenario you will only need to start a new instance of the `Contacts` application in the `showContacts()` method. 
To see how to use the Application Management API, click [here](../../../glue42-concepts/application-management/java/index.html#starting_applications). 
Keep in mind that the `Clients` app must be open when you click on a notification, if you want the `showContacts()` method to be invoked.

So, now the `Contacts` app is opened, but you still need to have data in it. You can get your data from the `Clients` app. 
Go to the `FindWhoToCallInterop` class in `Clients` application, implement the `register()` method. 
Then invoke it from `Contacts` using Interop API and use the result to update the UI.


## Congratulations!

You should now have the foundation to start working on Glue42 enabling your real applications.

Next step?

Build awesome Glue42 enabled apps!!
