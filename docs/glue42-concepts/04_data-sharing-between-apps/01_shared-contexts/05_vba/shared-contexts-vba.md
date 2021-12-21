## Opening or Creating a Context

Define a [`GlueContextManager`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluecontextmanager) instance and set its value using the [`GetGlueContext`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-glue42-getgluecontext) method passing the name of a context as an argument. Use the [`Open`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluecontextmanager-open) method of the `GlueContextManager` instance to open the selected context. If a context with the specified name doesn't exist, it will be automatically created.

Below is an example of a subroutine opening a context:

```vbnet
Dim WithEvents MyContext As GlueContextManager

Private Sub OpenContext()
    On Error GoTo HandleErrors
    
    Set MyContext = Glue.GetGlueContext("MyContext")
    MyContext.Open
    Exit Sub

    HandleErrors:
    ' Handle exceptions.

End Sub
```

*Note that when you declare a `GlueContextManager` instance with `WithEvents`, your application is automatically subscribed for the events exposed by `GlueContextManager` and you must provide implementations for handling these events.*

## Subscribing for Context Updates

To subscribe for context updates, use the [`HandleContextUpdate`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluecontextmanager-handlecontextupdate) event exposed by the [`GlueContextManager`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluecontextmanager) instance. Its handler is executed when the context data has been updated.

Below is an example of a context update handler:

```vbnet
Private Sub MyContext_HandleContextUpdate(ByVal contextUpdate As IGlueContextUpdate)

    Dim context As GlueContextManager
    Set context = contextUpdate.GetContext

    Dim ContextName as String
    ContextName = context.GetContextInfo.Name

    Dim Data
    Set Data = context.GetReflectData("")
    ' Examine the updated data.
    ...
    
    On Error GoTo HandleErrors
    Exit Sub

    HandleErrors:
    ' Handle exceptions.
    
End Sub
```

## Updating a Context

Use the [`SetValue`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluecontextmanager-setvalue) method of the [`GlueContextManager`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluecontextmanager) instance to update the context data:

```vbnet
' Create a root composite value and add data in it.
Dim Data
Set Data = Glue.CreateGlueValues

Dim variantArray(0 To 1) As Variant
variantArray(0) = 1
variantArray(1) = "textValue"
Data("myTuple") = variantArray

' Updating a context.
MyContext.SetValue "data", Data
```

## Listing All Available Contexts

To get a list of all available contexts, use the [`GetKnownContexts`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-glue42-getknowncontexts) method:

```vbnet
Dim AllContexts() as GlueContext

AllContexts = Glue.GetKnownContexts()
``` 