## Overview

*See the [Delphi 10](https://github.com/Glue42/native-examples/tree/main/glue-com/GlueDelphi) and [Delphi 7](https://github.com/Glue42/native-examples/tree/main/glue-com/GlueDelphi7) examples on GitHub which demonstrate various [**Glue42 Enterprise**](https://glue42.com/enterprise/) features.*

The Glue42 COM library allows you to Glue42 enable your Delphi applications, integrate them with other Glue42 enabled applications in [**Glue42 Enterprise**](https://glue42.com/enterprise/) and use Glue42 functionality in them. To access Glue42 functionalities in your Delphi application, you have to reference and initialize the Glue42 COM library. Currently, the Glue42 COM library supports Delphi 7 and Delphi 10.

## Using the Glue42 COM Library

### Referencing

The [Glue42 COM](https://www.nuget.org/packages/GlueCOM/) library is available as a package in NuGet which you can include in your projects. The Glue42 COM library is also distributed with [**Glue42 Enterprise**](https://glue42.com/enterprise/) and is usually located in the `%LocalAppData%\Tick42\GlueSDK\GlueCOMv2` folder.

To use any [**Glue42 Enterprise**](https://glue42.com/enterprise/) functionality, you need to add the following units to your Delphi project:

- `GlueCOM_TLB.pas` - declarations for the Glue42 COM type library;
- `mscorlib_TLB.pas` - declarations for the Microsoft Common Object Runtime library (the Glue42 COM type library depends on it);
- `GlueHelper.pas` - Delphi native types for wrapping low-level COM functionality;

*Note that you must not import the Glue42 COM type library (`GlueCOM.dll`) directly into your project. The provided `GlueCOM_TLB` unit has been modified to use specific alignment for some records. Importing the Glue42 COM type library will overwrite these required modifications.*

### Initialization

Initialize the Glue42 interface in the application main form by following these steps:

1. Import the `GlueCOM_TLB` unit:

```delphi
uses
  GlueCOM_TLB;
```

2. Declare a variable to hold the Glue42 COM object and a procedure to perform the initialization:

```delphi
type
  TMainForm = class(TForm)

  private
    // The Glue42 entry point handle.
    G42: IGlue42;
  protected
    procedure InitializeGlue;
```

3. Implement the initialization:

```delphi
procedure TMainForm.InitializeGlue;
var
  inst: GlueInstance;
  cfg: GlueConfiguration;
begin
  try
    // Create the Glue42 COM object.
    G42 := CoGlue42.Create() as IGlue42;

    // Configure own identity.
    ZeroMemory(@inst, sizeof(inst));
    inst.ApplicationName := 'MyDelphiApp';
    inst.Metadata := nil;
    ZeroMemory(@cfg, sizeof(cfg));
    cfg.AppDefinitionTitle := 'My Delphi App';
    G42.OverrideConfiguration(cfg);

    // Initialize and start Glue42.
    G42.Start(inst);
  except
    on E: Exception do
    begin
      // Handle errors.
    end;
  end;
end;
```

4. You can invoke `InitializeGlue` in the `OnCreate` event handler and perform cleanup in the `OnClose` event handler for the form:

```delphi
procedure TMainForm.FormCreate(Sender: TObject);
begin
  InitializeGlue;
end;

procedure TMainForm.FormClose(Sender: TObject; var Action: TCloseAction);
begin
  if Assigned(G42) then
  begin
    G42.Stop;
  end;
end;
```

## App Configuration

To add your Delphi application to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you must create a JSON file with application configuration. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

The following is an example configuration for a Delphi app:

```json
{
    "title": "My Delphi App",
    "type": "exe",
    "name": "my-delphi-app",
    "icon": "https://example.com/delphi-logo.jpg",
    "details": {
        "path": "%GDDIR%/../Demos/DelphiDemo/",
        "command": "MyDelphiApp.exe",
        "parameters": " --mode=1"
    }
}
```

| Property | Description |
|----------|-------------|
| `"type"` | Must be `"exe"`. |
| `"path"` | The path to the application - relative or absolute. You can also use the `%GDDIR%` environment variable, which points to the [Glue42 Enterprise](https://glue42.com/enterprise/) installation folder. |
| `"command"` | The actual command to execute (the EXE file name). |
| `"parameters"` | Specifies command line arguments. |

## Glue42 Delphi Concepts

Once the Glue42 COM library has been initialized, your application has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [App Management](../../../../glue42-concepts/application-management/delphi/index.html)
- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/delphi/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/delphi/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/delphi/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/delphi/index.html)

The following sections explain concepts related to using the Glue42 COM library.

### Glue42 Time

Glue42 time is essentially a Unix timestamp: the number of milliseconds since the Unix epoch, i.e. `1970-01-01 00:00:00 UTC`, ignoring leap seconds.

Glue42 timestamp parameters and values are stored in an `Int64` type. To convert from Glue42 time to Delphi `TDateTime` and vice versa, use the following functions defined in the `GlueHelper` unit:

```delphi
function GlueTimeToDateTime(msecs: Int64): TDateTime;
function DateTimeToGlueTime(dt: TDateTime): Int64;
```

*Note that Glue42 uses UTC internally, so the resulting `TDateTime` timestamp will also be in UTC, and not in local time.*

## Glue42 Helper Unit

The `GlueHelper` unit contains native [type definitions](#glue42_helper_unit-types) and additional helper classes and methods facilitating the use of the Glue42 COM library.

It is recommended to use the [conversion functions](#glue42_helper_unit-working_with_psafearray-conversion_functions) provided in the `GlueHelper` unit to transform the parameters or return values from a `PSafeArray` to native types and vice-versa:

- The functions in the `Create[type]_SA` format can be used to transform various native types to a `PSafeArray`. The returned values need to be destroyed with `SafeArrayDestroy` when no longer needed.

- The functions in the `SA_As[type]` format can be used to transform a `PSafeArray` to various native types.

### Types

#### Array Types

The following array types are defined in the `GlueHelper` unit:

| Array | Type |
|-------|------|
| `GlueContextArray` | [`GlueContext`](#types-gluecontext) |
| `GlueContextValueArray` | [`GlueContextValue`](#types-gluecontextvalue) |
| `GlueStreamSubscriberArray` | [`IGlueStreamSubscriber`](#interfaces-igluestreamsubscriber) |
| `GlueValueArray` | [`GlueValue`](#types-gluevalue) |
| `TDateTimeArray` | `TDateTime` |
| `TDoubleArray` | `Double` |
| `TGlueContextValueArray` | [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue) |
| `TGlueInstanceArray` | [`GlueInstance`](#types-glueinstance) |
| `TGlueInvocationResultArray` | [`GlueInvocationResult`](#types-glueinvocationresult) |
| `TGlueMethodArray` | [`GlueMethod`](#types-gluemethod) |
| `TGlueValueArray` | [`TGlueValue`](#glue42_helper_unit-types-tgluevalue) (pointer) |
| `TInt64Array` | `Int64` |
| `TIntArray` | `Integer` |
| `TStrArray` | `String` |
| `TWideStringArray` | `WideString` |
| `TWordBoolArray` | `WordBool` |

#### Pointer Types

The following pointer types are defined in the `GlueHelper` unit:

| Type | Points to |
|------|-----------|
| `PGlueContextValue` | [`GlueContextValue`](#types-gluecontextvalue) |
| `PGlueInvocationResult` | [`GlueInvocationResult`](#types-glueinvocationresult) |
| `PGlueMethod` | [`GlueMethod`](#types-gluemethod) |
| `PGlueResult` | [`GlueResult`](#types-glueresult) |
| `PGlueValue` | [`GlueValue`](#types-gluevalue) |
| `PTGlueValue` | [`TGlueValue`](#glue42_helper_unit-types-tgluevalue) |

#### Native Record Types

#### TGlueContextValue

This is a translated version (i.e. not using `PSafeArray` directly or indirectly) of [`GlueContextValue`](#types-gluecontextvalue) representing a name-value pair.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the value. |
| `Value` | `PTGlueValue` | Pointer to [`TGlueValue`](#glue42_helper_unit-types-tgluevalue). |

#### TGlueValue

This is a translated version (i.e. not using `PSafeArray` directly or indirectly) of [`GlueValue`](#types-gluevalue) representing an elementary or composite value.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `GlueType` | [`GlueValueType`](#enums-gluevaluetype) | Type of the Glue42 value. |
| `IsArray` | `WordBool` | Indicates whether the value is an array. |

The following properties will be initialized according to `GlueType` and `IsArray`:

| Name | Type | Description |
|------|------|-------------|
| `BoolValue` | `WordBool` | Boolean value. |
| `LongValue` | `Int64` | Integer value. |
| `DoubleValue` | `Double` | Double-precision floating-point value. |
| `StringValue` | `WideString` | String value. |
| `BoolArray` | `TWordBoolArray` | Array of `WordBool` values. |
| `LongArray` | `TInt64Array` | Array of `Int64` values. |
| `DoubleArray` | `TDoubleArray` | Array of `Double` values. |
| `StringArray` | `TStrArray` | Array of `String` values. |
| `DateTimeArray` | `TDateTimeArray` | Array of `TDateTime` values. |
| `Tuple` | `TGlueValueArray` | Array of `TGlueValue` values. |
| `CompositeValue` | `TGlueContextValueArray` | Array of [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue) values. |

### Working with PSafeArray

The `GlueHelper` unit provides a set of functions for converting from/to `PSafeArray` which is widely used when sending or receiving data from Glue42.

#### Summary

The table below summarizes the available functions to convert from/to `PSafeArray` based on the content type.

| Array Type | Array of | From | To |
|------------|----------|------|----|
| `TDoubleArray` | `Double` | `SA_AsDoubleArray` | `CreateArray_SA` |
| `TInt64Array` | `Int64` | `SA_AsInt64Array` | `CreateArray_SA` |
| `TStrArray` | `String` | `SA_AsStringArray` | `CreateArray_SA` |
| `TWideStringArray` | `WideString` | `SA_AsWideStringArray` | `CreateArray_SA` |
| `TWordBoolArray` | `WordBool` | `SA_AsWordBoolArray` | `CreateArray_SA` |
| `TDateTimeArray` | `TDateTime` | `SA_AsDateTimeArray`<br>*Note that the safe array is of `Int64` values representing Glue42 time.* | `-` |
| `GlueStreamSubscriberArray` | [`IGlueStreamSubscriber`](#interfaces-igluestreamsubscriber) | `SA_AsGlueStreamSubscriberArray` | `-` |
| `GlueContextArray` | [`GlueContext`](#types-gluecontext) | `SA_AsGlueContextArray` | `-` |
| `GlueContextValueArray` | [`GlueContextValue`](#types-gluecontextvalue) | `SA_AsGlueContextValueArray` | `CreateContextValues_SA` |
| `TGlueContextValueArray` | [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue) | `SA_AsTranslatedContextValues` | `AsGlueContextValueArray`, then `CreateContextValues_SA` |
| `TGlueInstanceArray` | [`GlueInstance`](#types-glueinstance) | `SA_AsGlueInstanceArray` | `CreateInstanceArray_SA` |
| `TGlueInvocationResultArray` | [`GlueInvocationResult`](#types-glueinvocationresult) | `SA_AsGlueInvocationResultArray` | `-` |
| `TGlueMethodArray` | [`GlueMethod`](#types-gluemethod) | `SA_AsGlueMethodArray` | `-` |
| `GlueValueArray` | [`GlueValue`](#types-gluevalue) | `SA_AsGlueValueArray` | `CreateTuple_SA` |

#### Conversion Functions

All conversion functions take a single parameter of the respective type. The returned `PSafeArray` must be destroyed with `SafeArrayDestroy` when no longer needed.

| Function | Parameter Type | Return Type | Array of Type |
|----------|----------------|-------------|------------ ---|
| `AsGlueContextValueArray` | `TGlueContextValueArray` | `GlueContextValueArray` | [`GlueContextValue`](#types-gluecontextvalue) |
| `CreateArray_SA` | `TDoubleArray` | `PSafeArray` | `Double` |
| `CreateArray_SA` | `TInt64Array` | `PSafeArray` | `Int64` |
| `CreateArray_SA` | `TStrArray` | `PSafeArray` | `String` |
| `CreateArray_SA` | `TWideStringArray` | `PSafeArray` | `WideString` |
| `CreateArray_SA` | `TWordBoolArray` | `PSafeArray` | `WordBool` |
| `CreateContextValues_SA` | `GlueContextValueArray` | `PSafeArray` | [`GlueContextValue`](#types-gluecontextvalue) |
| `CreateTuple_SA` | `GlueValueArray` | `PSafeArray` | [`GlueValue`](#types-gluevalue) |
| `SA_AsDateTimeArray` | `PSafeArray` | `TDateTimeArray` | `TDateTime` |
| `SA_AsDoubleArray` | `PSafeArray` | `TDoubleArray` | `Double` |
| `SA_AsGlueContextArray` | `PSafeArray` | `GlueContextArray` | [`GlueContext`](#types-gluecontext) |
| `SA_AsGlueContextValueArray` | `PSafeArray` | `GlueContextValueArray` | [`GlueContextValue`](#types-gluecontextvalue) |
| `SA_AsGlueInstanceArray` | `PSafeArray` | `TGlueInstanceArray` | [`GlueInstance`](#types-glueinstance) |
| `SA_AsGlueInvocationResultArray` | `PSafeArray` | `TGlueInvocationResultArray` | [`GlueInvocationResult`](#types-glueinvocationresult) |
| `SA_AsGlueMethodArray` | `PSafeArray` | `TGlueMethodArray` | [`GlueMethod`](#types-gluemethod) |
| `SA_AsGlueStreamSubscriberArray` | `PSafeArray` | `GlueStreamSubscriberArray` | [`IGlueStreamSubscriber`](#interfaces-igluestreamsubscriber) |
| `SA_AsGlueValueArray` | `PSafeArray` | `GlueValueArray` | [`GlueValue`](#types-gluevalue) |
| `SA_AsInt64Array` | `PSafeArray` | `TInt64Array` | `Int64` |
| `SA_AsStringArray` | `PSafeArray` | `TStrArray` | `String` |
| `SA_AsTranslatedContextValues` | `PSafeArray` | `TGlueContextValueArray` | [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue) |
| `SA_AsWideStringArray` | `PSafeArray` | `TWideStringArray` | `WideString` |
| `SA_AsWordBoolArray` | `PSafeArray` | `TWordBoolArray` | `WordBool` |

#### Functions for Creating Glue42 and Context Values

#### CreateContextValue

This function creates a [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue) representing a name-value pair. It can be put in a `TGlueContextValueArray` and eventually be converted to a `PSafeArray` in order to be sent to Glue42.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `string` | String representing the name in the name-value pair. |
| `Value` | [`TGlueValue`](#glue42_helper_unit-types-tgluevalue)| The value of the name-value pair. |

*Return value:* [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue)

#### CreateValue

This is a set of overloaded functions for creating [`TGlueValue`](#glue42_helper_unit-types-tgluevalue) values from various types. The overloads accepting arrays will create composite values. Each function accepts the following types as a single parameter:

- `Double`
- `Int64`
- `Integer`
- `String`
- `TDoubleArray`
- `TInt64Array`
- `TStrArray`

*Return value:* [`TGlueValue`](#glue42_helper_unit-types-tgluevalue)

#### CreateComposite

This function creates a composite value from a `TGlueContextValueArray`.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Value` | `TGlueContextValueArray` | An array of [`TGlueContextValue`](#glue42_helper_unit-types-tgluecontextvalue) values representing the contents of the composite value. |
| `IsArray` | `Bool` | Specifies whether the created composite value is an array. |

*Return value:* [`TGlueValue`](#glue42_helper_unit-types-tgluevalue)

#### CreateTuple

This function creates a composite value representing a tuple, i.e. an array of (possibly) heterogeneous elements.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Value` | `TGlueValueArray` | An array of [`TGlueValue`](#glue42_helper_unit-types-tgluevalue) values representing the elements of the tuple. |

*Return value:* [`TGlueValue`](#glue42_helper_unit-types-tgluevalue)

### Classes for Handling Events

A set of classes that implement callback interfaces in order to simplify the implementation of callback event handlers.

#### TGlueRequestHandler

This class implements the `IGlueRequestHandler` interface which invokes a callback whenever an already registered [Interop method is invoked](../../../../glue42-concepts/data-sharing-between-apps/interop/delphi/index.html#method_invocation).

*Class constructor parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Cookie` | `TCallbackCookie` | Optional pointer to custom data which will be passed to the callback procedure. |
| `handlerLambda` | `TRequestHandlerLambda` | The procedure which will be invoked when the associated Interop method is invoked. |

*Callback procedure signature:*

```delphi
TRequestHandlerLambda = procedure(Sender: TGlueRequestHandler;
    Method: GlueMethod;
    Instance: GlueInstance;
    Args: GlueContextValueArray;
    callback: IGlueServerMethodResultCallback;
    Cookie: TCallbackCookie;
    argsSA: PSafeArray) of object;
```

#### TGlueResultHandler

This class implements the `IGlueInvocationResultHandler` interface which invokes a callback when the result from an Interop method invocation becomes available.

*Class constructor parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Cookie` | `TCallbackCookie` | Optional pointer to custom data which will be passed to the callback procedure. |
| `handlerLambda` | `TResultHandlerLambda` | The procedure which will be invoked when the result from the associated Interop method invocation becomes available. |

*Callback procedure signature:*

```delphi
TResultHandlerLambda = procedure(Sender: TGlueResultHandler;
    GlueResult: TGlueInvocationResultArray;
    Cookie: TCallbackCookie;
    const correlationId: WideString) of object;
```

#### TGlueStreamHandler

This class implements the `IGlueStreamHandler` interface which invokes callbacks for events related to [Interop streaming](../../../../glue42-concepts/data-sharing-between-apps/interop/delphi/index.html#streaming) methods.

*Class constructor parameters:*

| Name | Type | Description |
|------|------|-------------|
| `dataLambda` | `TStreamDataLambda` | The procedure which will be invoked when data is pushed to the stream and to the subscriber. |

*Callback procedure signature:*

```delphi
TStreamDataLambda = procedure(Method: GlueMethod;
    data: GlueContextValueArray;
    dataAsSA: PSafeArray) of object;
```

## COM/Delphi Reference

This reference describes the components in the Glue42 COM library relevant to Delphi.

*Note that the library also contains components that aren't intended to be directly used by Delphi applications.*

## Enums

### GlueInstanceIdentity

| Name | Value | Hex | Binary |
|------|-------|-----|--------|
| `GlueInstanceIdentity_None` | 0 | 00 | 00000000 |
| `GlueInstanceIdentity_MachineName` | 1 | 01 | 00000001 |
| `GlueInstanceIdentity_ApplicationName` | 2 | 02 | 00000010 |
| `GlueInstanceIdentity_UserName` | 4 | 04 | 00000100 |
| `GlueInstanceIdentity_Instance` | 7 | 07 | 00000111 |
| `GlueInstanceIdentity_Environment` | 8 | 08 | 00001000 |
| `GlueInstanceIdentity_Region` | 16 | 10 | 00010000 |
| `GlueInstanceIdentity_LocalizedInstance` | 31 | 1F | 00011111 |
| `GlueInstanceIdentity_ServiceName` | 32 | 20 | 00100000 |
| `GlueInstanceIdentity_Full` | 63 | 3F | 00111111 |
| `GlueInstanceIdentity_Pid` | 64 | 40 | 01000000 |
| `GlueInstanceIdentity_InstanceId` | 128 | 80 | 10000000 |

### GlueMethodFlags

| Name | Value | Hex | Binary |
|------|-------|-----|--------|
| `GlueMethodFlags_None` | 0 | 00 | 00000000 |
| `GlueMethodFlags_ReturnsResult` | 1 | 01 | 00000001 |
| `GlueMethodFlags_IsGuiOperation` | 2 | 02 | 00000010 |
| `GlueMethodFlags_IsUserSpecific` | 4 | 04 | 00000100 |
| `GlueMethodFlags_IsMachineSpecific` | 8 |08 |00001000 |
| `GlueMethodFlags_OutsideDomain` | 16 | 10 | 00010000 |
| `GlueMethodFlags_SupportsStreaming` | 32 | 20 | 00100000 |

### GlueMethodInvocationStatus

| Name | Value |
|------|-------|
| `GlueMethodInvocationStatus_Succeeded` | 0 |
| `GlueMethodInvocationStatus_Failed` | 1 |
| `GlueMethodInvocationStatus_TimedOut` | 2 |
| `GlueMethodInvocationStatus_NotAvailable` | 3 |
| `GlueMethodInvocationStatus_Started` | 4 |

### GlueState

| Name | Value |
|------|-------|
| `GlueState_Unknown` | 0 |
| `GlueState_Pending` | 1 |
| `GlueState_Connected` | 2 |
| `GlueState_Disconnected` | 3 |
| `GlueState_Inactive` | 4 |

### GlueStreamState

| Name | Value |
|------|-------|
| `GlueStreamState_Pending` | 0 |
| `GlueStreamState_Stale` | 1 |
| `GlueStreamState_Opened` | 2 |
| `GlueStreamState_Closed` | 3 |
| `GlueStreamState_SubscriptionRejected` | 4 |
| `GlueStreamState_SubscriptionFailed` | 5 |

### GlueValueType

| Name | Value |
|------|-------|
| `GlueValueType_Bool` | 0 |
| `GlueValueType_Int` | 1 |
| `GlueValueType_Double` | 2 |
| `GlueValueType_Long` | 3 |
| `GlueValueType_String` | 4 |
| `GlueValueType_DateTime` | 5 |
| `GlueValueType_Tuple` | 6 |
| `GlueValueType_Composite` | 7 |

### GlueWindowEventType

*Note that some event types aren't applicable to Delphi applications.*

| Name | Value | Hex |
|------|-------|-----|
| `GlueWindowEventType_Unknown` | 0 | 00 |
| `GlueWindowEventType_Snapshot` | 1 | 01 |
| `GlueWindowEventType_WindowFrameAdded` | 2 | 02 |
| `GlueWindowEventType_WindowFrameRemoved` | 3 | 03 |
| `GlueWindowEventType_WindowFrameChanged` | 4 | 04 |
| `GlueWindowEventType_ButtonClicked` | 5 | 05 |
| `GlueWindowEventType_ButtonRemoved` | 6 | 06 |
| `GlueWindowEventType_ButtonAdded` | 7 | 07 |
| `GlueWindowEventType_GroupHeaderVisibilityChanged` | 8 | 08 |
| `GlueWindowEventType_CompositionChanged` | 9 | 09 |
| `GlueWindowEventType_FrameColorChanged` | 10 | 0A |
| `GlueWindowEventType_Created` | 11 | 0B |
| `GlueWindowEventType_Closed` | 12 | 0C |
| `GlueWindowEventType_UrlChanged` | 13 | 0D |
| `GlueWindowEventType_ContextChanged` | 14 | 0E |
| `GlueWindowEventType_VisibilityChanged` | 15 | 0F |
| `GlueWindowEventType_BoundsChanged` | 16 | 10 |
| `GlueWindowEventType_StateChanged` | 17 | 11 |
| `GlueWindowEventType_FocusChanged` | 18 | 12 |
| `GlueWindowEventType_TitleChanged` | 19 | 13 |
| `GlueWindowEventType_WindowCanvasWindowChanged` | 20 | 14 |
| `GlueWindowEventType_TabHeaderVisibilityChanged` | 21 | 15 |
| `GlueWindowEventType_FrameSelectionChanged` | 22 | 16 |
| `GlueWindowEventType_ShowFlydownBoundsRequested` | 23 | 17 |
| `GlueWindowEventType_WindowZoomFactorChanged` | 24 | 18 |
| `GlueWindowEventType_FrameIsLockedChanged` | 25 | 19 |
| `GlueWindowEventType_DOMReady` | 26 | 1A |
| `GlueWindowEventType_Hibernated` | 27 | 1B |
| `GlueWindowEventType_Resumed` | 28 | 1C |

## Types

### GlueAppDefinition

Definition of a child application to be registered in Glue42.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Category` | `WideString` | Application category. |
| `Name` | `WideString` | Application name with which the app will be registered in Glue42. |
| `title` | `WideString` | Application title as it will appear in the UI of the Glue42 Application Manager. |

### GlueConfiguration

Used for overriding the default Glue42 configuration.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`OverrideConfiguration`](#interfaces-iglue42-overrideconfiguration) method.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `LoggingConfigurationPath` | `WideString` | Logging configuration path. |
| `GWUri` | `WideString` | Glue42 Gateway URI. |
| `AppDefinitionStartup` | `WideString` | Application startup file. |
| `AppDefinitionStartupArgs` | `WideString` | Application startup arguments. |
| `AppDefinitionTitle` | `WideString` | Application title. |

### GlueContext

Stores information about a Glue42 shared context.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name of the Glue42 shared context. |
| `Id` | `WideString` | Identifier of the Glue42 shared context. |

### GlueContextValue

This type represents a name-value pair.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the value. |
| `Value` | [`GlueValue`](#types-gluevalue)| A Glue42 value. |

### GlueInstance

Describes the identity of a Glue42 instance, i.e. how the instance is seen by other Glue42 peers.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `InstanceId` | `WideString` | Identifier of the Glue42 instance. |
| `Version` | `WideString` | Version reported by the application instance. |
| `MachineName` | `WideString` | Machine (network) name. |
| `ProcessId` | `Integer` | Process ID (PID). |
| `ProcessStartTime` | `Int64` | Glue42 time when the process was started. |
| `UserName` | `WideString` | User name associated with the process. |
| `ApplicationName` | `WideString` | Glue42 application name. |
| `Environment` | `WideString` | Glue42 environment. |
| `Region` | `WideString` | Glue42 region. |
| `ServiceName` | `WideString` | Glue42 service name. |
| `MetricsRepositoryId` | `WideString` | Glue42 metrics repository identifier. |
| `Metadata` | `PSafeArray` | Optional array of [`GlueContextValue`](#types-gluecontextvalue) values providing additional information about the instance. |

### GlueInvocationResult

Holds information about the invoked Interop method and the result it has returned.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Method` | [`GlueMethod`](#types-gluemethod) | Information about the Interop method returning the result. |
| `Result` | [`GlueResult`](#types-glueresult) | The result returned by the Interop method. |

### GlueMethod

Describes an Interop method.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Method name. |
| `Input` | `WideString` | *Optional.* String representation of the method input arguments signature. |
| `Output` | `WideString` | *Optional.* String representation of the method return value signature. |
| `Instance` | [`GlueInstance`](#types-glueinstance) | Information about the Glue42 application instance that has registered the Interop method. |
| `RegistrationCookie` | `WideString` | Method registration cookie. |
| `Flags` | [`GlueMethodFlags`](#enums-gluemethodflags) | Interop method flags. |
| `ObjectTypes` | `PSafeArray` | *Optional.* Array of `WideString` values specifying the types of objects that the method works with (e.g., `Instrument`, `Client`, etc.). |

### GlueResult

Holds information about the result from invoking an Interop method.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `Values` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the result. |
| `Status` | [`GlueMethodInvocationStatus`](#enums-gluemethodinvocationstatus) | Status of the method invocation. |
| `Message` | `WideString` | Message related to the method invocation status. |
| `LogDetails` | `WideString` | Log details related to the result. |

### GlueValue

Represents an elementary or composite value.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `GlueType` | [`GlueValueType`](#enums-gluevaluetype) | Type of the Glue42 value. |
| `IsArray` | `WordBool` | Indicates whether the value is an array. |

The following properties will be initialized according to `GlueType` and `IsArray`:

| Name | Type | Description |
|------|------|-------------|
| `BoolValue` | `WordBool` | Boolean value. |
| `LongValue` | `Int64` | Integer value. |
| `DoubleValue` | `Double` | Double-precision floating-point value. |
| `StringValue` | `WideString` | String value. |
| `BoolArray` | `PSafeArray` | Array of `WordBool` values. |
| `LongArray` | `PSafeArray` | Array of `Int64` values. |
| `DoubleArray` | `PSafeArray` | Array of `Double` values. |
| `StringArray` | `PSafeArray` | Array of `WideString` values. |
| `Tuple` | `PSafeArray` | Array of `GlueValue` values. |
| `CompositeValue` | `PSafeArray` | Array of [`GlueContextValue`](#types-gluecontextvalue) values. |

## Interfaces

### IAppAnnouncer

An instance of this is passed to the implementation of the [`CreateApp`](#interfaces-iappfactory-createapp) method of the [`IAppFactory`](#interfaces-iappfactory) interface.

**Methods**

#### AnnounceAppCreationFailure

Informs Glue42 that a new child application instance couldn't be created.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `error` | `WideString` | Text passed back to Glue42 as an error message. |

*Return value:* None

#### RegisterAppInstance

Registers a new child application instance in Glue42.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | `Integer` | Handle to the window representing the Glue42 application instance. |
| `glueApp` | [`IGlueApp`](#interfaces-iglueapp) | An object of a class implementing the `IGlueApp` interface. |

*Return value:* [`IGlueWindow`](#interfaces-igluewindow)

### IAppFactory

Implementing this interface allows an object to act as a child application factory.

*See also:*

- [`IAppFactoryRegistry`](#interfaces-iappfactoryregistry) - the [`RegisterAppFactory`](#interfaces-iappfactoryregistry-registerappfactory) method.

**Methods**

#### CreateApp

Creates a new instance of a child application.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `appDefName` | `WideString` | Name with which the child application has been registered. |
| `state` | [`GlueValue`](#types-gluevalue) | Saved application state. |
| `announcer` | [`IAppAnnouncer`](#interfaces-iappannouncer) | Object used for announcing to Glue42 successful or failed application creation. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IAppFactoryRegistry

This interface is used for registering application factories and application instances.

**Methods**

#### RegisterAppFactory

Registers an app factory as a creator of a child application with the provided definition.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `appDefinition` | [`GlueAppDefinition`](#types-glueappdefinition) | Definition of the app to register. |
| `factory` | [IAppFactory](#interfaces-iappfactory) | An object of a class implementing the `IAppFactory` interface. |

*Return value:* None

#### RegisterAppInstance

Register an application instance in Glue42.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `appDefName` | `WideString` | Application name. |
| `glueWindow` | [IGlueWindow](#interfaces-igluewindow) | A registered Glue42 Window. |
| `glueApp` | [IGlueApp](#interfaces-iglueapp) | An object of a class implementing the `IGlueApp` interface. |

*Return value:* None

### IGlue42

Create a single object of this class in order to access Glue42 functionality.

**Properties**

| Name | Type | Description |
|------|------|-------------|
| `AppFactoryRegistry` | [IAppFactoryRegistry](#interfaces-iappfactoryregistry) | Object for registering app factories and app instances. |

**Methods**

#### BuildAndInvoke

Invokes an Interop method on a single or multiple targets.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Method` | `WideString` | Name of the method to invoke. |
| `builderCallback` | [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) | An object of a class implementing the `IGlueContextBuilderCallback` interface. This can be used to build the method invocation arguments. |
| `targets` | `PSafeArray` | *Optional.* Allows filtering the invocation targets. If provided, this must be an array of [`GlueInstance`](#types-glueinstance) objects. |
| `all` | `WordBool` | Indicates whether the method should be invoked on all matching targets that offer it or only on the first one. |
| `identity` | [`GlueInstanceIdentity`](#enums-glueinstanceidentity) | Specifies the identity properties to be matched when applying the `targets` filter. |
| `resultHandler` | [`IGlueInvocationResultHandler`](#interfaces-iglueinvocationresulthandler) | An object of a class implementing the `IGlueInvocationResultHandler` interface. This is used to handle the result from the method invocation. |
| `invocationTimeoutMsecs` | `Int64` | The method invocation will time out after the specified number of milliseconds. If the provided value is less than or equal to zero, then the default timeout value will be used. |
| `correlationId` | `WideString`| *Optional.* A parameter that will be passed to the implementation of the [`HandleResult`](#interfaces-iglueinvocationresulthandler-handleresult) method of the [`IGlueInvocationResultHandler`](#interfaces-iglueinvocationresulthandler) interface. |

*Return value:* None

#### BuildGlueContextValues

Creates a `PSafeArray` of [`GlueContextValue`](#types-gluecontextvalue) values using a callback method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `contextBuilderCallback` | [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) | An object of a class implementing the `IGlueContextBuilderCallback` interface |

*Return value:* `PSafeArray`

An array of [`GlueContextValue`](#types-gluecontextvalue) values. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.


#### GetAllInstances

Gets an array of all available Glue42 instances.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`GlueInstance`](#types-glueinstance) objects. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetAllMethods

Gets an array of the available Interop methods from all Glue42 instances, including Interop streaming methods.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`GlueMethod`](#types-gluemethod) objects. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetChannels

Gets an array of the names of all available Glue42 Channels.

*Parameters:* None

*Return value:* `PSafeArray`

An array of `WideString` values with the Channel names. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetInstance

Gets the current Glue42 instance.

*Parameters:* None

*Return value:* [`GlueInstance`](#types-glueinstance)

Contains information about the current Glue42 application instance.

#### GetKnownContexts

Gets an array of all available Glue42 shared contexts.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`GlueContext`](#types-gluecontext) values. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetMethodNamesForTarget

Gets the names of the Interop methods exposed by an application.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `targetRegex` | `WideString` | *Optional.* Regular expression for filtering by application name. An empty string will match all application names. |

*Return value:* `PSafeArray`

An array of `WideString` values with the method names. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetMethodsForInstance

Gets the Interop methods exposed by matching one or more application instances.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Instance` | [`GlueInstance`](#types-glueinstance) | A Glue42 instance with one or more properties set to the values to match. |
| `identity` | [`GlueInstanceIdentity`](#enums-glueinstanceidentity) |  Specifies which properties set in the `Instance` parameter to be matched. |

*Return value:* `PSafeArray`

An array of [`GlueMethod`](#types-gluemethod) objects. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetStartingContext

Gets the starting context for the application.

*Parameters:* None

*Return value:* [`GlueValue`](#types-gluevalue)

A [`GlueValue`](#types-gluevalue) containing the starting context for the application.

#### GetStartupWindowSettings

Gets the default settings for the startup window.

*Parameters:* None

*Return value:* [`IGlueWindowSettings`](#interfaces-igluewindowsettings)

An object of [`IGlueWindowSettings`](#interfaces-igluewindowsettings) containing the default settings for the startup window.

#### GetTargets

Gets the names of all active applications (targets) currently offering Interop methods.

*Parameters:* None

*Return value:* `PSafeArray`

An array of `WideString` values with the application names. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### InvokeMethod

Invokes an Interop method on a specific single target.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Method` | [`GlueMethod`](#types-gluemethod) | Information about the method to invoke. |
| `invocationArgs` | `PSafeArray` | Method invocation arguments. This must be an array of [`GlueContextValue`](#types-gluecontextvalue) values. |
| `resultHandler` | [`IGlueInvocationResultHandler`](#interfaces-iglueinvocationresulthandler) | An object of a class implementing the `IGlueInvocationResultHandler` interface. This is used to handle the result from the method invocation. |
| `invocationTimeoutMsecs` | `Int64` | The method invocation will time out after the specified number of milliseconds. If the provided value is less than or equal to zero, then the default timeout value will be used. |
| `correlationId` | `WideString` | *Optional.* A parameter that will be passed to the implementation of the [`HandleResult`](#interfaces-iglueinvocationresulthandler-handleresult) method of the [`IGlueInvocationResultHandler`](#interfaces-iglueinvocationresulthandler) interface. |

*Return value:* None

#### InvokeMethods

Invokes an Interop method on a single or multiple targets.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Method` | `WideString` | Method name. |
| `invocationArgs` | `PSafeArray` | Method invocation arguments. This must be an array of [`GlueContextValue`](#types-gluecontextvalue) values. |
| `targets` | `PSafeArray` | *Optional.* Allows filtering the invocation targets. If provided, this must be an array of [`GlueInstance`](#types-glueinstance) objects. |
| `all` | `WordBool` | Indicates whether the method should be invoked on all matching targets that offer it or only on the first one. |
| `identity` | [`GlueInstanceIdentity`](#enums-glueinstanceidentity) | Specifies the identity properties to be matched when applying the `targets` filter. |
| `resultHandler` | [`IGlueInvocationResultHandler`](#interfaces-iglueinvocationresulthandler) | An object of a class implementing the `IGlueInvocationResultHandler` interface. This is used to handle the result from the method invocation. |
| `invocationTimeoutMsecs` | `Int64` | The method invocation will time out after the specified number of milliseconds. If the provided value is less than or equal to zero, then the default timeout value will be used. |
| `correlationId` | `WideString` | *Optional.* A parameter that will be passed to the implementation of the [`HandleResult`](#interfaces-iglueinvocationresulthandler-handleresult) method of the [`IGlueInvocationResultHandler`](#interfaces-iglueinvocationresulthandler) interface. |

*Return value:* None

#### InvokeSync

Invokes an Interop method synchronously.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `methodName` | `WideString` | Name of the method to invoke. |
| `argsAsJson` | `WideString` | Arguments in JSON format to pass to the method. |
| `resultFieldPath` | `WideString` | *Optional.* Field path. If provided, the return value will be the value of the specified field within the entire result structure. |
| `targetRegex` | `WideString` | *Optional.* Regular expression for filtering targets by application name. If provided, the method will be invoked on all targets with application name matching the regular expression. If not provided, the method will be invoked on a single target. |

*Return value:* `WideString`

The string represents the return values from the method invocation in JSON format.

#### IsLaunchedByGD

Determines whether the application was started by [**Glue42 Enterprise**](https://glue42.com/enterprise/) or as a standalone executable.

*Parameters:* None

*Return value:* `WordBool`

The string represents the return values from the method invocation in JSON format.

#### JsonToVariant

Converts a JSON string to a variant array.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `json` | `WideString` | The JSON string to convert. |

*Return value:* `PSafeArray`

An array of variants. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### Log

Outputs a message to the Glue42 log.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `level` | `Byte` | Log level. The following values are accepted: `0` (trace), `1` (debug), `2` (info), `3` (warn), `4` (error), `5` (fatal). |
| `Message` | `WideString` | The message text to log. |

*Return value:* None

#### OverrideConfiguration

Overrides the default Glue42 configuration. This method must be invoked before invoking the [`Start`](#interfaces-iglue42-start) method, otherwise it will have no effect.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `configuration` | [`GlueConfiguration`](#types-glueconfiguration) | The configuration to use. |

*Return value:* None

#### RegisterGlueWindow

Initiates the registration of a window as a Glue42 Window and sets the related event handler.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | `Integer` | Handle of the window to register. |
| `windowEventHandler` | `IGlueWindowEventHandler` | An object of a class implementing the [`IGlueWindowEventHandler`](#interfaces-igluewindoweventhandler) interface. |

*Return value:* [`IGlueWindow`](#interfaces-igluewindow)

An object representing the registered Glue42 Window.

*Note that the returned value shouldn't be used to interact with the Glue42 Window until the [`HandleWindowReady`](#interfaces-igluewindoweventhandler-handlewindowready) callback method is invoked (i.e. the registration is complete).*

#### RegisterGlueWindowWithSettings

Initiates the registration of a window as a Glue42 Window with specific settings and sets the related event handler.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | `Integer` | Handle of the window to register. |
| `settings` | [`IGlueWindowSettings`](#interfaces-igluewindowsettings) | An object of `IGlueWindowSettings` containing the settings to use during the window registration. |
| `windowEventHandler` | `IGlueWindowEventHandler` | An object of a class implementing the [`IGlueWindowEventHandler`](#interfaces-igluewindoweventhandler) interface. |

*Return value:* [`IGlueWindow`](#interfaces-igluewindow)

*Note that the returned value shouldn't be used to interact with the Glue42 Window until the [`HandleWindowReady`](#interfaces-igluewindoweventhandler-handlewindowready) callback method is invoked (i.e. the registration is complete).*

#### RegisterMethod

Registers an Interop method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `methodName` | `WideString` | Name for the method to register. |
| `requestHandler` | `IGlueRequestHandler` | An object of a class implementing the [`IGlueRequestHandler`](#interfaces-igluerequesthandler) interface. |
| `Input` | `WideString` | *Optional.* String representation of the method input arguments signature. |
| `Output` | `WideString` | *Optional.* String representation of the method return value signature. |
| `ObjectTypes` | `PSafeArray` | *Optional.* Array of `WideString` values specifying the types of objects that the method works with (e.g., `Instrument`, `Client`, etc.). |

*Return value:* [`GlueMethod`](#types-gluemethod)

An object describing the registered Interop method. The return value can be used to unregister the method by using its [`UnregisterMethod`](#interfaces-iglue42-unregistermethod) method.

#### RegisterStartupGlueWindow

Initiates the registration of a window as a Glue42 Window and sets the related event handler. Uses the default startup options, if any.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | `Integer` | Handle of the window to register. |
| `windowEventHandler` | `IGlueWindowEventHandler` | An object of a class implementing the [`IGlueWindowEventHandler`](#interfaces-igluewindoweventhandler) interface. |

*Return value:* [`IGlueWindow`](#interfaces-igluewindow)

An object representing the registered Glue42 Window.

*Note that the returned value shouldn't be used to interact with the Glue42 Window until the [`HandleWindowReady`](#interfaces-igluewindoweventhandler-handlewindowready) callback method is invoked (i.e. the registration is complete).*

#### RegisterStartupGlueWindowWithSettings

Initiates the registration of a window as a Glue42 Window with specific settings and sets the related event handler. Uses the default startup options, if any.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | Integer | Handle of the window to register. |
| `settings` | [`IGlueWindowSettings`](#interfaces-igluewindowsettings) | An object of `IGlueWindowSettings` containing the settings to use during the window registration. |
| `windowEventHandler` | `IGlueWindowEventHandler` | An object of a class implementing the [`IGlueWindowEventHandler`](#interfaces-igluewindoweventhandler) interface. |

*Return value:* [`IGlueWindow`](#interfaces-igluewindow)

*Note that the returned value shouldn't be used to interact with the Glue42 Window until the [`HandleWindowReady`](#interfaces-igluewindoweventhandler-handlewindowready) callback method is invoked (i.e. the registration is complete).*

#### RegisterStream

Registers an Interop stream (streaming method) and sets the related subscription handler.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `streamName` | `WideString` | Name of the Interop stream to register. |
| `subscriptionHandler` | [`IGlueSubscriptionHandler`](#interfaces-igluesubscriptionhandler) | An object of a class implementing the `IGlueSubscriptionHandler` interface. This is used to handle incoming subscription requests. |
| `Input` | `WideString` | *Optional.* String representation of the method input arguments signature. |
| `Output` | `WideString` | *Optional.* String representation of the method return value signature. |
| `ObjectTypes` | `PSafeArray` | *Optional.* Array of `WideString` values specifying the types of objects that the method works with (e.g., `Instrument`, `Client`, etc.). |
| `out stream` | [`IGlueStream`](#interfaces-igluestream) | Output parameter. An object representing the newly created stream. |

*Return value:* [`GlueMethod`](#types-gluemethod)

An object describing the registered Interop streaming method. The return value can be used to unregister the streaming method by using its [UnregisterMethod](#interfaces-iglue42-unregistermethod) method.

#### SetChannelData

Sets the value of a field in an existing Glue42 Channel.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `channel` | `WideString` | The Channel name. |
| `fieldPath` | `WideString` | Path to the field in JavaScript notation (e.g., `'data.object1.object2.field1'`). |
| `data` | `WideString` | A string representing the value to set. |

*Return value:* None

*Note that:*
- *If any of the intermediate objects specified in `fieldPath` (e.g., `object1`) or the field itself (`field1`) don't exist in the JSON tree, they will be automatically created.*
- *If any of the intermediate objects specified in `fieldPath` already exist in the JSON tree but aren't objects, the call will fail.*
- *If the field specified in `fieldPath` (e.g., `field1`) already exists in the JSON tree, its value will be replaced. This will work also if the existing field is an object.*

#### SetLogConfigurationPath

Overrides the Glue42 logging configuration path. This method must be invoked before invoking the [Start](#interfaces-iglue42-start) method, otherwise it will have no effect.

*See also:*

- [IGlue42](#interfaces-iglue42) - the [OverrideConfiguration](#interfaces-iglue42-overrideconfiguration) method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `logConfigPath` | `WideString` | Path to the Glue logging configuration. |

*Return value:* None

#### Start

Connects to Glue42 and announces the application instance. The connection to Glue42 is necessary for using any Glue42 functionality.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Instance` | [`GlueInstance`](#types-glueinstance) | Describes the identity to use when announcing the Glue42 application instance. |

*Return value:* None

#### StartWithAppName

Connects to Glue42 and announces the application instance. The connection to Glue42 is necessary for using any Glue42 functionality.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `ApplicationName` | `WideString` | The application name to use when announcing the Glue42 application instance. |

*Return value:* None

*Note that invoking this method is equivalent to invoking the [`Start`](#interfaces-iglue42-start) method with only the `ApplicationName` property of the `Instance` parameter initialized.*

#### Stop

Disconnects from Glue42, shutting down all communication.

*Parameters:* None

*Return value:* None

#### Subscribe

Subscribes an object of a class implementing the [`IGlueEvents`](#interfaces-iglueevents) interface for receiving various notifications from Glue42.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `handler` | [`IGlueEvents`](#interfaces-iglueevents) | An object of a class implementing the `IGlueEvents` interface. |

*Return value:* None

#### SubscribeGlueContext

Subscribes to a Glue42 shared context.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `contextName` | `WideString` | Name of the shared context. If the context doesn't exist, it will be automatically created. |
| `handler` | [`IGlueContextHandler`](#interfaces-igluecontexthandler) | An object of a class implementing the `IGlueContextHandler` interface. |

*Return value:* None

#### SubscribeStream

Subscribes to an Interop stream published by a Glue42 application instance and sets the related stream event handler.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | [`GlueMethod`](#types-gluemethod) | Description of the Interop streaming method to which to subscribe. The `Name` and `Instance.InstanceId` properties must be initialized respectively with the name of the Interop stream and the instance ID of the Glue42 application publishing the stream. |
| `subscriptionRequestArgs` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the arguments to pass with the subscription request. |
| `streamHandler` | [`IGlueStreamHandler`](#interfaces-igluestreamhandler) | An object of a class implementing the `IGlueStreamHandler` interface which will receive the stream event notifications. |
| `subscriptionTimeoutMsecs` | `Int64` | The subscription request will time out after the specified number of milliseconds. If the provided value is less than or equal to zero, then the default timeout value will be used. |

*Return value:* None

#### SubscribeStreams

Subscribes to an Interop stream published by one or more Glue42 applications and sets the related stream event handler.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `streamName` | `WideString` | Name of the Interop stream to subscribe to. |
| `subscriptionRequestArgs` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the arguments to pass with the subscription request. |
| `targets` | `PSafeArray` | *Optional.* Allows filtering the subscription targets. If provided, this must be an array of [`GlueInstance`](#types-glueinstance). |
| `all` | `WordBool` | Indicates whether the subscription request should be sent to all matching targets or only to the first one. |
| `identity` | [`GlueInstanceIdentity`](#enums-glueinstanceidentity) | Specifies the identity properties to be matched when applying the `targets` filter. |
| `streamHandler` | [`IGlueStreamHandler`](#interfaces-igluestreamhandler) | An object of a class implementing the `IGlueStreamHandler` interface which will receive the stream event notifications. |
| `subscriptionTimeoutMsecs` | `Int64` | The subscription request will time out after the specified number of milliseconds. If the provided value is less than or equal to zero, then the default timeout value will be used. |

*Return value:* None

#### SubscribeStreamsFilterTargets

Subscribes to an Interop stream published by one or more Glue42 applications and sets the related stream event handler. Subscription targets can be filtered by name.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `streamName` | `WideString` | Name of the Interop stream to subscribe to. |
| `subscriptionRequestArgs` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the arguments to pass with the subscription request. |
| `targetRegex` | `WideString` | *Optional.* Regular expression for filtering targets by application name. An empty string will match all application names. |
| `all` | `WordBool` | Indicates whether the subscription request should be sent to all matching targets or only to the first one. |
| `streamHandler` | [`IGlueStreamHandler`](#interfaces-igluestreamhandler) | An object of a class implementing the `IGlueStreamHandler` interface which will receive the stream event notifications. |
| `subscriptionTimeoutMsecs` | `Int64` | The subscription request will time out after the specified number of milliseconds. If the provided value is less than or equal to zero, then the default timeout value will be used. |

*Return value:* None

#### UnregisterMethod

Unregisters an Interop method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Method` | [`GlueMethod`](#types-gluemethod) | Description of the method to unregister. This must be the value returned by [`RegisterMethod`](#interfaces-iglue42-registermethod) or [`RegisterStream`](#interfaces-iglue42-registerstream). |

*Return value:* None

#### Unsubscribe

Cancels a subscription created with the [`Subscribe`](#interfaces-iglue42-subscribe) method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `handler` | [`IGlueEvents`](#interfaces-iglueevents) | An object of a class implementing the `IGlueEvents` interface. This must be the same object previously passed to the `Subscribe` method. |

*Return value:* None

#### Other Methods

The following methods of `IGlue42` exposed in the COM library aren't intended to be used in Delphi:

- `AddCorrelationInterest`
- `CloseResource`
- `CreateGlueData`
- `CreateGlueValues`
- `CreateMethodInvocator`
- `CreateServerMethod`
- `CreateServerStream`
- `CreateStreamConsumer`
- `G4O_XL_OpenSheet`
- `GetGlueContext`
- `InvokeAsync`
- `InvokeVariantData`
- `RegisterGlueWindowInSink`
- `RegisterMethodInSink`
- `RegisterSingleBranchStream`
- `RegisterStreamInSink`
- `RegisterVariantMethodInSink`
- `RegisterVoidMethodInSink`
- `SubscribeChannel`
- `SubscribeStreamInSink`
- `TranslateVbObject`
- `YieldCallbackData`
- `YieldCallbackVariantData`

### IGlueApp

Implementing this interface allows an object to be registered as a Glue42 application instance.

*See also:*

- [`IAppAnnouncer`](#interfaces-iappannouncer) - the [`RegisterAppInstance`](#interfaces-iappannouncer-registerappinstance) method.
- [`IAppFactoryRegistry`](#interfaces-iappfactoryregistry) - the [`RegisterAppInstance`](#interfaces-iappfactoryregistry-registerappinstance) method.

**Methods**

#### Initialize

Initializes a child application.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `state` | [`GlueValue`](#types-gluevalue) | A previously saved application state. |
| `glueWindow` | [`IGlueWindow`](#interfaces-igluewindow) | The registered Glue42 Window for the application. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### SaveState

Saves the application state.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `receiver` | [`IGlueValueReceiver`](#interfaces-igluevaluereceiver) | An instance of [`IGlueValueReceiver`](#interfaces-igluevaluereceiver) which can be used to set the the application state to be saved. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### Shutdown

Shuts down the application when Glue42 shuts down.

*Parameters:* None

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueContext

This interface is implemented by Glue42 shared context objects.

**Methods**

#### BuildAndSetContextData

Sets new (replaces) shared context data using a callback method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `builderCallback` | [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) | An object of a class implementing the `IGlueContextBuilderCallback` interface. |

*Return value:* None

#### BuildAndUpdateContextData

Updates the shared context data using a callback method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `builderCallback` | [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) | An object of a class implementing the `IGlueContextBuilderCallback` interface. |

*Return value:* None

#### Close

Unsubscribes from the associated Glue42 shared context.

*Parameters:* None

*Return value:* None

#### GetContextInfo

Gets information about the shared context.

*Parameters:* None

*Return value:* [`GlueContext`](#types-gluecontext)

#### GetData

Gets the shared context data.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`GlueContextValue`](#types-gluecontextvalue) values. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetDataAsJson

Retrieves data from the shared context by field path and returns it as JSON encoded data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `fieldPath` | `WideString` | Dot-separated string that represents a path in the shared context tree from where the JSON encoded data is to be read - e.g., `"data.contact"`. |

*Return value:* `WideString`

JSON encoded data that represents the shared context data - e.g., `"{parent: {child: {age: 5, name:\"Jay\"}}}"`.

#### Remove

Removes an elementary or composite value from the shared context.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `fieldPath` | `WideString` | Dot-separated string that represents a path in the shared context to the value to be removed, e.g. `"data.contact"`. |

*Return value:* None

#### SetContextData

Sets new (replaces) shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the new context data. |

*Return value:* None

#### SetContextDataOnFieldPath

Sets an array of context values under a context field specified by its path. If parts of the values in the path are missing, empty values will be created for them.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `fieldPath` | `WideString` | Dot-separated string that represents a path in the shared context tree where the JSON encoded data is to be placed - e.g., `"data.contact"`. |
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the new context data. |

*Return value:* None

#### UpdateContextData

Updates the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the new context data. |

*Return value:* None

#### UpdateContextDataJson

Updates the shared context data by field path with JSON encoded data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `fieldPath` | `WideString` | Dot-separated string that represents a path in the shared context tree where the JSON encoded data is to be placed - e.g., `"data.contact"`. |
| `jsonEncodedData` | `WideString` | Valid JSON encoded data - e.g., `"{parent: {child: {age: 5, name:\"Jay\"}}}"` |

*Return value:* None

#### Other Methods

The following methods of `IGlueContext` exposed in the COM library aren't intended to be used in Delphi:

- `GetReflectData`
- `Open`
- `SetValue`

### IGlueContextBuilder

An instance of this is passed as a parameter to the implementation of the [`Build`](#interfaces-igluecontextbuildercallback-build) method of the [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) interface.

**Methods**

#### AddBool

Adds a Boolean value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `WordBool` | The value to add. |

*Return value:* None

#### AddBoolArray

Adds an array of Boolean values to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `PSafeArray` | An array of `WordBool` values. |

*Return value:* None

#### AddComposite

Adds a composite value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `composite` | `PSafeArray` | The composite value as a `PSafeArray`. |
| `IsArray` | `WordBool` | Indicates whether the composite value is an array. |

*Return value:* None

#### AddContextValue

Adds a value represented as a [`GlueContextValue`](#types-gluecontextvalue) to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `GlueContextValue` | [`GlueContextValue`](#types-gluecontextvalue) | The [`GlueContextValue`](#types-gluecontextvalue) to add. |

*Return value:* None

#### AddDatetime

Adds a datetime value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `Int64` | An integer representation of Glue42 time. This is defined as the number of milliseconds since the Unix epoch, i.e. `1970-01-01 00:00:00 UTC`. |

*Return value:* None

#### AddDatetimeArray

Adds an array of datetime values to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `PSafeArray` | An array of `Int64` values representing Glue42 time. |

*Return value:* None

#### AddDouble

Adds a double-precision floating-point value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `Double` | The value to add. |

*Return value:* None

#### AddDoubleArray

Adds an array of double-precision floating-point values to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `PSafeArray` | An array of `Double` values. |

*Return value:* None

#### AddGlueValue

Adds a value represented as a [`GlueValue`](#types-gluevalue) to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | [`GlueValue`](#types-gluevalue) | The value to add. |

*Return value:* None

#### AddInt

Adds an integer value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `Integer` | The value to add. |

*Return value:* None

#### AddIntArray

Adds an array of integer values to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `PSafeArray` | An array of `Integer` values. |

*Return value:* None

#### AddLong

Adds a long integer value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `Int64` | The value to add. |

*Return value:* None

#### AddLongArray

Adds an array of long integer values to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `PSafeArray` | An array of `Int64` values. |

*Return value:* None

#### AddString

Add a string value to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `WideString` | The value to add. |

*Return value:* None

#### AddStringArray

Adds an array of string values to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | `PSafeArray` | An array of `WideString` values. |

*Return value:* None

#### AddTuple

Adds a tuple of values to the context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Tuple` | `PSafeArray` | Array of [`GlueValue`](#types-gluevalue) values. |

*Return value:* None

#### AddTupleValue

Adds a tuple of values represented as a [`GlueValue`](#types-gluevalue) to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `Value` | [`GlueValue`](#types-gluevalue) | The value representing the tuple. |

*Return value:* None

#### BuildComposite

Builds a value using a callback method and adds it to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `callback` | [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) | An object of a class implementing the `IGlueContextBuilderCallback` interface. |
| `IsArray` | `WordBool` | Indicates whether the value is an array. |

*Return value:* None

#### BuildTuple

Builds a tuple value using a callback method and adds it to the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Name` | `WideString` | Name associated with the shared context value. |
| `callback` | [`IGlueContextBuilderCallback`](#interfaces-igluecontextbuildercallback) | An object of a class implementing the `IGlueContextBuilderCallback` interface. |

*Return value:* None

#### Clear

Clears all shared context data.

*Parameters:* None

*Return value:* None

### IGlueContextBuilderCallback

An instance of this is passed to the implementations of the methods which use callbacks for building shared context data values (e.g., [`BuildAndSetContextData`](#interfaces-igluecontext-buildandsetcontextdata) or [`BuildComposite`](#interfaces-igluecontextbuilder-buildcomposite)).

**Methods**

#### Build

Builds the shared context data.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `builder` | [`IGlueContextBuilder`](#interfaces-igluecontextbuilder) | An instance of `IGlueContextBuilder` which can be used to build the shared context data. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueContextHandler

Implementing this interface allows an object to receive notifications about changes in a shared context.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`SubscribeGlueContext`](#interfaces-iglue42-subscribegluecontext) method.

**Methods**

#### HandleContext

Handles the shared context when a subscription is activated via [`SubscribeGlueContext`](#interfaces-iglue42-subscribegluecontext).

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | [`IGlueContext`](#interfaces-igluecontext) | Use to set or update the shared context data. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleContextUpdate

Handles updates of the associated shared context.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `contextUpdate` | [`IGlueContextUpdate`](#interfaces-igluecontextupdate) | This object can be used to obtain information about the update of the shared context. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueContextUpdate

Describes an update of a shared context.

**Methods**

#### GetAdded

Gets an array of the values which have been added to the shared context.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`GlueContextValue`](#types-gluecontextvalue) values. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetContext

Gets the shared context which has been updated.

*Parameters:* None

*Return value:* [`IGlueContext`](#interfaces-igluecontext)

#### GetRemoved

Gets an array of the names of the properties which have been removed from the shared context.

*Parameters:* None

*Return value:* `PSafeArray`

An array of `WideString` values. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### GetUpdated

Gets an array of the values which have been updated in the shared context.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`GlueContextValue`](#types-gluecontextvalue) values. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

### IGlueEvents

Implementing this interface allows an object to receive notifications about various Glue42 events.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`Subscribe`](#interfaces-iglue42-subscribe) method.

**Methods**

#### HandleConnectionStatus

Handles the event when the Glue42 connection status changes.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `state` | [`GlueState`](#enums-gluestate) | The new Glue42 connection state. |
| `Message` | `WideString` | Message text related to the connection status change. |
| `date` | `Int64` | Glue42 time of the connection status change. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleGlueContext

Handles the event when a new shared context is created.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | [`GlueContext`](#types-gluecontext) | Information about the shared context associated with the event. |
| `created` | `WordBool` | Indicates whether the shared context is newly created. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleInstanceStatus

Handles the event when a Glue42 application instance appears or disappears.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Instance` | [`GlueInstance`](#types-glueinstance) | Information about the Glue42 application instance associated with the event. |
| `active` | `WordBool` | Indicates whether the application instance is now active. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleMethodStatus

Handles the event when an Interop method is registered or unregistered by an application instance.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Method` | [`GlueMethod`](#types-gluemethod) | Information about the Interop method associated with the event. |
| `active` | `WordBool` | Indicates whether the Interop method is now active. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleException

Handles the event when an exception is thrown during the operation of Glue42.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Message` | `WideString` | Message associated with the exception. |
| `ex` | [`GlueValue`](#types-gluevalue) | A [`GlueValue`](#types-gluevalue) containing additional information about the exception. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueInvocationResultHandler

Implementing this interface allows an object to handle Interop method invocation results.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`BuildAndInvoke`](#interfaces-iglue42-buildandinvoke) method.
- [`IGlue42`](#interfaces-iglue42) - the [`InvokeMethod`](#interfaces-iglue42-invokemethod) method.
- [`IGlue42`](#interfaces-iglue42) - the [`InvokeMethods`](#interfaces-iglue42-invokemethods) method.

**Methods**

#### HandleResult

Handles the result from invoking an Interop method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `invocationResult` | `PSafeArray` | An array of [`GlueInvocationResult`](#types-glueinvocationresult) values. |
| `correlationId` | `WideString` | Correlation ID of the Interop method that was invoked. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueRequestHandler

Implementing this interface allows an object to handle Interop method invocation requests.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`RegisterMethod`](#interfaces-iglue42-registermethod) method.

**Methods**

#### HandleInvocationRequest

Handles the invocation of a registered Interop method.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Method` | [`GlueMethod`](#types-gluemethod) | Information about the Interop method that is invoked. |
| `caller` | [`GlueInstance`](#types-glueinstance) | Information about the Glue42 application instance that has invoked the method. |
| `requestValues` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the method invocation arguments. |
| `resultCallback` | [`IGlueServerMethodResultCallback`](#interfaces-iglueservermethodresultcallback) | An object of a class implementing the `IGlueServerMethodResultCallback` interface. Can be used to send the result of the method invocation to the caller application instance. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueServerMethodResultCallback

An instance of this is passed to the implementation of the [`HandleInvocationRequest`](#interfaces-igluerequesthandler-handleinvocationrequest) method of the  [`IGlueRequestHandler`](#interfaces-igluerequesthandler) interface.

**Methods**

#### SendResult

Sends the result from invoking an Interop method to the caller application instance.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Result` | [`GlueResult`](#types-glueresult) | The result to send back to the caller. |

*Return value:* None

### IGlueServerSubscriptionCallback

An instance of this is passed to the implementation of the [`HandleSubscriptionRequest`](#interfaces-igluesubscriptionhandler-handlesubscriptionrequest) method of the [`IGlueSubscriptionHandler`](#interfaces-igluesubscriptionhandler) interface.

**Methods**

#### Accept

Accepts an Interop stream subscription request.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `branch` | `WideString` | *Optional.* Name of the stream branch on which the subscriber will be accepted. If the specified branch doesn't exist, it will be created. |
| `Result` | [`GlueResult`](#types-glueresult) | Subscription result information to be sent back to the subscriber. |

*Return value:* [`IGlueStreamBranch`](#interfaces-igluestreambranch)

An instance of `IGlueStreamBranch` representing the branch on which the subscriber was accepted. If the `branch` parameter is empty, the default (unnamed) branch for the stream will be returned.

#### Reject

Rejects an Interop stream subscription request.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `Result` | [`GlueResult`](#types-glueresult) | Rejection information to be sent back to the application requesting the subscription. |

*Return value:* None

### IGlueStream

Represents a streaming Interop method.

**Methods**

#### CloseBranch

Closes a stream branch, disconnecting all subscribers to the branch and removing the branch from the stream.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `branch` | `WideString` | Name of the branch to close. |

*Return value:* None

#### CloseStream

Closes a stream, disconnecting all subscribers to the stream on all branches. All branches will be removed from the stream.

*Parameters:* None

*Return value:* None

#### GetBranch

Gets an existing stream branch.

| Name | Type | Description |
|------|------|-------------|
| `branchKey` | `WideString` | The name of the branch to get. |

*Return value:* [`IGlueStreamBranch`](#interfaces-igluestreambranch)

Can be used to perform stream actions specific to the branch.

#### GetBranchKeys

Get the names of the available stream branches.

*Parameters:* None

*Return value:* `PSafeArray`

An array of `WideString` values with the names of the available branches. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### Push

Pushes data to the stream.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the data to be pushed to the stream. |
| `branch` | `WideString` | *Optional.* Branch name to which to push the data. The branch must have been created previously. If not specified, the data will be broadcast to all subscribers. |

*Return value:* None

### IGlueStreamBranch

Represents a branch of an Interop stream.

**Methods**

#### Close

Closes a stream branch, disconnecting all subscribers to the branch and removing the branch from the stream.

*Parameters:* None

*Return value:* None

#### GetKey

Gets the name of the stream branch.

*Parameters:* None

*Return value:* `WideString`

A string containing the name of the stream branch.

#### GetStream

Gets the Interop stream to which the branch belongs.

*Parameters:* None

*Return value:* [`IGlueStream`](#interfaces-igluestream)

An instance representing the Interop stream to which the branch belongs.

#### GetSubscribers

Gets all subscribers to the stream branch.

*Parameters:* None

*Return value:* `PSafeArray`

An array of [`IGlueStreamSubscriber`](#interfaces-igluestreamsubscriber) instances representing all current subscribers to the stream branch. The returned value must be destroyed with `SafeArrayDestroy` when no longer needed.

#### Push

Pushes data to the stream branch.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the data to be pushed to the stream branch. |

*Return value:* None

### IGlueStreamHandler

Implementing this interface allows an object to receive notifications about Interop stream events.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`SubscribeStream`](#interfaces-iglue42-subscribestream) method.
- [`IGlue42`](#interfaces-iglue42) - the [`SubscribeStreams`](#interfaces-iglue42-subscribestreams) method.

**Methods**

#### HandleStreamData

Handles the data that has been pushed by an Interop stream to the subscriber.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | [`GlueMethod`](#types-gluemethod) | Information about the Interop streaming method that pushed the data. |
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the data pushed by the streaming method. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleStreamStatus

Handles changes of the state of the Interop stream subscription (e.g., when the subscription is accepted, rejected, closed).

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | [`GlueMethod`](#types-gluemethod) | Information about the Interop streaming method related to the subscription. |
| `state` | [`GlueStreamState`](#enums-gluestreamstate) | The new state of the stream subscription. |
| `Message` | `WideString` | Message related to the subscription status change. |
| `dateTime` | `Int64` | Glue42 time of the subscription state change. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### StreamClosed

Handles the event when the Interop stream associated with the subscription has been closed.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
|`stream`|[GlueMethod](#types-gluemethod)|Information about the Glue streaming method related to the subscription.|

*Return value:* `HResult`
*Note that the implementation must return `S_OK`.*

#### SubscriptionActivated

Handles the event when the subscription request has been dispatched to Glue42.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `GlueStreamSubscription` | [`IGlueStreamSubscription`](#interfaces-igluestreamsubscription) | An instance of the stream subscription that can later be used to close the subscription from the client side. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueStreamSubscriber

Describes an Interop stream subscriber.

**Methods**

#### Close

Disconnects the subscriber from the Interop stream.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values. This parameter is currently ignored. |

*Return value:* None

#### GetSubscriberInstance

Gets the Glue42 application instance subscribed to the stream.

*Parameters:* None

*Return value:* [`GlueInstance`](#types-glueinstance)

Description of the Glue42 application instance subscribed to the stream.

#### Push

Pushes data to the subscriber.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `data` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the data to be pushed to the subscriber. |

*Return value:* None

### IGlueStreamSubscription

Describes an Interop stream subscription.

**Methods**

#### Close

Disconnects the subscriber from the stream.

*Parameters:* None

*Return value:* None

### IGlueSubscriptionHandler

Implementing this interface allows an object to handle stream subscription events for a registered Interop stream.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`RegisterStream`](#interfaces-iglue42-registerstream) method.

**Methods**

#### HandleSubscriber

Handles the event when a new subscriber to the stream has been accepted.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `subscriberInstance` | [`GlueInstance`](#types-glueinstance) | Description of the Glue42 application instance which has subscribed to the stream. |
| `glueStreamSubscriber` | [`IGlueStreamSubscriber`](#interfaces-igluestreamsubscriber) | An instance of `IGlueStreamSubscriber` representing the new subscriber. |
| `requestValues` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the arguments passed with the subscription request. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleSubscriberLost

Handles the event when a subscriber has unsubscribed from the stream. This also includes the cases where subscribers have been forcefully unsubscribed when the `Close` method has been called for a subscriber, stream or stream branch.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `subscriberInstance` | [`GlueInstance`](#types-glueinstance) | Description of the Glue42 application instance which is no longer subscribed to the stream. |
| `glueStreamSubscriber` | [`IGlueStreamSubscriber`](#interfaces-igluestreamsubscriber) | An instance of `IGlueStreamSubscriber` representing the lost subscriber. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleSubscriptionRequest

Handles the event when a Glue42 application instance requests to subscribe to the stream.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | [`GlueMethod`](#types-gluemethod) | Description of the Interop stream for which the subscription is requested. |
| `caller` | [`GlueInstance`](#types-glueinstance) | Description of the Glue42 application instance requesting to subscribe to the stream. |
| `requestValues` | `PSafeArray` | An array of [`GlueContextValue`](#types-gluecontextvalue) values representing the arguments passed with the subscription request. |
| `callback` | [`IGlueServerSubscriptionCallback`](#interfaces-iglueserversubscriptioncallback) | An instance of `IGlueServerSubscriptionCallback` which can be used to accept or reject the subscription request. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueValueReceiver

An instance of this is passed to the implementation of the [`SaveState`](#interfaces-iglueapp-savestate) method of the [`IGlueApp`](#interfaces-iglueapp) interface.

**Methods**

#### SendGlueValue

Sends a [`GlueValue`](#types-gluevalue) to be set as an application state.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `glueValue` | [`GlueValue`](#types-gluevalue) | Value to set as an application state. |

*Return value:* None

### IGlueWindow

Describes a Glue42 Window.

**Methods**

#### Activate

Activates the window.

*Parameters:* None

*Return value:* None

#### GetChannelContext

Gets the current Channel context for the window.

*Parameters:* None

*Return value:* [`IGlueContext`](#interfaces-igluecontext)

If the window is not currently bound to a Channel, returns `Null`.

#### GetChannelSupport

Determines whether the Channel Selector box is visible in the Glue42 Window title bar.

*Parameters:* None

*Return value:* `WordBool`

#### GetId

Gets the Glue42 Window identifier as a string.

*Parameters:* None

*Return value:* `WideString`

#### GetTitle

Gets the Glue42 Window title.

*Parameters:* None

*Return value:* `WideString`

#### IsVisible

Determine whether the Glue42 Window is visible.

*Parameters:* None

*Return value:* `WordBool`

#### SetChannelSupport

Hides or shows the Channel Selector box in the Glue42 Window title bar.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `showLink` | `WordBool` | Indicates whether the Channel Selector box is to be visible or not. Note that hiding or showing the Channel Selector box doesn't enable or disable Channel support for the window - this can only be done during the window registration. |

*Return value:* None

#### SetTitle

Sets the Glue42 Window title.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `title` | `WideString` | Text to set in the window title bar. |

*Return value:* None

#### SetVisible

Makes the Glue42 Window visible or invisible.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `visible` | `WordBool` | Indicates whether the window is to be visible or not. |

*Return value:* None

#### Unregister

Unregisters the Glue42 Window.

*Parameters:* None

*Return value:* None

### IGlueWindowEventHandler

Implementing this interface allows an object to receive Channel change or update notifications for a registered Glue42 Window.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`RegisterGlueWindow`](#interfaces-iglue42-registergluewindow) method.
- [`IGlue42`](#interfaces-iglue42) - the [`RegisterGlueWindowWithSettings`](#interfaces-iglue42-registergluewindowwithsettings) method.
- [`IGlue42`](#interfaces-iglue42) - the [`RegisterStartupGlueWindow`](#interfaces-iglue42-registerstartupgluewindow) method.
- [`IGlue42`](#interfaces-iglue42) - the [`RegisterStartupGlueWindowWithSettings`](#interfaces-iglue42-registerstartupgluewindowwithsettings) method.

**Methods**

#### HandleWindowReady

Handles the event when the Glue42 Window registration has completed.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `glueWindow` | [`IGlueWindow`](#interfaces-igluewindow) | The registered Glue42 Window. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleChannelChanged

Handles the event when the user changes the Channel via the Channel Selector box in the window title bar.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `glueWindow` | [`IGlueWindow`](#interfaces-igluewindow) | The registered Glue42 Window for the application. |
| `channel` | [`IGlueContext`](#interfaces-igluecontext) | The context object of the newly selected Channel. |
| `prevChannel` | [`GlueContext`](#types-gluecontext) | Information about the previously selected Channel. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleChannelData

Handles the event when the data in the currently selected Channel has changed or when the user selects a new Channel via the Channel Selector box in the window title bar.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `glueWindow` | [`IGlueWindow`](#interfaces-igluewindow) | The registered Glue42 Window for the application. |
| `channelUpdate` | [`IGlueContextUpdate`](#interfaces-igluecontextupdate) | This object can be used to obtain information about the update of the Channel context. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleWindowEvent

Handles additional window events.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `glueWindow` | [`IGlueWindow`](#interfaces-igluewindow) | The registered Glue42 Window for the application. |
| `eventType` | [`GlueWindowEventType`](#enums-gluewindoweventtype) | The type of the window event. |
| `eventData` | [`GlueValue`](#types-gluevalue) | Additional information about the window event. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

#### HandleWindowDestroyed

Handles the event when the Glue42 Window is destroyed.

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `glueWindow` | [`IGlueWindow`](#interfaces-igluewindow) | The registered Glue42 Window for the application. |

*Return value:* `HResult`

*Note that the implementation must return `S_OK`.*

### IGlueWindowSettings

Contains properties representing the settings to use during a Glue42 Window registration.

*See also:*

- [`IGlue42`](#interfaces-iglue42) - the [`RegisterGlueWindowWithSettings`](#interfaces-iglue42-registergluewindowwithsettings) method.

**Properties**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `AllowTabClose` | `WordBool` | `True` | Specifies whether a tab window will have a "Close" button. Only applicable when the window `Type` is set to `"Tab"`. |
| `AllowUnstick` | `WordBool` | `True` | Specifies whether the Glue42 Window can be separated from other Glue42 Windows once it has been stuck to another Glue42 Window or window group. (Currently unusable.) |
| `Channel` | `WideString` | `''` | Specifies the Channel to be initially selected. No Channel is selected by default. |
| `ChannelSupport` | `WordBool` | `False` | Specifies whether the window will have Channel support enabled. |
| `FrameColor` | `WideString` | `''` | The color of the window frame. (Currently unusable.) |
| `Icon` | `WideString` | `''` | Icon for the application. (Currently unusable.) |
| `IsSticky` | `WordBool` | `True` | Specifies whether the window can be stuck to other Glue42 Windows or window groups. |
| `MaxHeight` | `Integer` | `0` | Specifies the maximum height (in pixels) to which the window can be resized. A value of `0` means no limit. The height of the window title bar isn't included. |
| `Maximizable` | `Integer` | `True` | Specifies whether the window will have a "Maximize" button. |
| `MaxWidth` | `Integer` | `0` | Specifies the maximum width (in pixels) to which the window can be resized. A value of `0` means no limit. |
| `MinHeight` | `Integer` | `0` | Specifies the minimum height (in pixels) to which the window can be resized. The height of the window title bar isn't included. |
| `Minimizable` | `WordBool` | `True` | Specifies whether the window will have a "Minimize" button. |
| `MinWidth` | `Integer` | `0` | Specifies the minimum width (in pixels) to which the window can be resized. |
| `ShowTaskbarIcon` | `WordBool` | `True` | Specifies whether the window should have an icon in the taskbar. |
| `StandardButtons` | `WideString` | `''` | Optional list of comma-separated values specifying which command buttons will be available in the window title bar. See [Standard Buttons](#interfaces-igluewindowsettings-standard_buttons) below. |
| `SynchronousDestroy` | `WordBool` | `False` | Delphi applications must always set this value to `True`. |
| `Title` | `WideString` | `''` | Specifies the window title. If an empty string is provided, the initial window title will be set to the name of the Glue42 application registering the window. |
| `Type` | `WideString` | `'Flat'` | Specifies the window type. The available types are `"Flat"` and  `"Tab"`. |

#### StandardButtons

The `StandardButtons` property specifies which command buttons will be available in the window title bar. A comma-separated list of the following values may be provided:

| Value | Description |
|-------|-------------|
| `'None'` | No command buttons will be shown. This is used to hide all command buttons and can't be combined with other values. |
| `'LockUnlock'` | Show the "Lock/Unlock" button. |
| `'Extract'` | Show the "Extract" button. |
| `'Collapse'` | Show the "Collapse" button. |
| `'Minimize'` | Show the "Minimize" button. Works in conjunction with the `Minimizable` property which must also be set to `True` for the button to be visible. |
| `'Maximize'` | Show the "Maximize". Works in conjunction with the `Maximizable` property which must also be set to `True` for the button to be visible. |
| `'Close'` | Show the "Close" button. |
| `'Default'` \| `''` | Equivalent to `'Collapse,Minimize,Maximize,Close'`. |