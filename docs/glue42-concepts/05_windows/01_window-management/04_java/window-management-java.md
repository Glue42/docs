## Glue42 Windows

In order for windows created from an external Java applications to become Glue42 Windows, they must first be registered via the Java Window Management API.

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

Once an application window is registered, Glue42 Windows will accept full control over the window position, size and visibility. The application shouldn't use native methods (for example, Swing calls) to control the window as this will interfere with the Glue42 window management.

The Java Window Management API allows you to control the following window properties:

### Type

The window type is controlled by the `mode` window option which can be specified in the application definition or during window registration:

```java
glue.windows().register(handle, options -> options.mode(WindowMode.FLAT));
```

### Title

The following example demonstrates how to set the window title during the window registration (this will ignore the title specified in the application configuration):

```java
glue.windows().register(handle, options -> options.title("My Title"));
```

To change the window title at runtime, use the `changeTitle()` method of a Glue42 Window instance and pass the new title as an argument:

```java
window.changeTitle("New Title");
```

### Size and Position

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

You can put extra buttons in the frame area of the window and handle clicks for those buttons.

### Adding a New Button

Use the `addFrameButton()` method to add a new button:

```java
window.addFrameButton("search-button",
                      ButtonOptions.builder()
                              .toolTip("Search")
                              .order(1)
                              .image(new byte[0]) // needs to be a valid image
                              .build())
        .thenRun(() -> System.out.println("created button"));
```

### Removing Button

Use the `removeFrameButton()` method to remove a button from the frame:

```java
window.removeFrameButton("search-button")
        .thenRun(() -> System.out.println("removed button"));
```

### Handling Clicks

Use the `onFrameButtonClicked()` method to subscribe for button clicks:

```java
window.onFrameButtonClicked(e -> {
    if ("search-button".equals(e.getButtonId()))
    {
        System.out.println("Search button clicked");
    }
});
```