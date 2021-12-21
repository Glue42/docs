## Using Shared Contexts

In order to use shared contexts, you must implement the [`IGlueContextHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluecontexthandler) interface:

```delphi
TMainForm = class(TForm, IGlueContextHandler)
...
private
  G42: IGlue42;
  // The shared context handle.
  myContext: IGlueContext;
protected
  function HandleContext(const context: IGlueContext): HResult; stdcall;
  function HandleContextUpdate(const contextUpdate: IGlueContextUpdate): HResult; stdcall;
  ...
```

## Subscribing for Context Updates

To subscribe for context updates, use the [`SubscribeGlueContext`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-subscribegluecontext) method. If the context doesn't exist, a new empty context will be created:

```delphi
TMainForm = class(TForm, IGlueContextHandler)
  ...
  procedure TMainForm.InitializeGlue;
    ...
    G42.Start(inst);
    // Subscribe for context updates.
    G42.SubscribeGlueContext('myContextName', Self);
    ...
```

## Handling Context Updates

To handle a newly activated shared context subscription, use the [`HandleContext`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluecontexthandler-handlecontext) method:

```delphi
function TMainForm.HandleContext(const context: IGlueContext): HResult; stdcall;
begin
  // Store the context handle.
  if context.GetContextInfo.Name = 'myContextName' then
    myContext := context;

  Result := S_OK;
end;
```

To handle updates of the shared context data, use the [`HandleContextUpdate`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluecontexthandler-handlecontextupdate) method:

```delphi
function TMainForm.HandleContextUpdate(const contextUpdate: IGlueContextUpdate): HResult; stdcall;
var
  context: IGlueContext;
  contextName: WideString;
  data: TGlueContextValueArray;
begin
  context := contextUpdate.GetContext();
  contextName := context.GetContextInfo.Name;
  // Get the context data in a native data structure.
  rawData := context.GetData();
  data := SA_AsTranslatedContextValues(rawData);
  ...
  SafeArrayDestroy(rawData);
  Result := S_OK;
end;
```

## Updating a Context

To update the data of a shared context, use the [`SetContextData`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluecontext-setcontextdata) method of the [`IGlueContext`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluecontext) interface. The data must be provided as a `PSafeArray` containing [`GlueContextValue`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#types-gluecontextvalue) values:

```delphi
var
  psaData: PSafeArray;
  nativeData: array[0..0] of TGlueContextValue;
  emails: array of TGlueValue;
begin
  // Initiate an array with data.
  SetLength(emails,2);
  emails[0] := CreateValue('vernon.mullen@acme.com');
  emails[1] := CreateValue('vernon.d.mullen@acme.com');

  // Create the data with native types.
  nativeData[0] := CreateContextValue('contact', CreateComposite([
    CreateContextValue('name', CreateComposite([
      CreateContextValue('firstName', CreateValue('Vernon')),
      CreateContextValue('lastName', CreateValue('Mullen'))
    ], false)),
    CreateContextValue('displayName', CreateValue('Vernon Mullen')),
    CreateContextValue('emails', CreateTuple(emails))
  ], false));
  
  // Pack the native data into a `PSafeArray`.
  psaData := CreateContextValues_SA(AsGlueContextValueArray(nativeData));

  // Update the context data.
  context01.SetContextData(psaData);

  // Clean up.
  SafeArrayDestroy(psaData);
end;
```

The resulting context data from the previous example, represented as JSON:

```json
{
    "contact": {
        "name": {
            "firstName": "Vernon",
            "lastName": "Mullen"
        },
        "displayName": "Vernon Mullen",
        "emails": [
            "vernon.mullen@acme.com",
            "vernon.d.mullen@acme.com"
        ]
    }
}
```

## Listing All Available Contexts

To obtain a list of all available contexts, use the [`GetKnownContexts`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-getknowncontexts) method:

```delphi
var
  saContexts: PSafeArray;
  contexts: array of GlueContext;
begin
  saContexts := G42.GetKnownContexts();
  contexts := SA_AsGlueContextArray(saContexts);
  ...
  SafeArrayDestroy(saContexts);
end;
```