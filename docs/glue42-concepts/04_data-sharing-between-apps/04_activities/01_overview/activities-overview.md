## Overview

An Activity is a collection of windows organized in a layout and sharing a private context. Here is an example of an Activity consisting of four windows:

![Example Activity](../../../../images/activities/activity.gif)

Activities are usually registered as components in the application configuration and can be instantiated either as applications from the toolbar or programmatically - on demand.

## Activities Terms Definitions

An *Activity type* is a definition template for an Activity, consisting of a collection of *window types*, their layout and an initial *Activity context*.

A *window type* is a definition of a window, typically configured in the application configuration settings. However, the Activities API allows for an application to dynamically define both (window types and Activity types) at runtime.

An *Activity context* is an object containing a set of key/value pairs which hold the current state of an Activity - e.g., the currently selected `party`, `instrument`, `order`, etc.

An *Activity instance* is an instance of an *Activity type*, just like an object is an instance of a class in class-based languages. "Activity" is often used as a synonym for Activity instance.

The Activities API enables:

- the definition of window types and Activity types (collection of window types, layout and an initial context);
- starting an Activity instance of a specific Activity type;
- reacting to Activity events from an Activity-aware window, such as joining and leaving an Activity;
- application state synchronization via Activity context management functions;
- intra-Activity application collaboration via Activity [Interop](../../interop/overview/index.html) extension functions; 