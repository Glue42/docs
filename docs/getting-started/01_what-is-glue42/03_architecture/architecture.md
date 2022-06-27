## Architectural Diagram

<glue42 name="diagram" image="../../../images/architecture/glue-architecture.png">

## How It All Works

The main components of [**Glue42 Enterprise**](https://glue42.com/enterprise/) are Glue42 Desktop and the Glue42 Gateway. These two components can be packed together or deployed independently.

### Glue42 Desktop

Glue42 Desktop is an Electron based app which offers APIs for:

[App Management](../../../glue42-concepts/application-management/overview/index.html)

- Loading the list of app configurations for the current user;
- Managing the apps life-cycle (starting/stopping apps);

[Window Management](../../../glue42-concepts/windows/window-management/overview/index.html)

- Hosting web apps in desktop windows;
- Sticking windows together;
- Advanced window management - [Workspaces UI](../../../glue42-concepts/windows/workspaces/overview/index.html)
- Saving and restoring layouts - [Layouts](../../../glue42-concepts/windows/layouts/overview/index.html)

[Data Sharing Between Apps](../../../glue42-concepts/data-sharing-between-apps/overview/index.html)

- Synchronizing cross-app data by publishing and subscribing for updates of data objects - [Shared Contexts](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/overview/index.html)
- UI based on Shared Contexts - [Channels](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html)
- Registering and invoking methods - [Interop](../../../glue42-concepts/data-sharing-between-apps/interop/overview/index.html)

The Glue42 APIs are available in JavaScript/TypeScript (as well as through lightweight Glue42 React and Angular wrappers) for web apps and in .NET, Java, and COM/VBA for desktop apps.

Glue42 Desktop is highly customizable and many of its features can be enabled, disabled or [re-configured](../../../developers/configuration/overview/index.html).

### Glue42 Gateway

The Glue42 Gateway is a transport with domain specific protocols. It enables the communication between apps running in [**Glue42 Enterprise**](https://glue42.com/enterprise/).
By default, it uses WebSockets for delivering messages to apps. Other options are also available upon request. The Glue42 Gateway is written in Clojure and can be transpiled to Java and JavaScript.

### App Stores

[**Glue42 Enterprise**](https://glue42.com/enterprise/) loads the list of apps available to the user from a pre-configured set of [app stores](../../../glue42-concepts/application-management/overview/index.html#app_stores). They can be of different type:

- Local stores - definitions are read from folders on the machine that is running [**Glue42 Enterprise**](https://glue42.com/enterprise/);
- Remote stores - definitions are fetched from a REST service;
- Glue42 Server stores - definitions are retrieved from a [Glue42 Server](../../../glue42-concepts/glue42-server/index.html);

To add an app to [**Glue42 Enterprise**](https://glue42.com/enterprise/), you need to add it to an app store.

### Connectors

[**Glue42 Enterprise**](https://glue42.com/enterprise/) comes with a set of [Connectors](../../../connectors/general-overview/index.html) for integrating third-party apps (MS Office, Bloomberg, Salesforce, etc.) with Glue42 enabled apps. The Connectors expose functionalities provided by a specific third-party app, so that they can be re-used in Glue42 enabled apps.The Glue42 [Excel Connector](../../../connectors/ms-office/excel-connector/javascript/index.html), for instance, allows apps to open an Excel sheet, populate it with data from the app and receive updates when the data in Excel is changed.

## Additional Services

### Notifications

The [Glue42 Notification Service](../../../glue42-concepts/notifications/overview/index.html) (GNS) handles notifications from GNS servers and delivers them to the end-user via a notifications UI.

### Global Search

The [Global Search](../../../glue42-concepts/global-search/index.html) performs global searches on all defined search providers and returns consolidated results to the user.

### Metrics

Extensive [Metrics](../../../glue42-concepts/metrics/overview/index.html) for tracking the user journey can be collected from all Glue42 enabled apps and published to a data center.

### Glue42 Server

[Glue42 Server](../../../glue42-concepts/glue42-server/index.html) is a server-side app that provides data to Glue42 (apps, Layouts, preferences) and allows monitoring and interacting with users running Glue42. It also includes an Admin UI that helps managing the data stored in the Glue42 Server easier.