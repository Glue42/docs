## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

[Fidessa](https://www.fidessa.com/) provides solutions for the entire investment life-cycle. Fidessa applications can now be integrated with other in-house/vendor applications in order to produce a more efficient financial desktop. Automatic data synchronization between Fidessa and other applications eliminates the risk of copy/paste errors and dramatically reduces task completion times.  

The Glue42 **Fidessa Connector** provides integration between Fidessa applications and Glue42 enabled applications. It uses the Fidessa WebSocket API specification.

When the Fidessa Connector is running, the Fidessa Tracking Groups and the [Glue42 Channels](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html) are synchronized automatically. Each Fidessa Tracking Group is mapped to a respective Glue42 Channel. This means that when a user clicks an instrument in a Fidessa application, all Glue42 enabled applications that are on the Channel corresponding to that Fidessa Tracking Group will also start showing data about the clicked instrument and vice versa. For example, you can have a Fidessa application showing real time market data about instruments and an in-house data chart application that shows detailed information about an instrument. The chart application is specifically designed for the needs of your company and you want to integrate it with the Fidessa applications. Your development team needs to write only a few lines of code in order to [Glue42 enable](../../../getting-started/how-to/glue42-enable-your-app/javascript/index.html) the chart application and modify it to work with the Glue42 Channels.

*To see how this works in practice, watch the Fidessa Connector videos from [this blog post](https://glue42.com/blog/fidessa-connector/).* 

The Fidessa Connector can be used together with the [Bloomberg Connector](../../bloomberg-connector/overview/index.html) in order to achieve syncing data between Fidessa, Bloomberg and all Glue42 enabled apps.

*Note that the Glue42 Fidessa Connector is an optional component of [**Glue42 Enterprise**](https://glue42.com/enterprise/) and isn't available in the public trial installer. To find out more about the Fidessa Connector, please [contact us](https://glue42.com/sales-contact/) at `sales@glue42.com`.*

## Requirements

The Glue42 Fidessa Connector works with all versions of the Fidessa "Managed Enterprise" Trading Platform that support the WebSocket API:

- v2018.2.25 and up;
- v2019.8.5 and up;
- v2020.2.1 and up; 