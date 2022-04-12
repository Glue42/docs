## Method Registration

*See the [Delphi 10](https://github.com/Glue42/native-examples/tree/main/glue-com/GlueDelphi) and [Delphi 7](https://github.com/Glue42/native-examples/tree/main/glue-com/GlueDelphi7) examples on GitHub.*

To expose a Glue42 method that can be invoked by other Glue42 enabled applications, you must register the method in Glue42 and provide its implementation. The application must also implement the [`IGlueRequestHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluerequesthandler) interface to dispatch and handle the method invocation requests.

This can be simplified by using the [`TGlueRequestHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#glue42_helper_unit-classes_for_handling_events-tgluerequesthandler) class of the [Glue42 Helper Unit](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#glue42_helper_unit) which takes care of implementing [`IGlueRequestHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluerequesthandler) and dispatching the request to a specific Delphi procedure.

### Registering Methods

To register an Interop method, use the [`RegisterMethod`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-registermethod) method of the [`IGlue42`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42) interface after Glue42 has been initialized. Provide a name and a [method implementation](#method_registration-method_implementation):

```delphi
TMainForm = class(TForm)
...
private
  G42: IGlue42;
  // The "DelphiAdd" Interop method handle.
  methodAdd: GlueMethod;
protected
  // Implementation of the "DelphiAdd" method.
  procedure GlueMethodAdd(Sender: TGlueRequestHandler;
    Method: GlueMethod;
    Instance: GlueInstance;
    Args: GlueContextValueArray;
    callback: IGlueServerMethodResultCallback;
    Cookie: TCallbackCookie;
    argsSA: PSafeArray);
  ...
  procedure TMainForm.InitializeGlue;
    ...
    G42.Start(inst);
    // Register the "DelphiAdd" method and create a handler linked to the `GlueMethodAdd` procedure.
    methodAdd := G42.RegisterMethod('DelphiAdd',
      TGlueRequestHandler.Create(nil,GlueMethodAdd),'','',nil);
    ...
```

### Method Implementation

The following example demonstrates a sample implementation of the "DelphiAdd" method. The method accepts two integers as arguments and returns their sum. If the arguments are invalid or missing, returns an error message:

```delphi
procedure TMainForm.GlueMethodAdd(Sender: TGlueRequestHandler;
  Method: GlueMethod;
  Instance: GlueInstance;
  Args: GlueContextValueArray;
  callback: IGlueServerMethodResultCallback;
  Cookie: TCallbackCookie;
  argsSA: PSafeArray);
var
  I: Integer;
  a: Int64;
  b: Int64;
  aValid: WordBool;
  bValid: WordBool;
  errorMessage: WideString;
  gValue: GlueValue;
  gResult: GlueResult;
begin
  // Validate the arguments.
  aValid := False;
  bValid := False;
  for I := Low(Args) to High(Args) do
  begin
    gValue := Args[I].Value;
    if (Args[I].Name = 'a') and ((gValue.GlueType = GlueValueType_Int)
     or (gValue.GlueType = GlueValueType_Long)) then begin;
      a := gValue.LongValue;
      aValid := True;
    end;
    if (Args[I].Name = 'b') and ((gValue.GlueType = GlueValueType_Int)
     or (gValue.GlueType = GlueValueType_Long)) then begin;
      b := gValue.LongValue;
      bValid := True;
    end;
  end;
  if not aValid then
    errorMessage := 'Argument a is missing or invalid.';
  if not bValid then
    errorMessage := 'Argument b is missing or invalid.';

  ZeroMemory(@gResult, sizeof(gResult));

  // Send an error message and a failed method invocations status if the arguments are invalid.
  if not (aValid and bValid) then begin
    gResult.Status := GlueMethodInvocationStatus_Failed;
    gResult.Message := errorMessage;
    callback.SendResult(gResult);
    Exit;
  end;

  // Prepare the result as a `PSafeArray` and send it via the callback.
  gResult.Values := CreateContextValues_SA(AsGlueContextValueArray([
    CreateContextValue('sum', CreateValue(a+b))
  ]));
  gResult.Status := GlueMethodInvocationStatus_Succeeded;
  callback.SendResult(gResult);
  SafeArrayDestroy(gResult.Values);
end;
```

## Method Invocation

To invoke Interop methods, use [`BuildAndInvoke`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-buildandinvoke), [`InvokeMethod`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-invokemethod) or [`InvokeMethods`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-invokemethods). [`BuildAndInvoke`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-buildandinvoke) accepts an instance of [`IGlueContextBuilderCallback`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluecontextbuildercallback) for building the arguments that will be passed to the invoked Interop method, while the other two methods accept the invocation arguments directly as a `PSafeArray`. If multiple applications or application instances have registered the same Interop method, you may select the instances which will service the method invocation (see [Targeting](#method_invocation-targeting)). After invoking an Interop method, you must [handle the returned result](#method_invocation-handling_invocation_results), if any.

### Invoking Methods

The following example demonstrates invoking the previously registered "DelphiAdd" Interop method by using [`InvokeMethods`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-invokemethods). The arguments passed to the method must be in a `PSafeArray`:

```delphi
// Prepare the arguments in a `PSafeArray`.
psaArgs := CreateContextValues_SA(AsGlueContextValueArray([
  CreateContextValue('a', CreateValue(5)),
  CreateContextValue('b', CreateValue(6))
]));

// Invoke the Interop method.
G42.InvokeMethods('DelphiAdd', psaArgs, nil, False,
  GlueInstanceIdentity_None, MethodAddHandler, 3000, '');

SafeArrayDestroy(psaArgs);
```

### Targeting

Targeting allows to invoke a method on one or more specific applications or application instances that have registered the same Interop method.

The following example demonstrates how to use [`InvokeMethods`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-invokemethods) to target various application instances:

```delphi
var
  psaArgs: PSafeArray;
  psaInstances: PSafeArray;
  instances: TGlueInstanceArray;
  instance: GlueInstance;
begin
...
  // Invoke a method on any of the application instances.
  G42.InvokeMethods('DelphiAdd', psaArgs, nil, False,
    GlueInstanceIdentity_None, MethodAddHandler, 3000, '');

  // Invoke a method on all application instances.
  G42.InvokeMethods('DelphiAdd', psaArgs, nil, True,
    GlueInstanceIdentity_None, MethodAddHandler, 3000, '');

  // Invoke a method on any instance of an application named "specific-app".
  ZeroMemory(@instance, sizeof(instance));
  instance.ApplicationName := 'specific-app';
  SetLength(instances, 1);
  instances[0] := instance;
  psaInstances := CreateInstanceArray_SA(instances);

  G42.InvokeMethods('DelphiAdd', psaArgs, psaInstances, False,
    GlueInstanceIdentity_ApplicationName, MethodAddHandler, 1000, '');
  SafeArrayDestroy(psaInstances);
...
```

### Handling Invocation Results

To handle the results from Interop method invocations, implement the [`IGlueInvocationResultHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglueinvocationresulthandler). This can be simplified by using the [`TGlueResultHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#glue42_helper_unit-classes_for_handling_events-tglueresulthandler) class of the [Glue42 Helper Unit](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#glue42_helper_unit).

The following example demonstrates how to set up the invocation result handler:

```delphi
TMainForm = class(TForm, IGlueContextHandler)
...
protected
  MethodAddHandler: TGlueResultHandler;
  procedure HandleAddResult(Sender: TGlueResultHandler;
    GlueInvResults: TGlueInvocationResultArray;
    Cookie: TCallbackCookie;
    const correlationId: WideString);
  ...
procedure TMainForm.InitializeGlue;
  ...
  // Create a method handler linked to the `HandleAddResult` procedure.
  MethodAddHandler := TGlueResultHandler.Create(nil, HandleAddResult);
  ...
```

When invoking multiple instances at once, the result handler will receive the result from each invocation in an element of the `GlueInvResults` array. If there aren't any available instances that match the targeting criteria, a single error result will be received by the handler.

The following example demonstrates a sample implementation of the result handler:

```delphi
procedure TMainForm.HandleAddResult(Sender: TGlueResultHandler;
  GlueInvResults: TGlueInvocationResultArray;
  Cookie: TCallbackCookie;
  const correlationId: WideString);
var
  gResult: GlueResult;
  data: TGlueContextValueArray;
  sum: Int64;
begin
  gResult := GlueInvResults[0].result;
  if gResult.Status <> GlueMethodInvocationStatus_Succeeded then begin
    // Invocation failed, more information may be available in `gResult.Message`.
    Exit;
  end;

  // Translate the result values.
  data := SA_AsTranslatedContextValues(gResult.Values);
  if (Length(data) <> 1) or (data[0].Name <> 'sum') then begin
    // Unexpected result.
    Exit;
  end;

  sum := data[0].Value.LongValue;
end;
```

## Discovery

Your application can discover registered Interop methods and streams and other applications (Interop servers) which offer them.

### Listing All Methods and Streams

To find all registered Interop methods/streams, use use [`GetAllMethods`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-getallmethods) method:

```delphi
var
  saMethods: PSafeArray;
  methods: array of GlueContext;
begin
  saMethods := G42.GetAllMethods();
  methods := SA_AsGlueContextArray(saMethods);
  ...
  SafeArrayDestroy(saMethods);
end;
```

### Listing Methods and Streams for Target

To find all Interop methods/streams registered by a single or multiple applications, use use [`GetMethodNamesForTarget`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-getmethodnamesfortarget). The method accepts a regular expression to match against registered application names as a required parameter.

The following example shows how to list all Interop methods and streams registered by applications with names starting with "client":

```delphi
var
  saMethods: PSafeArray;
  methods: array of GlueContext;
begin
  saMethods := G42.GetMethodNamesForTarget('^client.*');
  methods := SA_AsGlueContextArray(saMethods);
  ...
  SafeArrayDestroy(saMethods);
end;
```

### Listing All Interop Servers

To get a list of the names of all applications offering Interop methods/streams, use the [`GetTargets`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-gettargets) method:

```delphi
var
  saTargetNames: PSafeArray;
  targetNames: TStrArray;
begin
  saTargetNames := G42.GetTargets();
  targetNames := SA_AsStringArray(saTargetNames);
  ...
  SafeArrayDestroy(saTargetNames);
end;
```

## Streaming

### Overview

Your application can publish events that can be observed by other applications, or it can provide real-time data (e.g., market data, news alerts, notifications, etc.) to other applications by publishing an Interop stream. Your application can also receive and react to these events and data by creating an Interop stream subscription.

Applications that create and publish to Interop Streams are called *publishers*, and applications that subscribe to Interop Streams are called *subscribers*. An application can be both.

<glue42 name="diagram" image="../../../../images/interop/interop-streaming.gif">

Interop Streams are used extensively in [**Glue42 Enterprise**](https://glue42.com/enterprise/) products and APIs:

- in Glue42 Windows - to publish notifications about window status change (events);
- in application configuration settings - to publish application configuration changes, and notifications about application instance state change (events);
- in the Glue42 Notification Service (GNS) Desktop Manager and GNS Interop Servers - to publish Notifications (real-time data);
- in the Window Management and Application Management APIs (events);

## Publishing Stream Data

To expose a data stream to which other applications can subscribe, you must register a stream and provide implementations for handling the server side streaming events (subscription requests, added/removed subscribers). Once a stream has been successfully registered, the publishing application can start pushing data to it.

### Creating Streams

To register Interop streams, use the [`RegisterStream`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-registerstream) method after Glue42 has been initialized. To handle subscription requests, implement the [`IGlueSubscriptionHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluesubscriptionhandler) interface.

The following example demonstrates registering an Interop streaming method named "DelphiStream":

```delphi
TMainForm = class(TForm, IGlueSubscriptionHandler)
...
private
  G42: IGlue42;
  // Handles for the resulting stream and the Interop streaming method.
  delphiStream: IGlueStream;
  deplhiStreamMethod: GlueMethod;
protected
  // Implement `IGlueSubscriptionHandler`.
  function HandleSubscriptionRequest(stream: GlueMethod;
    caller: GlueInstance;
    requestValues: PSafeArray;
    const callback: IGlueServerSubscriptionCallback): HResult; stdcall;

  function HandleSubscriber(subscriberInstance: GlueInstance;
    const glueStreamSubscriber: IGlueStreamSubscriber;
    requestValues: PSafeArray): HResult; stdcall;

  function HandleSubscriberLost(streamSubscriber: GlueInstance;
    const glueStreamSubscriber: IGlueStreamSubscriber): HResult; stdcall;
...
  procedure TMainForm.InitializeGlue;
    ...
    G42.Start(inst);
    // Register the Interop stream.
    delphiStreamMethod := G42.RegisterStream('DelphiStream', Self, '', '', nil, delphiStream);
...
```

### Accepting or Rejecting Subscription Requests

The implementation of the `HandleSubscriptionRequest` callback method allows you to handle subscription requests. Use the `Accept` and `Reject` methods of the [`IGlueServerSubscriptionCallback`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglueserversubscriptioncallback) instance passed to `HandleSubscriptionRequest`:

```delphi
function TMainForm.HandleSubscriptionRequest(stream: GlueMethod;
  caller: GlueInstance;
  requestValues: PSafeArray;
  const callback: IGlueServerSubscriptionCallback): HResult; stdcall;
var
  reqArgs: GlueContextValueArray;
  gResult: GlueResult;
begin
  // Get the request arguments passed by the caller.
  reqArgs := SA_AsGlueContextValueArray(requestValues);
  if (Length(reqArgs) > 0) and (reqArgs[0].Name = 'rejectme') then begin
    ZeroMemory(@gResult, sizeof(gResult));
    gResult.Message := 'Rejected as per request.';
    // Reject the subscriber.
    callback.Reject(gResult);
    Result := S_OK;
    Exit;
  end;

  // Accept the subscriber.
  ZeroMemory(@gResult, sizeof(gResult));
  callback.Accept('', gResult);
  Result := S_OK;
end;
```

### Added or Removed Subscriptions

#### Handling New Subscriptions

The `HandleSubscriber` callback method is invoked when a new subscriber has been accepted. As new subscribers won't automatically get the data that has been previously published to the stream, this handler can be used to privately push some initial data to the new subscriber only. Use the `Push` method of the [`IGlueStreamSubscriber`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestreamsubscriber) instance to send the data to the subscriber:

```delphi
function TMainForm.HandleSubscriber(subscriberInstance: GlueInstance;
  const glueStreamSubscriber: IGlueStreamSubscriber;
  requestValues: PSafeArray): HResult; stdcall;
var
  psaData: PSafeArray;
begin

  // Put the inital data into a `PSafeArray`.
  psaData := CreateContextValues_SA(AsGlueContextValueArray([
     CreateContextValue('initialData', CreateValue('Welcome!'))
    ]));
  // Push the data privately to the new subscriber.
  glueStreamSubscriber.Push(psaData);
  SafeArrayDestroy(psaData);
  Result := S_OK;
end;
```

#### Handling Removed Subscriptions

The `HandleSubscriberLost` callback method is invoked when a stream subscription has been terminated. The following example demonstrates a minimal implementation:

```delphi
function TMainForm.HandleSubscriberLost(streamSubscriber: GlueInstance;
  const glueStreamSubscriber: IGlueStreamSubscriber): HResult; stdcall;
begin
  Result := S_OK;
end;
```

### Pushing Data

You can push data to a stream by using the [`Push`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestream-push) method of an [`IGlueStream`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestream) instance. Data can be sent to all subscribers on the stream, or to a group of subscribers on a specific stream branch.

The following example demonstrates how to push data to all subscribers on a stream:

```delphi
var
  psaData: PSafeArray;
  value01: TGlueContextValue;
  value02: TGlueContextValue;
begin
  value01 := CreateContextValue('instrument', CreateValue('GOOG'));
  value02 := CreateContextValue('price', CreateValue(1764.70));
  psaData := CreateContextValues_SA(AsGlueContextValueArray([value01,value02]));
  // Pushing data to the Interop stream.
  delphiStream.Push(psaData,'');
  SafeArrayDestroy(psaData);
end;
```

You can also push data directly to a subscriber by using the [`Push`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestreamsubscriber-push) method of an [`IGlueStreamSubscriber`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestreamsubscriber) instance (see [Handling New Subscriptions](#publishing_stream_data-added_or_removed_subscriptions-handling_new_subscriptions)).

### Using Stream Branches

Using stream branches allows you to group subscribers by any criterion and target stream data at specific groups of subscribers. Branches are distinguished by their name (key). Each Glue42 stream has a default (unnamed) branch on which it accepts subscribers and to which it pushes data if no branch is specified.

To accept a subscription on a branch, specify the branch name when accepting the subscription. If the branch doesn't exist, it will be automatically created:

```delphi
function TMainForm.HandleSubscriptionRequest(stream: GlueMethod;
  caller: GlueInstance;
  requestValues: PSafeArray;
  const callback: IGlueServerSubscriptionCallback): HResult; stdcall;
var
  gResult: GlueResult;
begin
  // Accept the subscriber on branch "branch01".
  ZeroMemory(@gResult, sizeof(gResult));
  callback.Accept('branch01',gResult);
  Result := S_OK;
end;
```
To push data to subscribers on a specific branch, specify a branch name when pushing data to the stream:

```delphi
delphiStream.Push(psaData, 'branch01');
```

*Note that the branch must have been previously created by accepting at least one subscriber on it.*

To list the names of all branches of an Interop stream, use the [`GetBranchKeys`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestream-getbranchkeys) method:

```delphi
var
  psaBranchNames: PSafeArray;
  branchNames: TStrArray;
begin
  psaBranchNames := delphiStream.GetBranchKeys();
  branchNames := SA_AsStringArray(psaBranchNames);
  ...
  SafeArrayDestroy(psaBranchNames);
end;
```

## Consuming Stream Data

To receive data published on an Interop stream, an application must subscribe to it using the [`SubscribeStream`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-subscribestream) or [`SubscribeStreams`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-subscribestreams) methods, and implement the [`IGlueStreamHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluestreamhandler) interface to be able to handle stream-related events. This can be simplified by using the [`TGlueStreamHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#glue42_helper_unit-classes_for_handling_events-tgluestreamhandler) class of the [Glue42 Helper Unit](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#glue42_helper_unit).

### Subscribing to Streams

To subscribe to an Interop stream offered by a specific application instance, use the [`SubscribeStream`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-subscribestream) method. To subscribe to an Interop stream offered by one or more instances, use the [`SubscribeStreams`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-subscribestreams) method.

The following example demonstrates how to subscribe to the "DelphiStream" Interop stream by targeting the first instance that has registered it, and how to set up the related data handler:

```delphi
TMainForm = class(TForm)
...
private
  G42: IGlue42;
protected
  // Stream data handling.
  procedure HandleDelphiStreamData(Method: GlueMethod;
    data: GlueContextValueArray;
    dataAsSA: PSafeArray);
...
  procedure TMainForm.InitializeGlue;
  var
    streamMethod: GlueMethod;
...
  G42.Start(inst);
  // Subscribe to the stream and create a stream data handler linked to the `HandleDelphiStreamData` procedure.
  G42.SubscribeStreams('DelphiStream', nil, nil, false,
    GlueInstanceIdentity_None,
    TGlueStreamHandler.Create(HandleDelphiStreamData), 0);
...
```

### Handling Subscriptions Client Side

The following example demonstrates a sample procedure for handling Interop stream data:

```delphi
procedure TMainForm.HandleDelphiStreamData(Method: GlueMethod;
  data: GlueContextValueArray;
  dataAsSA: PSafeArray);
var
  I: integer;
  gValue: GlueValue;
  initialData: WideString;
  instrument: WideString;
  price: Double;
begin
  initialData := '';
  instrument := '';
  price := -1;
  for I := Low(data) to High(data) do
  begin
    gValue := data[I].Value;
    if data[I].Name = 'initialData' then begin
      initialData := gValue.StringValue;
    end;
    if data[I].Name = 'instrument' then begin
      instrument := gValue.StringValue;
    end;
    if data[I].Name = 'price' then begin
      if (gValue.GlueType = GlueValueType_Long)
      or (gValue.GlueType = GlueValueType_int) then begin
        price := gValue.LongValue;
      end;
      if gValue.GlueType = GlueValueType_Double then begin
        price := gValue.DoubleValue;
      end;
    end;
  end;

  // Use the data.
  ...
end;
```