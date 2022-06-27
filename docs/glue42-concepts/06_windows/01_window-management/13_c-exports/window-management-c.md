## Registering Windows

*See the C++ [MFC example](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/mfc-example) on GitHub.*

You can register the main and child windows of your app as Glue42 Windows in order to enable them to consume Glue42 functionalities ([Interop](../../../data-sharing-between-apps/interop/c-exports/index.html), [Channels](../../../data-sharing-between-apps/channels/c-exports/index.html), [Shared Contexts](../../../data-sharing-between-apps/shared-contexts/c-exports/index.html)) and participate in all Glue42 visual integration operations like grouping or ungrouping with other Glue42 Windows. Window registration is usually executed when [initializing Glue42](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#initialization) - the stage where you can also handle the [save and restore state of the main window](../../../application-management/c-exports/index.html#app_state-main_window) in case it's saved or restored in a Global [Layout](../../layouts/overview/index.html), [register an app factory](../../../application-management/c-exports/index.html#app_factories) and handle the [save and restore state of child windows](../../../application-management/c-exports/index.html#app_state-child_windows).

### Main Window

To register the main window of your app, use [`glue_register_main_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow) or [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow).

The [`glue_register_main_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow) method accepts as arguments the window handle - `HWND`, an [app event](../../../application-management/c-exports/index.html#events) handler as an [`app_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction), a [window event](#events) handler as a [`glue_window_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction), an optional title for the Glue42 Window, and an optional callback cookie.

The following example demonstrates how to register the main window of your app using [`glue_register_main_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow):

```cpp
// Here `my_app` is a previously defined MFC object from which you can extract the window handle.
const HWND handle = my_app->m_pMainWnd->m_hWnd;

// Callback for handling Glue42 app events.
void app_event_callback (glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    std::cout << "App event for main window: " << command << std::endl;
}

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for main window: " << command << std::endl;
}

// Register the main window of your app.
glue_register_main_window(handle, &app_event_calback, &window_event_callback, "Main Window");
```

The [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow) method accepts as arguments the window handle - `HWND`, a [window event](#events) handler as a [`glue_window_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction), an optional title for the Glue42 Window, an optional callback cookie, and a `bool` flag indicating whether this is the main (startup) or a child window - it must be set to `true` when registering the main window.

The following example demonstrates how to register the main window of your app using [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow):

```cpp
// Here `my_app` is a previously defined MFC object from which you can extract the window handle.
const HWND handle = my_app->m_pMainWnd->m_hWnd;

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for main window: " << command << std::endl;
}

// Register the main window by setting the startup flag to `true`.
glue_register_window(handle, &window_event_callback, "Main Window", nullptr, true);
```

*Note that [`glue_register_main_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow) and [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow) will register the main window simultaneously as a Glue42 Window and as a [Glue42 app](../../../application-management/c-exports/index.html#registering_the_main_window) within the Glue42 environment.*

### Child Windows

To register a child window of your app, use [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow), as when registering the main window, but either omit or explicitly set the startup `bool` flag to `false` to indicate that this is a child window.

Child windows won't be saved or restored in a [Layout](../../layouts/overview/index.html), but can participate in all visual operations like sticking or unsticking to other Glue42 Windows, and can also consume Glue42 functionalities.

The following example demonstrates how to register a child window of your app:

```cpp
// Here `my_child_window` is a previously defined MFC object from which you can extract the window handle.
const HWND handle = my_child_window->m_hWnd;

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for child window: " << command << std::endl;
}

// Register a child window - omitting the startup `bool` flag automatiaclly sets it to `false`.
glue_register_window(handle, &window_event_callback, "Child Window");
```

## Events

To handle Glue42 Window events (initializing the window, changing or updating the Channel to which the window is joined), use the [`glue_window_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction) passed to [`glue_register_main_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow) or [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow) when registering the window. It will be invoked with a window command as a [`glue_window_command`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#enums-gluewindowcommand) indicating the Glue42 Window event, the name of the Channel to which the window is currently joined, and an optional callback cookie.

The following example demonstrates how to handle Glue42 Window events in the main window of your app:

```cpp
// Here `my_app` is a previously defined MFC object from which you can extract the window handle.
HWND handle = my_app->m_pMainWnd->m_hWnd;

// Callback for handling Glue42 app events.
void app_event_callback (glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    std::cout << "App event for main window: " << command << std::endl;
}

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {

    // Handle the Glue42 Window events based on the received command.
    switch (command) {
        // The event triggered when the window has been initialized.
        case glue_window_command::init:
            std::cout << "The window has been initialized." << std::endl;
            break;
        // The event triggered when the window Channel has been changed.
        case glue_window_command::channel_switch:
            std::cout << "Window has been switched to Channel \"" << context_name << "\"." << std::endl;
            break;
        // The event triggered when the window Channel has been updated.
        case glue_window_command::data_update:
            std::cout << "Channel \"" << context_name << "\" has been updated." std::endl;
            break;
        default:
            break;
    }
}

glue_register_main_window(handle, &app_event_callback, &window_event_callback, "Main Window");
```

Glue42 Window events for child windows can be handled in the same manner either in the [`glue_window_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction) passed to [`glue_register_window()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow) when you register a child window, or in the [`glue_window_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction) passed to [`glue_app_announce_instance()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappannounceinstance) when you [register an app factory](../../../application-management/c-exports/index.html#app_factories) for child windows.