## Overview

[**Glue42 Core**](https://glue42.com/core/) is the world's first open source web integration platform. It enables multiple standalone web apps to share data between each other, expose functionality, open and manipulate windows. [**Glue42 Core**](https://glue42.com/core/) is mainly targeted for use with Progressive Web Apps. Combining PWAs with [**Glue42 Core**](https://glue42.com/core/) not only leverages the advantages of PWAs (native-like feel, working offline, enhanced performance, etc.), but incorporates an interoperability layer in your web app ecosystem as well.

[**Glue42 Core+**](https://glue42.com/core-plus/) includes all free open-source features of [**Glue42 Core**](https://glue42.com/core/) and offers an extensive set of additional features available under a paid license, thus bridging the gap between [**Glue42 Core**](https://glue42.com/core/) and [**Glue42 Enterprise**](https://glue42.com/enterprise/).

## Glue42 Core

A [**Glue42 Core**](https://glue42.com/core/) project consists of a [Main app](https://core-docs.glue42.com/developers/core-concepts/web-platform/overview/index.html) which uses the Glue42 [Web Platform](https://www.npmjs.com/package/@glue42/web-platform) library and multiple [Web Client](https://core-docs.glue42.com/developers/core-concepts/web-client/overview/index.html) apps that use the [Glue42 Web](https://core-docs.glue42.com/reference/core/latest/glue42%20web/index.html) library. The Main app acts as a hub through which the user can access all other apps part of the [**Glue42 Core**](https://glue42.com/core/) project while the [Web Platform](https://www.npmjs.com/package/@glue42/web-platform) library provides the communication connection between all client apps. The [Glue42 Web](https://core-docs.glue42.com/reference/core/latest/glue42%20web/index.html) library provides Glue42 functionality to the client apps through sets of Glue42 APIs.

All apps participating in a [**Glue42 Core**](https://glue42.com/core/) project, including the [Main app](https://core-docs.glue42.com/developers/core-concepts/web-platform/overview/index.html), can use Glue42 functionalities like App Management, Intents, Shared Contexts, Channels, Interop, Window Management, Workspaces, Notifications and more.

*For more details about the available Glue42 functionalities and how to use them, see the [Capabilities](https://core-docs.glue42.com/capabilities/application-management/index.html) section of the [**Glue42 Core**](https://glue42.com/core/) documentation.*

*For more information, see the [Glue42 Core product page](https://glue42.com/core/), the [official Glue42 Core documentation](https://core-docs.glue42.com) and the [Glue42 Core GitHub repo](https://github.com/Glue42/core).*

## Glue42 Core+

A [**Glue42 Core+**](https://glue42.com/core-plus/) project has the same structure as a standard [**Glue42 Core**](https://glue42.com/core/) one, but offers rich additional functionalities enabling you to meet the business needs and requirements of your enterprise-level web projects.

The following diagram shows the features offered by [**Glue42 Core**](https://glue42.com/core/) and [**Glue42 Core+**](https://glue42.com/core-plus/):

<glue42 name="diagram" image="../../images/core-plus/core-plus.png">

In a licensed [**Glue42 Core+**](https://glue42.com/core-plus/) project, you can use:

- Layouts to save and restore multi-monitor app arrangements;
- Metrics to track performance;
- Connectors to integrate your web apps with Bloomberg, Fidessa, Excel, Outlook and more;
- which replaces the need for a [Main app](https://core-docs.glue42.com/developers/core-concepts/web-platform/overview/index.html) and provides many other handy features for your web project
- automatic [FDC3](https://fdc3.finos.org/) implementation injection;
- Glue42 Developer Tools (Interop Viewer, Context Viewer) for tracking the behavior of [Interop methods and streams](https://core-docs.glue42.com/capabilities/data-sharing-between-apps/interop/index.html), and [shared contexts](https://core-docs.glue42.com/capabilities/data-sharing-between-apps/shared-contexts/index.html).

[**Glue42 Core+**](https://glue42.com/core-plus/) enables you to connect to a [Glue42 Server](../../glue42-concepts/glue42-server/index.html), and even achieve interoperability across different user machines. Customer support is also available under a paid license for [**Glue42 Core+**](https://glue42.com/core-plus/).

*For more details, see the [Glue42 Core+ product page](https://glue42.com/core-plus/) and the [official Glue42 Core+ documentation](https://core-docs.glue42.com/getting-started/what-is-glue42-core-plus/index.html).*