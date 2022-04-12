## Registering the Main Window

*See the C++ [MFC example](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/mfc-example) on GitHub.*

The main window of your app is automatically registered and announced as a Glue42 app when you register it as a Glue42 Window using [`glue_register_main_window()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow) or [`glue_register_window()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow):

```cpp
// Here `my_app` is a previously defined MFC object from which you can extract the window handle.
const HWND handle = my_app->m_pMainWnd->m_hWnd;

// Callback for handling Glue42 application events.
void app_event_callback (glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    std::cout << "App event for main window: " << command << std::endl;
}

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for main window: " << command << std::endl;
}

// The main window of your app will be registered as a Glue42 Window and as a Glue42 app.
glue_register_main_window(handle, &app_event_calback, &window_event_callback, "Main Window");
```

The `app_event_callback` passed as a second argument is an [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) which allows you to handle [app events](#events) for the main window.

*For more details on registering app windows as Glue42 Windows, see the [Window Management](../../windows/window-management/c-exports/index.html) section.*

## App Factories

Registering an app factory allows you to register the child windows of your app as Glue42 apps, announce their instances in the Glue42 environment and handle [app events](#events) for them. To register an app factory, use [`glue_app_register_factory()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappregisterfactory). It accepts as arguments the name of the app to be created, an app event handler as an [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction), and an optional callback cookie. The app event handler will be invoked with the `create` [`glue_app_command`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#enums-glueappcommand) when an instance of the child app is created.

To announce the created app instance in the Glue42 environment, use [`glue_app_announce_instance()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappannounceinstance) within the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction). The [`glue_app_announce_instance()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappannounceinstance) method accepts as arguments a correlational callback (the same one that is passed as an argument to the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction)), the window handle, an app event handler as an [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction), a window event handler as a [`glue_window_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction), and an optional cookie.

The following example demonstrates how to register an app factory and announce the child instances in the Glue42 environment:

```cpp
// Callback for handling app events for each announced child instance. Registered when announcing the child.
void child_app_event_callback(const glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    std::cout << "App event for child window: " << command << std::endl;
}

// Callback for handling window events for each announced child instance. Registered when announcing the child.
void child_window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for child window: " << command << std::endl;
}

// Callback that will be invoked with the `create` command when a child instance is created by the app factory.
void app_factory_callback (glue_app_command command, const void* callback, const ::glue_payload* payload, COOKIE cookie) {
    switch (command) {
        case glue_app_command::create: {
            // Here `my_child_window` is a previously defined MFC object from which you can extract the window handle.
            const HWND handle = my_child_window->m_hWnd;

            // Announce the child instance passing the same correlational callback received from the `app_factory_callback`
            // the child window handle, and the app and window event handlers.
            glue_app_announce_instance(callback, handle, &child_app_event_callback, &child_window_event_callback);

            // Alternatively, factories can deny creating instances by pushing failure:
            // glue_push_failure(callback, "Can't create a child instance.")
            break;
        }
        default:
            break;
    }
}

// Register the app factory.
const void* my_app_factory = glue_app_register_factory("Child App", &app_factory_callback);
```

Registering an app factory returns a reference to the created app factory. Use [`glue_destroy_resource()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource) to destroy it:

```cpp
glue_destroy_resource(my_app_factory);
```

## App State

You can save a state for the main and child windows of your app, as well as restore an initial or previously saved state. The initial state of an app can be passed through [application configuration](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#application_configuration).

### Main Window

To save and restore a state for your main window, use the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) passed to [`glue_register_main_window()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow) when registering your main window. When the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) is invoked with the `save` [`glue_app_command`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#enums-glueappcommand), push a state to be saved, and when it's invoked with the `init` command, restore the saved state using the passed payload:

```cpp
// Here `my_app` is a previously defined MFC object from which you can extract the window handle.
const HWND handle = my_app->m_pMainWnd->m_hWnd;

// Callback for handling Glue42 application events.
void app_event_callback (glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    // Check the command with which the app event handler was invoked.
    switch (command) {
        case glue_app_command::save:
            // Use the Glue42 methods prefixed with `glue_push_` to push a save state.
            glue_push_json_payload(callback, "{glue: {a: 42, b: \"forty-two\"}}");
            break;
        case glue_app_command::init: {
            // Use the Glue42 methods prefixed with `glue_read_` to extract the initial or restored state.
            const auto restore_state = glue_read_json(payload->reader, nullptr);
            std::cout << "Restored state: " << restore_state << std::endl;
            break;
        }
        default:
            break;
    }
}

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for main window: " << command << std::endl;
}

glue_register_main_window(handle, &app_event_calback, &window_event_callback, "Main Window");
```

If you have registered your main window via [`glue_register_window()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterwindow), which doesn't accept a callback for handling application events, you can use [`glue_set_save_state()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesetsavestate) to save you app state. It accepts an [`invocation_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-invocationcallbackfunction) as an argument which you can use to push a state to be saved. The state can then be restored by extracting it from the [**Glue42 Enterprise**](https://glue42.com/enterprise/) starting context with [`glue_get_starting_context_reader()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluegetstartingcontextreader). Before attempting to extract a saved state from the starting context, you must check whether your app has been started by [**Glue42 Enterprise**](https://glue42.com/enterprise/) using [`glue_is_launched_by_gd()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueislaunchedbygd):

```cpp
void save_state_callback (const char* endpoint_name, COOKIE cookie, const ::glue_payload* payload, const void* endpoint) {
    glue_push_json_payload(endpoint, "{my_saved_state: {a: 42, b: \"glue\"}}");
}

// Set a callback that will be invoked whenever user-specific data
// for this instance is to be persisted by Glue42 Enterprise.
glue_set_save_state(&save_state_callback);

// Check whether your app has been started by Glue42 Enterprise.
if (glue_is_launched_by_gd()) {
    // Get the starting context reader and extract the initial or restored state.
    const auto starting_context_reader = glue_get_starting_context_reader();
    const auto val = glue_read_glue_value(starting_context_reader, "state");
}

// Here `my_app` is a previously defined MFC object from which you can extract the window handle.
const HWND handle = my_app->m_pMainWnd->m_hWnd;

// Callback for handling Glue42 Window events.
void window_event_callback (glue_window_command command, const char* context_name, COOKIE cookie) {
    std::cout << "Window event for main window: " << command << std::endl;
}

// Register the main window.
glue_register_window(handle, &window_event_callback, "Main Window", nullptr, true);
```

### Child Windows

The save and restore state of child windows is handled in the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) passed to [`glue_app_announce_instance()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappannounceinstance) when announcing a child app instance created by an [app factory](#app_factories). When the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) is invoked with the `save` [`glue_app_command`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#enums-glueappcommand), push a state to be saved, and when it's invoked with the `init` command, restore the saved state using the passed payload:

```cpp
// Callback for handling app events for each announced child instance.
// Passed as an argument to `glue_app_announce_instance()`.
void child_app_event_callback(const glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    // Check the command with which the app event handler was invoked.
    switch (command) {
        case glue_app_command::save:
            // Use the Glue42 methods prefixed with `glue_push_` to push a save state.
            glue_push_json_payload(callback, "{glue: {a: 42, b: \"forty-two\"}}");
            break;
        case glue_app_command::init: {
            // Use the Glue42 methods prefixed with `glue_read_` to extract the initial or restored state.
            const auto restore_state = glue_read_json(payload->reader, nullptr);
            std::cout << "Restored state: " << restore_state << std::endl;
            break;
        }
        default:
            break;
    }
}
```

*For a full example of registering an app factory for child windows, see the [App Factories](#app_factories) section.*

## Events

To handle Glue42 app events (creating a child instance, saving and restoring state, shutdown), use the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) passed to [`glue_register_main_window()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregistermainwindow), [`glue_app_register_factory()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappregisterfactory) or [`glue_app_announce_instance()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappannounceinstance). It will be invoked with an app command as a [`glue_app_command`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#enums-glueappcommand) indicating the Glue42 app event, a correlational callback that can be passed to other Glue42 methods (e.g., [`glue_app_announce_instance()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueappannounceinstance) when announcing a newly created app instance, or to [`glue_push_payload()`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushpayload) when saving the application state), a [`glue_payload`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluepayload) value holding the initial or restored state, and an optional callback cookie.

The `create` [`glue_app_command`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#enums-glueappcommand) is used only when creating child app instances and therefore this event can be handled only when registering an [app factory](#app_factories):

```cpp
// Callback that will be invoked with the `create` command when a child instance is created by the app factory.
void app_factory_callback (glue_app_command command, const void* callback, const ::glue_payload* payload, COOKIE cookie) {
    switch (command) {
        case glue_app_command::create: {
            // Create and announce a child app instance.
            // Handle the app events related to that instance.
            break;
        }
        default:
            break;
    }
}

// Register the app factory.
const void* my_app_factory = glue_app_register_factory("Child App", &app_factory_callback);
```

All other app events can be handled in the [`app_callback_function`](../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-appcallbackfunction) when registering a main window or announcing a child instance:

```cpp
// Callback for handling Glue42 application events.
void app_event_callback (glue_app_command command, const void* callback, const glue_payload* payload, COOKIE cookie) {
    // Check the command with which the app event handler was invoked.
    switch (command) {
        case glue_app_command::save:
            // Use the Glue42 methods prefixed with `glue_push_` to push a save state.
            glue_push_json_payload(callback, "{glue: {a: 42, b: \"forty-two\"}}");
            break;
        case glue_app_command::init: {
            // Use the Glue42 methods prefixed with `glue_read_` to extract the initial or restored state.
            const auto restore_state = glue_read_json(payload->reader, nullptr);
            std::cout << "Restored state: " << restore_state << std::endl;
            break;
        }
        case glue_app_command::shutdown:
            std::cout << "The application will shutdown." << std::endl;
            // Close the window and free resources.
            break;
        default:
            break;
    }
}
```