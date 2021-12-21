## Initialization

The **BBG Market Data** library is available as an `npm` package - [`@glue42/bbg-market-data`](https://www.npmjs.com/package/@glue42/bbg-market-data). To install it, run the following command in your project directory:

```cmd
npm install @glue42/bbg-market-data
```

The **BBG Market Data** API depends on Glue42 [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/overview/index.html), an instance of which must be passed to the `BBGMarketData()` factory function. The function also accepts as a second parameter a configuration object that controls logging behavior and can also provide an optional custom logger implementation. The configuration object can also specify the interval at which to attempt reconnection to the Bloomberg Connector if a connection doesn't exist or is interrupted.

```typescript
import BBGMarketData from "@glue42/bbg-market-data";
import GlueCoreFactory from "@glue42/core";

GlueCoreFactory().then(glue => {
    const bbgMarketData: BBGMarketDataAPI = BBGMarketData(glue.interop, { debug: true, connectionPeriodMsecs: 7000 });
});
```

- `debug` - whether to enable debugging mode (`false` by default);
- `connectionPeriodMsecs` - the interval at which to attempt reconnection to the Bloomberg Connector (in ms, `5000` by default);

## Connection

To receive updates about changes in the connection between the **BBG Market Data** API and the Bloomberg Connector, you can use the `onConnectionStatusChanged()` method. It accepts a callback with which you can handle changes in the connection status and returns an unsubscribe function that you can invoke to stop receiving updates about the connection status:

```typescript
function handleConnectionChanges (status: ConnectionStatus): void {
    
    if (status === ConnectionStatus.Connected) {
        // Connected.
    } else {
        // Disconnected.
    };
};

const unsubscribe: UnsubscribeFunction = bbgMarketData.onConnectionStatusChanged(handleConnectionChanges);
```

## Handling Requests

A request to a Bloomberg service must first be created. After creating a request instance, you can optionally provide callbacks for handling response data, errors, subscription failures, request status changes and request related events. After that, the request is sent to the Bloomberg service. A request can also be closed if its status is `Opened` or `Active`.

### Creating a Request

Below is an example of creating a `HistoricalDataRequest` using the `createHistoricalDataRequest()` method of the API:

```typescript
const requestArgs: HistoricalDataRequestArguments = {
    securities: ["IBM US Equity", "MSFT US Equity"],
    fields: ["LAST_PRICE"],
    startDate: "20190101",
    endDate: "20191231"
};

const request: HistoricalDataRequest = bbgMarketData.createHistoricalDataRequest(requestArgs);
```

### Opening a Request

To actually send the request to the Bloomberg service, invoke the `open()` method of the request instance. Requests can be opened/reopened when they have a `Created`, `Closed`, `Failed` or `Completed` status. When the request is pending (its status is `Opened` or `Active`), `open()` will immediately throw an error.

```typescript
// Request options. 
const requestOptions: OpenRequestOptions = {
    session: Session.LargeHistoricalRequests,
    aggregateResponse: false
};

// Send the actual request to the Bloomberg service.
request.open(requestOptions);
```

The `open()` method accepts an optional `OpenRequestOptions` object with the following properties:

- `session` - the type of session to use. The default for non-subscription requests is `DataRequests`, the default for subscription requests is `RealTime`;
- `aggregateResponse` - whether to return an aggregated response. If `true` (default), `open()` returns a `Promise` which resolves with the aggregated response. If `false`, the `Promise` resolves immediately and the partial responses are passed to the callback attached to the `onData()` method of the request object.

As opening a request is asynchronous, if any issue occurs (e.g., the Bloomberg Connector fails to open the Bloomberg core service), the error will be delivered asynchronously to the request `onError()` callback.

### Closing a Request

To close a request, invoke the `close()` method of a request instance. A request can be closed only if its status is `Opened` or `Active`. Otherwise, `close()` will just resolve. If any issues occur when attempting to close the request, the `close()` method rejects with an error.

```typescript
request
    .close()
    .then(() => {
        // Request closed successfully. Now can be reopened.
    })
    .catch(error => {
        // Unable to close the request. Check error.
    });
```

### Response Data

To handle response data, attach a callback to the `onData()` method of the request instance. The method returns an `UnsubscribeFunction`. The `onData()` method handles data from real-time subscriptions and data from partial responses from static reference data requests.

When handling data from a non-subscription request, the data is wrapped in a `ResponseData` object. The boolean property `isLast` is set to `false` when the response contains a Bloomberg `PARTIAL_RESPONSE` event and is set to `true` when the response contains a Bloomberg `RESPONSE` event. After a `RESPONSE` event, no more events will be received and the application can now process the returned data accordingly.

Handling partial response data from static reference data requests:

```typescript
request.onData(function handleResponse(
    response: ResponseData<HistoricalData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});
```

If you want to directly get the final aggregated response from a static reference data request, you can await the `Promise` returned from the `open()` method of the request instance. (This applies only if you haven't explicitly set the `aggregateResponse` property of the optional object passed to `open()` to `false`):

```typescript
const response: HistoricalData[] | undefined = await request.open();

if (response) {
    // Handle aggregated response.
};
```

When handling data from a subscription request, the data for each subscription is wrapped in a `SubscriptionData` object.

Handling data from real-time subscription requests:

```typescript
request.onData(function handleSubscriptionsData(
    subscriptionsData: Array<SubscriptionData>
): void {
    // Handle subscription updates.
});
```

### Response Errors

To handle errors in a response, attach a callback to the `onError()` method of the request instance. The method returns an `UnsubscribeFunction`. An error can be returned when:

- Unable to invoke the Bloomberg Connector.
- The Bloomberg Connector has thrown an exception.
- The Bloomberg Connector wasn't able to create the request to the Bloomberg service and returns an unsuccessful result.
- An active request to a Bloomberg service has failed (e.g., a "RequestFailure" message was received for a non-subscription request).

### Request Status

To track the current request status, attach a callback to the `onStatus()` method of the request instance. The method returns an `UnsubscribeFunction`.

There are six statuses:

- `Created` - The request has been created but not sent to a Bloomberg service.
- `Opened` - The actual request has been sent to a Bloomberg service, but isn't active yet. Response still not available.
- `Active` - The request has been sent to a Bloomberg service successfully. Responses may be received.
- `Failed` - The request has failed to open or an error is received from a Bloomberg service.
- `Closed` - The request was closed before completing. No more responses will be received.
- `Completed` - The request was completed successfully. No more responses will be received.

### Request Events

If you want to receive all Bloomberg events related to the current request, you can attach a callback to the `onEvent()` method of the request instance. The method returns an `UnsubscribeFunction`.

```typescript
request.onEvent(function handleBloombergEvent(event: BloombergEvent) {
    // Track all events related to the current request.
});
```

## Request Types

### Market Data Subscription

A Market Data Subscription request enables retrieval of streaming data for securities that are priced intraday by using the Bloomberg API Subscription paradigm. It uses the Bloomberg API core service `//blp/mktdata`. The subscriber receives updates once a field value changes at the source. Desired fields must explicitly be listed in the subscription to receive updates for them. It is **required** for each subscription to have a `security` property and at least one field in the `fileds` property. A full range of options (like `subscriptionId`, `intervalInSeconds`, `delayed`) can be specified in the `Subscription`.

Below is an example of creating and opening a Market Data Subscription request:

```typescript
const subscriptions: Array<Subscription> = [
    {
        security: "IBM US Equity",
        fields: ["LAST_PRICE", "BID", "ASK"],
        // Interval for receiving conflated data.
        intervalInSeconds: 2,
        // Whether the received data to be real-time or delayed.
        delayed: false
    }
];

// Creating the request.
const request: SubscriptionRequest = bbgMarketData.createMarketDataRequest(subscriptions);

request.onData(function handleSubscriptionsData(
    subscriptionsData: Array<SubscriptionData>
): void {
    // Handle subscription updates.
});

// The callback in the `onFail()` method will be invoked when a subscription for a security 
// fails on the Bloomberg side. E.g., you may have sent a request with
// five subscriptions for five different securities and two of the subscriptions fail.
request.onFail(function handleSubscriptionsError(
    subscriptionErrors: Array<SubscriptionError>
): void {
    // Handle subscription errors.
});

// The callback in the `onError()` method will be invoked whenever an error occurs with
// the request itself. E.g., error in creating or sending the request.
request.onError(function handleRequestError(error): void {
    // Handle request error.
});

// Sending the request to a Bloomberg service.
request.open();
```

Each subscription must have a unique ID. If not explicitly set, the library assigns one:

```typescript
import { CorrelationId, Subscription } from "@glue42/bbg-market-data";

const subscriptions: Subscription[] = [
    // This subscription has explicitly assigned ID.
    {
        subscriptionId: new CorrelationId(),
        security: "IBM US Equity",
        fields: ["LAST_PRICE", "BID", "ASK"]
    },

    // А `subscriptionId` will be assigned automatically for this subscription.
    {
        security: "MSFT US Equity",
        fields: ["LAST_PRICE", "BID", "ASK"]
    }
];
```

### Historical Data

A Historical Data request enables the retrieval of end-of-day data for a set of securities and fields over a specified period, which can be set to daily, monthly, quarterly, semi-annually or annually by using the Bloomberg Request/Response paradigm. It uses the Bloomberg API core service `//blp/refdata`.

At least one value in the `securities` and in the `fields` properties is **required** along with a `startDate` and an `endDate`. A range of other options can be specified in the `HistoricalDataRequestArguments` object. To create a Historical Data request, use the `createHistoricalDataRequest()` method:

```typescript
const requestArgs: HistoricalDataRequestArguments = {
    securities: ["IBM US Equity", "MSFT US Equity"],
    fields: ["LAST_PRICE"],
    startDate: "20190101",
    endDate: "20191231"
};

// Creating the request.
const request: HistoricalDataRequest = bbgMarketData.createHistoricalDataRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<HistoricalData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### Reference Data

A Reference Data request retrieves the current data available for a security/field pair by using the Bloomberg Request/Response paradigm.
It uses the Bloomberg API core service `//blp/refdata`.

At least one value in the `securities` and in the `fields` properties is **required**. A range of other options can be specified in the `ReferenceDataRequestArguments` object. To create a Reference Data request, use the `createReferenceDataRequest()` method.

```typescript
const requestArgs: ReferenceDataRequestArguments = {
    securities: ["IBM US Equity"],
    fields: ["LAST_PRICE"]
};

// Creating the request.
const request: ReferenceDataRequest = bbgMarketData.createReferenceDataRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<ReferenceData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### Instrument List

The Instrument List request performs a search for securities based on a specified search string. This functionality resembles the `SECF <GO>` function of the Bloomberg Professional Service. It uses the Bloomberg API core service `//blp/instruments`.

Specifying a search string and a maximum number of results is **required**. A range of other options can be specified in the `InstrumentListRequestArguments` object. To create an Instrument List request, use the `createInstrumentListRequest()` method:

```typescript
const requestArgs: InstrumentListRequestArguments = {
    query: "VOD",
    maxResults: 5
};

// Creating the request.
const request: InstrumentListRequest = bbgMarketData.createInstrumentListRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<InstrumentListData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### Intraday Bar

The Intraday Bar request enables retrieval of summary intervals for intraday data covering five event types: `TRADE`, `BID`, `ASK`, `BEST BID` and `BEST ASK`, over a period of time by using the Bloomberg Request/Response paradigm. It uses the Bloomberg API core service `//blp/refdata`.

It is **required** to specify a `security` and `eventType`. Also, `startDateTime` and `endDateTime` points in UTC must be specified. A range of other options can be specified in the `IntraDayBarRequestArguments` object. To create an Intraday Bar request, use the `createIntraDayBarRequest()` method:

```typescript
const requestArgs: IntraDayBarRequestArguments = {
    security: "IBM US Equity",
    eventType: IntraDayEventType.ASK,
    startDateTime: "2019-01-01T13:00:00",
    endDateTime: "2019-12-31T13:00:00"
};

// Creating the request.
const request: IntraDayBarRequest = bbgMarketData.createIntraDayBarRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<IntraDayBarData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### Snapshot

The Snapshot request enables retrieval of static market list snapshot data by using the Bloomberg Request/Response paradigm. It uses the Bloomberg API core service `//blp/mktlist`.

It is **required** to specify a `security`. To create a Snapshot request, use the `createSnapshotRequest()` method:

```typescript
const requestArgs: SnapshotRequestArguments = {
    security: "VOD LN Equity"
};

// Creating the request.
const request: SnapshotRequest = bbgMarketData.createSnapshotRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<SnapshotData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### Field List

The Field List request returns all fields of the same type by using the Bloomberg Request/Response paradigm. It uses the Bloomberg API core service `//blp/apiflds`. Possible `fieldType` values include:

- `All` - returns all fields in the API Data Dictionary;
- `Static` - returns all static fields contained in the API Data Dictionary;
- `RealTime` - returns all real-time fields contained in the API Data Dictionary;

A range of options can be specified in the `FieldListRequestArguments` object. To create a Field List request, use the `createFieldListRequest()` method:

```typescript
const requestArgs: FieldListRequestArguments = {
    fieldType: FieldType.RealTime
};

// Creating the request.
const request: FieldListRequest = bbgMarketData.createFieldListRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<FieldListData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### Field Search

The Field Search request returns all fields matching a specified search criterion by using the Bloomberg Request/Response paradigm. It uses the Bloomberg API core service `//blp/apiflds`.

A range of options can be specified in the `FieldSearchRequestArguments` object. To create a Field Search request, use the `createFieldSearchRequest()` method:

```typescript
const requestArgs: FieldSearchRequestArguments = {
    searchSpec: "mkt"
};

// Creating the request.
const request: FieldSearchRequest = bbgMarketData.createFieldSearchRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<FieldSearchData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

### User Entitlements

The User Entitlements request returns a list of entitlement IDs by using the Bloomberg Request/Response paradigm. It uses the Bloomberg API core service `//blp/apiauth`. To create a User Entitlements request, use the `createUserEntitlementsRequest()` method:

```typescript
const requestArgs: UserEntitlementsRequestArguments = {
    uuid: 1000
};

// Creating the request.
const request: UserEntitlementsRequest = bbgMarketData.createUserEntitlementsRequest(requestArgs);

request.onData(function handleResponse(
    response: ResponseData<UserEntitlementsData>
): void {
    if (response.isLast) {
        // Handle the final response.
    } else {
        // Handle partial responses.
    }
});

// Sending the request to a Bloomberg service.
request.open();
```

## Session

*Note that it is strongly recommended to use the default session settings when opening a request.*

For advanced scenarios, however, you can pass an optional object parameter to the `open()` method of a request instance where you can specify what type of session you want to use (or create). The BBG Market Data library also offers a Sessions API with which you can retrieve all currently open user-defined sessions (opened via the API and only by *this* instance of the API), as well as explicitly close a custom session.

### Session Types

There are five `Session` types:

- `RealTime` - Session for real-time subscriptions. Default for subscription requests.
  
- `DataRequests` - Session for static reference data. Default for non-subscription requests.
 
- `LargeHistoricalRequests` - Dedicated session for a large volume data request.
  
- `CreateNew` - Creates a new session which closes immediately after the request is completed.
  
- `Default` - Explicitly states that the default session should be used.

If you need to use a session different from the default one, you need to pass an options object to the `open()` call, specifying the session type.

Below is an example of using a dedicated session for a large historical data request:

```typescript
// We have already created a Historical Data `request` instance
// and attached to it the respective callbacks for handling data and errors.

const requestOptions: OpenRequestOptions = { session: Session.LargeHistoricalRequests };

const responseData: HistoricalData = await request.open(requestOptions);
```

### Custom Sessions

Besides the predefined session types, you can specify your own custom session type (in highly unlikely scenarios). Bear in mind that custom sessions are kept open until you close them explicitly. (If you want to use a new session that is automatically closed when the request has completed, use the `CreateNew` session type.)

Below is an example of creating a custom session for real-time market data subscription:

```typescript
// We have already created a Subscription `request` instance.
// and attached to it the respective callbacks for handling data and errors.

const requestOptions: OpenSubscriptionRequestOptions = { session: "MyCustomSession" };

request.open(requestOptions);

// You can use the response data here, use the custom session for other requests, etc.

// Note that you must explicitly close a custom session.
bbgMarketData.sessions.close("MyCustomSession");
``` 