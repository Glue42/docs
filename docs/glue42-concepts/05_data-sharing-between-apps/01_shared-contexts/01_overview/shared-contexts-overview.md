## Overview

A shared context is a named object (holding a `map` of key/value pairs) that stores cross app data. The context object can hold any cross-app data. Any app can update a context or subscribe for context updates and react to them by using the name of the context.

The Shared Contexts API offers a simple and effective solution for sharing data between your apps. Imagine you have an app showing a list of clients and an app showing client portfolios. What you need, is your "Portfolio" app to show the portfolio of a specific client that the user has selected from the "Clients" app. You can easily achieve this in a few simple steps by using the Shared Contexts API:

- instruct the "Clients" app to publish updates to a context object holding the `id` of the currently selected client;
- instruct the "Portfolio" app to subscribe for updates of that same context object and specify how the "Portfolio" app should handle the received data in order to update its current state;

<glue42 name="diagram" image="../../../../images/shared-contexts/shared-contexts.gif">