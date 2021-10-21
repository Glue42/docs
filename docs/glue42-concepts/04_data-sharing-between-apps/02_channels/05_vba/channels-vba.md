## Overview

The Glue42 Channels are enabled by default when registering a VBA window as a Glue42 Window. You have to provide handler implementations for the Channel related events that will be raised due to interaction with the Glue42 Window instance - [changing Channels](#handling_channel_change) and [handling Channel data updates](#subscribing_for_channel_updates). When the Glue42 Channels are enabled, a Channel Selector box is available in the window title bar.

To check whether the Channel Selector is visible, use the [`GetChannelSupport`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-getchannelsupport) method of a [`GlueWindow`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow) instance. To hide or show the Channel Selector, use the [`SetChannelSupport`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-setchannelsupport) method.

*Note that currently the `SetChannelSupport` method only hides or shows the Channel Selector, but does not enable or disable the Channel support. This can be done only during window registration.*

```vbnet
' Hiding the Channel selector.
GlueWin.SetChannelSupport False
```

## Handling Channel Change

The [`HandleChannelChanged`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlechannelchanged) event is raised when the user changes the Channel using the Channel selection box in the window title bar. Use its handler to enable your application to react to Channel changes:  

```vbnet
Private Sub GlueWin_HandleChannelChanged(ByVal GlueWindow As IGlueWindow, ByVal channel As IGlueContext, ByVal prevChannelName As String)
    On Error GoTo HandleErrors

    Dim ChannelContext
    Dim NewData
    Dim NewChannelName As String

    If channel Is Nothing Then
        ' Do something when the user deselects a Channel.
        ...
    Else
        ChannelContext = channel.GetReflectData("")    
        NewChannelName = ChannelContext("name")
        NewData = ChannelContext("data")
        ' Do something when the user selects a new Channel.
        ...
    End If
    Exit Sub

    HandleErrors:
    ' Handle exceptions.

End Sub
```
  
## Subscribing for Channel Updates

To subscribe for Channel data updates, use the [`HandleChannelData`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlechanneldata) event which is raised in two cases:
- when the data in the currently selected Channel is updated;
- after the [`HandleChannelChanged`](../../../../getting-started/how-to/glue42-enable-your-app/vba/index.html#classes-gluewindow-handlechannelchanged) event, when the user switches to another Channel;  

```vbnet
Private Sub GlueWin_HandleChannelData(ByVal GlueWindow As IGlueWindow, ByVal channelUpdate As IGlueContextUpdate)
    On Error GoTo HandleErrors
    
    Dim ChannelContext
    Dim Data
    Dim ChannelName As String

    ChannelContext = channelUpdate.GetContext.GetReflectData("")
    ChannelName = ChannelContext("name")
    Data = ChannelContext("data")
    ' Do something with the new data.
    ...
    Exit Sub

    HandleErrors:
    ' Handle exceptions.
   
End Sub
``` 