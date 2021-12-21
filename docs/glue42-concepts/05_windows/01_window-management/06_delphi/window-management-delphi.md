## Glue42 Windows

In order for form windows of Delphi applications to become Glue42 Windows, they must be registered as Glue42 Windows after the Glue42 COM library has been initialized.

## Registering Delphi Forms

To enable registration as a Glue42 Window, the form class must implement the [`IGlueWindowEventHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler) interface:

```delphi
TMainForm = class(TForm, IGlueWindowEventHandler)
...
private
  G42: IGlue42;
  // The Glue42 Window handle.
  glueWin: IGlueWindow;
protected
  function HandleWindowReady(const glueWindow: IGlueWindow): HResult; stdcall;
  function HandleChannelData(const glueWindow: IGlueWindow; const channelUpdate: IGlueContextUpdate): HResult; stdcall;
  function HandleChannelChanged(const glueWindow: IGlueWindow; const channel: IGlueContext; prevChannel: GlueContext): HResult; stdcall;
  function HandleWindowDestroyed(const glueWindow: IGlueWindow) : HResult; stdcall;
  ...
```

You can register the form as a Glue42 Window using the [`RegisterGlueWindow`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglue42-registergluewindow) method immediately after initializing Glue42:

```delphi
procedure TMainForm.InitializeGlue;
  ...
    G42.Start(inst);
    // Register the Glue42 Window.
    G42.RegisterGlueWindow(Self.Handle, Self);
  ...
```

Unregister the Glue42 Window when the form is closed:

```delphi
procedure TMainForm.FormClose(Sender: TObject; var Action: TCloseAction);
begin
  if Assigned(glueWin) then
  begin
    glueWin.Unregister;
  end;
  ...
end;
```

## Window Events

You must provide implementations for the methods of the [`IGlueWindowEventHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler) interface. The following example demonstrates a minimal implementation of the event handlers:  

```delphi
//  Handles the event when the Glue42 Window registration has completed.
function TMainForm.HandleWindowReady(const glueWindow: IGlueWindow): HResult;
begin
  glueWin := glueWindow;
  Result := S_OK;
end;

// Handles the event when the user changes the Channel via the Channel Selector in the window title bar.
function TMainForm.HandleChannelChanged(const glueWindow: IGlueWindow; const channel: IGlueContext; prevChannel: GlueContext): HResult;
begin
  Result := S_OK;
end;

// Handles the event when the data in the currently selected Channel has changed.
function TMainForm.HandleChannelData(const glueWindow: IGlueWindow; const channelUpdate: IGlueContextUpdate): HResult;
begin
  Result := S_OK;
end;

// Handles the event when the Glue42 Window is destroyed.
function TMainForm.HandleWindowDestroyed(const glueWindow: IGlueWindow): HResult;
begin
  Result := S_OK;
end;
```

*For more information about using Glue42 Channels, see the [Channels](../../../data-sharing-between-apps/channels/delphi/index.html) section.*

## Window Operations

Once a Glue42 Window has been registered, you can use the [`HandleWindowReady`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler-handlewindowready) event to perform operations on it.

### Title

To get the current window title, use the [`GetTitle`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow-gettitle) method:

```delphi
var
  winTitle: WideString;
...
  winTitle := glueWin.GetTitle();
```

To change the window title, use the [`SetTitle`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow-settitle) method:

```delphi
glueWin.SetTitle('New Window Title');
```

### Visibility

To check whether the window is visible, use [`IsVisible`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow-isvisible). To hide or show a window, use [`SetVisible`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow-setvisible) and pass a `WordBool` value as an argument:  

```delphi
if glueWin.IsVisible() then
  glueWin.SetVisible(False)
else
  glueWin.SetVisible(True);
```