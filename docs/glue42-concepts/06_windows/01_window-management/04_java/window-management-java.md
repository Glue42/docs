## Glue42 Windows

In order for windows created from an external Java apps to become Glue42 Windows, they must first be registered via the Java Window Management API.

### Registering a Swing Window

The Window Management API is accessible through `glue.windows()`.

Currently, Glue42 Windows requires the underlying native handle (`hwnd` in the case of Windows) to be initialized and attached to the `JFrame` before the window can be registered. This is achieved internally through Glue42 Windows:

```java
WindowHandle<JFrame> handle = windows.getWindowHandle(frame);
glue.windows().register(handle)
        .thenAccept(window ->
                frame.addWindowListener(new WindowAdapter()
                {
                    @Override
                    public void windowClosing(WindowEvent e)
                    {
                        window.closeAsync();
                    }
                }));
```

## Controlling the Window

Once an app window is registered, Glue42 Windows will accept full control over the window position, size and visibility. The app shouldn't use native methods (for example, Swing calls) to control the window as this will interfere with the Glue42 window management.

The Java Window Management API allows you to control the following window properties:

### Type

The window type is controlled by the `mode` window option which can be specified in the app definition or during window registration:

```java
glue.windows().register(handle, options -> options.mode(WindowMode.FLAT));
```

### Title

The following example demonstrates how to set the window title during the window registration (this will ignore the title specified in the app configuration):

```java
glue.windows().register(handle, options -> options.title("My Title"));
```

To change the window title at runtime, use the `changeTitle()` method of a Glue42 Window instance and pass the new title as an argument:

```java
window.changeTitle("New Title");
```

### Size & Position

To change the window bounds, use the `changeBounds()` method:

```java
window.changeBounds(new Bounds(10, 10, 200, 200));
```

### Visibility

To hide or show the window, use the `changeVisibility()` method. Note that changing the window visibility also affects its associated icon:

```java
window.changeVisibility(false);
```

## Frame Buttons

You can add extra buttons in the frame area of the window and [handle clicks](#window_events-frame_buttons) for them.

### Adding Buttons

To add a new button to the window frame, use the `addFrameButton()` method:

```java
window.addFrameButton("search-button",
                      ButtonOptions.builder()
                              .toolTip("Search")
                              .order(1)
                              .image(new byte[0]) // needs to be a valid image
                              .build())
        .thenRun(() -> System.out.println("created button"));
```

### Removing Buttons

To remove a button from the window frame, use the `removeFrameButton()` method:

```java
window.removeFrameButton("search-button")
        .thenRun(() -> System.out.println("removed button"));
```

## Window Events

The Java Window Management API offers methods for handling Glue42 Window events related to changes of the window title, bounds, visibility, context and more.

### Title

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.15">

To subscribe for changes of the window title, use the `onTitleChanged()` method:

```java
window.onTitleChanged(e -> System.out.println("Window title changed to: " + e.getTitle()));
```

### Size & Position

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.15">

To subscribe for changes of the window bounds, use the `onBoundsChanged()` method:

```java
window.onBoundsChanged(e -> System.out.println("Window bounds changed to: " + e.getBounds()));
```

### Visibility

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.15">

To subscribe for changes of the window visibility, use the `onVisibilityChanged()` method:

```java
window.onVisibilityChanged(e -> System.out.println("Window is now " + (e.isVisible() ? "visible." : "hidden.")));
```

### Focus

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.15">

To subscribe for changes of the window focus, use the `onFocusChanged()` method:

```java
window.onFocusChanged(e -> System.out.println("Window " + (e.isFocused() ? "is now on focus." : "has lost focus.")));
```

### Context

To subscribe for updates of the window context, use the `onContextUpdated()` method:

```java
window.onContextUpdated(e -> System.out.println("Window context udpated: " + e.getContext()));
```

### Frame Buttons

To subscribe for clicks on any [frame buttons](#frame_buttons) you may have added to the window, use the `onFrameButtonClicked()` method:

```java
window.onFrameButtonClicked(e -> {
    if ("search-button".equals(e.getButtonId()))
    {
        System.out.println("Search button clicked");
    }
});
```