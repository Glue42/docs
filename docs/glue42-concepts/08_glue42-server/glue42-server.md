## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

Glue42 Server is a server-side application that provides data to Glue42 (applications, Layouts, preferences) and allows monitoring and interacting with users running Glue42. It also includes an Admin UI that helps managing the data stored in the Glue42 Server easier.

<glue42 name="diagram" image="../../images/server/server-architecture.png">

The Glue42 Server is available on GitHub - for access and more details, [contact us](https://glue42.com/contacts/) at `info@glue42.com`.

## Features

### Application Store

The Glue42 Server provides an application definition store that can be used by [**Glue42 Enterprise**](https://glue42.com/enterprise/) to retrieve the list of applications for the current user.

### Layout Store

The Glue42 Server is a Layout store from where common or private user Layouts can be fetched.

### Application Preferences Store

The Glue42 Server is an application preferences store where any application running in [**Glue42 Enterprise**](https://glue42.com/enterprise/) can store custom data per user and retrieve it later.

### Diagnostics

The Glue42 Server comes with an Admin UI which offers the following functionalities:

- Command Center - send commands to specific user sessions for getting logs, triggering page refresh, restarting the desktop client or even executing custom code;
- Feedback and Native Crashes - monitor, review and comment user Feedback and Native Crashes reports;
- User Monitoring - monitor users and their sessions (current and closed), inspect their hardware setup;