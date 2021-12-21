## Overview 

The Glue42 Channels are enabled by default when registering a VBA window as a Glue42 Window. You have to provide handler implementations for the Channel related events that will be raised due to interaction with the Glue42 Window instance - [changing Channels](#handling_channel_change) and [handling Channel data updates](#subscribing_for_channel_updates). When the Glue42 Channels are enabled, a Channel Selector box is available in the window title bar.

To check whether the Channel Selector is visible, use the [`GetChannelSupport`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow-getchannelsupport) method of an [`IGlueWindow`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow) instance. To hide or show the Channel Selector, use the [`SetChannelSupport`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindow-setchannelsupport).

*Note that currently the `SetChannelSupport` method only hides or shows the Channel Selector, but doesn't enable or disable the Channel support. This can be done only during window registration.*

```delphi
// Toggle the visibility of the Channel Selector.
if  glueWin.GetChannelSupport() then
  glueWin.SetChannelSupport(False)
else
  glueWin.SetChannelSupport(True);
``` 

## Handling Channel Change
 
To handle the event when the user changes the Channel via the Channel Selector, use the [`HandleChannelChanged`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler-handlechannelchanged) method of the [`IGlueWindowEventHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler) interface:

```delphi
function TMainForm.HandleChannelChanged(const glueWindow: IGlueWindow; const channel: IGlueContext; prevChannel: GlueContext): HResult;
begin
  if not Assigned(channel) then
  begin
    // Previous Channel deselected, no new Channel selected.
    ...
  end  
  else
  begin
    // New Channel selected.
    ...
  end;
  Result := S_OK;
end;
```

## Subscribing for Channel Updates

To handle the event when the data in the current Channel is updated or the user selects a new Channel, use the [`HandleChannelData`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler-handlechanneldata) method of the [`IGlueWindowEventHandler`](../../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-igluewindoweventhandler) interface:

```delphi
function TMainForm.HandleChannelData(const glueWindow: IGlueWindow; const channelUpdate: IGlueContextUpdate): HResult;
var
  channel: IGlueContext;
  data: TGlueContextValueArray;
begin
  channel := channelUpdate.GetContext();
  // Get the Channel data in a native data structure.
  rawData := channel.GetData();
  data := SA_AsTranslatedContextValues(rawData);
  ...
  SafeArrayDestroy(rawData);
  Result := S_OK;
end;
```