## Overview

What is a notification? A notification is an event important enough to be brought to the attention of a user.

A few contextual examples:

- an *alert* from an app for an automatic machine restart, or from a metrics monitoring tool for a system running hot;
- a *trade order execution* notification from an Order Management System;
- a *workflow task* assigned to a user or a group of users that can optionally need a response (e.g., a change of client address);
- an *activity* that is forwarded/re-assigned to a user (e.g., handling a client call);

The Glue42 Notification Service (GNS) is an unopinionated architecture, a set of executables, APIs and specifications for delivering notifications directly to the desktop. It provides:

- A common data model for normalizing notifications raised by apps.
- Consolidation of notifications from multiple backend and desktop servers and their customizable delivery to the user's desktop via the GNS Desktop Manager.
- Specification for GNS servers for passive (REST) and active ([Interop](../../data-sharing-between-apps/interop/overview/index.html), HTTP Push) publishing of notifications.
- Interop Publishing API for desktop GNS Servers.
- A set of GNS Servers:
	- Outlook Mail Interop GNS Server (e.g., public folder messages, but may also be rule-based);
	- Thomson Reuters Eikon News Interop GNS Server;
	- Google Alerts Interop GNS Server;
- Interop Subscription API for desktop clients/subscribers (typically UIs);
- UIs for displaying notifications (*toasts* and *lists*) with customizable UX via Glue42 Routing;

![GNS UI](../../../images/notifications/gns-ui.png)

## Architecture

<glue42 name="diagram" image="../../../images/notifications/gns.gif">