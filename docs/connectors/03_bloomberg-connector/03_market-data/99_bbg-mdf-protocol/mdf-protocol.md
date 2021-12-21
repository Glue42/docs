## Overview

The Bloomberg Market Data offers retrieval of real-time/delayed streaming market data and static reference market data for securities through subscription and request/response mechanisms. Using the available Market Data protocols or [APIs](../javascript/index.html) you can easily acquire market data provided by Bloomberg in order to show it, use it or redistribute it throughout your applications.

## Creating Requests

### Streaming Data Subscription Request

- To create a Bloomberg Market Data Subscription request, use:

```csharp
(Composite:TerminalResult{String correlationId, Value elementSchema, String message, Value requestSchema, Value responseSchemas, Bool success} result) 
T42.MDFApi.CreateSubscriptionRequest 
(String callbackMethod, String requestCorrelationId, String service, Value settings, 
Composite:TerminalSubscription{String fields, String options, String security, String subscriptionId}[] subscriptions)
```

The `T42.MDFApi.CreateSubscriptionRequest` method will create a session (or reuse a named one by using the `sessionName` property), open the specified service and create a subscription request with the specified settings. It will return a composite `TerminalResult` that indicates whether the subscription request was successful or not.

### Static Reference Data Request

- To create a Bloomberg Static Reference Data request, use:

```csharp
(Composite:TerminalResult{String correlationId, Value elementSchema, String message, Value requestSchema, Value responseSchemas, Bool success} result) 
T42.MDFApi.CreateRequest (String callbackMethod, String operation, Composite[] operationArgs, String requestCorrelationId, String service, Value settings)
```

The `T42.MDFApi.CreateRequest` method will create a session (or reuse a named one by using the `sessionName` property), open a service and send the Bloomberg request. All returned data (including service and session status transitions) will be translated to Glue42 elements and returned to the method specified by the `callbackMethod` argument, using the specified `requestCorrelationId`. It will return a composite `TerminalResult` that indicates whether the request was successful or not.

## Canceling Requests

- To cancel a request, use:

```csharp
(Composite:TerminalResult{String correlationId, Value elementSchema, String message, Value requestSchema, Value responseSchemas, Bool success}[] result) 
T42.MDFApi.CancelRequests (String[] requestCorrelationIds)
```

This allows you to cancel previous requests and subscriptions by specifying their correlation IDs.

## Session and Service Schemas

### Closing a Session

- To close a named session, use:

```csharp
(Composite:TerminalResult{String correlationId, Value elementSchema, String message, Value requestSchema, Value responseSchemas, Bool success} result) 
T42.MDFApi.CloseSession (String sessionName)
```

This allows you to "clear" and close existing sessions by name.

### Service Operations Schemas

- To get descriptions (schemas) of service operations, use:

```csharp
(Composite:TerminalResult{String correlationId, Value elementSchema, String message, Value requestSchema, Value responseSchemas, Bool success} result) 
T42.MDFApi.DescribeServiceSchemas (String operation, String service, Value settings)
```

This allows you to describe service operations and get their request/response schemas. This is particularly helpful when you can't remember the arguments of a request or you have typos/wrong types of values. If you don't specify the operation argument, the method will return all operations within the specified service. 