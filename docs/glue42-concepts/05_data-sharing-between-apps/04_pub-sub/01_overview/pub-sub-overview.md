## Overview

The Pub/Sub API enables apps to:

- publish messages on a specific topic;
- subscribe for messages on a specific topic;

When an app publishes a message on a specific topic, the Pub/Sub API delivers it to other apps that have subscribed to that topic.

The "raw Pub/Sub" support allows [**Glue42 Enterprise**](https://glue42.com/enterprise/) to work with apps already using a Pub/Sub technology. Before writing new Pub/Sub based code, please consider the higher level [Interop](../../interop/overview/index.html) services provided by Glue42 Enterprise: Request/Response, Streaming, Discovery and Shared Contexts. Utilizing these services, instead of creating them from scratch, can save you time and also provide you with a more robust service that can interact with apps by different dev teams and vendors.