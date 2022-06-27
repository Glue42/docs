## Glue42 Windows

In order for windows of VBA apps to become Glue42 Windows, they must be registered as Glue42 Windows after the Glue42 COM library has been initialized.

## Registering VBA UserForms

Registering a VBA `UserForm` as a Glue42 Window will decorate it with a "sticky" frame allowing it to be visually integrated with other Glue42 enabled apps.

*Note that registering a VBA `UserForm` as a Glue42 Window imposes some restrictions on it (for more details, see [VBA UserForm Restrictions](#vba_userform_restrictions)). You can still use Glue42 functionality in the `UserForm` without registering it as a Glue42 Window.*

To register a VBA `UserForm` as a Glue42 Window with [default settings](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindowsettings), use the [`RegisterGlueWindow`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-glue42-registergluewindow) method:

```vbnet
Dim WithEvents GlueWin As GlueWindow

Private Sub RegisterGlueWindow()
    On Error GoTo HandleErrors

    If Not GlueWin Is Nothing Then
        ' The Glue42 Window has already been registered (or registration is still in progress).
        Exit Sub
    End If

    Set GlueWin = Glue.RegisterGlueWindow(GetFormHwnd(Me), Nothing)
    Exit Sub

    HandleErrors:
    ' Handle exceptions.

End Sub
```

*The example uses the [GetFormHwnd](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#glue42_vba_concepts-helper_functions-getformhwnd) helper function in order to retrieve the window handle (HWND) of the VBA `UserForm`.*

You can also initiate the window registration with custom settings by using [`RegisterGlueWindowWithSettings`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-glue42-registergluewindowwithsettings) instead:

```vbnet
' Create default window settings.
Dim WinSettings As GlueWindowSettings
Set WinSettings = Glue.CreateDefaultVBGlueWindowSettings

' Specify custom window settings.

' Must always be set to `True` in VBA.
WinSettings.SynchronousDestroy = True
' Disable Glue42 Channels.
WinSettings.ChannelSupport = False
' Set custom title.
WinSettings.Title = "Custom Title"

Set GlueWin = Glue.RegisterGlueWindowWithSettings(GetFormHwnd(Me), WinSettings, Nothing)
```

## Window Events

You must provide implementations for the events that will be raised as a result of the interaction with the registered Glue42 Window.

*The events [`HandleChannelChanged`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlechannelchanged) and [`HandleChannelData`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlechanneldata) are described in the [Channels](../../../data-sharing-between-apps/channels/vba/index.html) documentation.*

### Window Ready

The [`HandleWindowReady`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlewindowready) event of a [`GlueWindow`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow) instance is raised when the registration of the window has completed. You can use its handler to indicate that the registration has completed and is safe to perform other operations with the Glue42 Window instance (changing the title, visibility, etc.):

```vbnet
Dim FormRegistered As Boolean

Private Sub GlueWin_HandleWindowReady(ByVal window As IGlueWindow)
    ' Indicate that the Glue42 Window registration has completed.
    FormRegistered = True
    ' Perform additional Glue42 Window operations here.
End Sub
```

### Window Destroyed

The [`HandleWindowDestroyed`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlewindowdestroyed) event is raised when the Glue42 Window is being destroyed. The purpose for raising this event is to provide an opportunity for the VBA app to gracefully unload the VBA `UserForm` (see also [VBA UserForm Restrictions](#vba_userform_restrictions)):

```vbnet
Private Sub GlueWin_HandleWindowDestroyed(ByVal window As IGlueWindow)
    ' Unload the VBA `UserForm`.
    Unload Me
End Sub
```

### Additional Window Events

You may optionally implement a handler for [`HandleWindowEvent`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlewindowevent) which will be executed for various events related to the Glue42 Window, e.g. when the window is activated, moved, etc.

```vbnet
Private Sub GlueWin_HandleWindowEvent(ByVal window As IGlueWindow, ByVal eventType As GlueWindowEventType, ByVal eventData As GlueDynamicValue)
    If eventType = GlueWindowEventType_BoundsChanged Then
        ' Window was moved or resized, examine `eventData` for details.
        ...
    End If
End Sub
```

## Window Operations

Once the VBA window has been registered as a Glue42 Window, you can perform different operations on it.

### Title

To get the current window title, use the [`GetTitle`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-gettitle) method of a window instance:

```vbnet
Dim WinTitle as String

WinTitle = GlueWin.GetTitle()
```

To change the window title, use the [`SetTitle`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-settitle) method of a window instance:

```vbnet
GlueWin.SetTitle "New Title"
```

### Visibility

To check whether the window is visible, use [`IsVisible`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-isvisible). To hide or show a window, use [`SetVisible`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-setvisible) and pass a `Boolean` value as an argument:

```vbnet
If GlueWin.IsVisible() Then
    GlueWin.SetVisible False
Else
    GlueWin.SetVisible True
End If
```

### Activation

To activate the window, use the [`Activate`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-activate) subroutine:

```vbnet
GlueWin.Activate
```

## VBA UserForm Restrictions

The following restrictions apply to a VBA `UserForm` when it has been registered as a Glue42 Window:

- The app must provide a mandatory implementation of the [`HandleWindowDestroyed`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlewindowdestroyed) event in order to properly unload the VBA `UserForm`. Failing to unload the VBA `UserForm` will lead to deadlocks in the VBA execution thread.
- If implementing a handler for the `UserForm_QueryClose`, the app must not make any blocking calls (e.g., use I/O operations, display close confirmation popups to the user) or prevent the `UserForm` from unloading by setting a non-zero value to the `Cancel` parameter.
- The app shouldn't change directly the VBA `UserForm` visibility or position (e.g., with `Show`, `Hide`, `Move`).
- After a `UserForm` has been closed/unloaded, it can be displayed again by using `Show`. In this case you will need to repeat the Glue42 initialization and Glue42 Window registration for the `UserForm`.