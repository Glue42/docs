## Overview

The Interop API enables apps to:

- offer functionality to other apps (web and native) by registering Interop methods;
- discover apps which offer methods;
- invoke methods on the user's desktop and across the network;
- stream and subscribe for real-time data using the streaming methods of the Interop API;

Apps which offer methods and streams are called *Interop servers*, and apps which consume them - *Interop clients*, and collectively - *Interop instances*.

<glue42 name="diagram" image="../../../../images/interop/interop.gif">

Any running instance of an app is identified by its *Interop instance*, which is a set of known key/value pairs uniquely identifying an app.