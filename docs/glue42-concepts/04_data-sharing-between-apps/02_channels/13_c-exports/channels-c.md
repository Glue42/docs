## Using Channels

*See the C++ [Console example](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/cpp-console-example) on GitHub.*

The Glue42 Channels are named [shared contexts](../../shared-contexts/c-exports/index.html) that can be defined globally in a configuration file (see [Defining Channels](../overview/index.html#defining_channels)).

It is important to note that when using the Glue42 C Exports library to [read Channel data](#reading_channel_data), [subscribe for Channel updates](#subscribing_for_channel_updates) or [update a Channel](#updating_a_channel), you must pass the actual Channel context name and not the Channel name as defined in the configuration file. In the general case, you can obtain and save the name of the Channel context from the [`glue_window_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-gluewindowcallbackfunction) when registering your window and then use it in the various Channel operations.

## Subscribing for Channel Updates

To subscribe for Channel updates, use [`glue_subscribe_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribecontext). It accepts as arguments the Channel context name, an optional dot-separated field path (in case you are interested in a specific field of the Channel context), a Channel context update handler as a [`context_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-contextfunction), and an optional callback cookie.

The following example demonstrates how to subscribe to the "Red" Channel:

```cpp
// Callback for handling Channel context updates.
void channel_update_callback (const char* context_name, const char* field_path, const glue_value* glue_value, COOKIE cookie) {
        std::cout << "Received update from Channel: " << context_name << std::endl;
}

// Subscribe for updates of a Channel context.
// The `channel_context_name` has been obtained previously when registering the window.
const void* my_subscription = glue_subscribe_context(channel_context_name, nullptr, &channel_update_callback);
```

Subscribing to a Channel returns a reference to the created context subscription. To unsubscribe from the Channel, use [`glue_destroy_resource()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource) and pass the reference to the Channel subscription:

```cpp
// Unsubscribe from the Channel.
glue_destroy_resource(my_subscription);
```

## Handling Channel Updates

The [`context_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-contextfunction) passed to [`glue_subscribe_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribecontext) will be invoked each time the Channel is updated. It allows you to extract and handle the received data.

The following example demonstrates how to subscribe to a specific field in a Channel context and handle the updates:

```cpp
// Handle the received data in the callback passed when subscribing to the Channel.
void channel_update_callback (const char* context_name, const char* field_path, const glue_value* glue_value, COOKIE cookie) {

    if (glue_value == nullptr) {
        std::cout << "No update available for field \"" << field_path << "\" of Channel \"" << context_name << "\"." std::endl;

        return;
    }
    else {
        // Extract and use the new Channel data.
        const auto new_price = glue_value->f;

        std::cout << "New price: " << new_price << std::endl;
    }
}

const void* my_subscription = glue_subscribe_context(channel_context_name, "data.instrument.price", &channel_update_callback);
```

## Reading Channel Data

You can access Channel context data synchronously, as well as asynchronously.

### Sync

To read Channel context data synchronously, use [`glue_read_context_sync()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadcontextsync) and pass the Channel context name as an argument. This returns a reference to the reader for the specified Channel context:

```cpp
// Get a reference to a Channel context reader.
const void* channel_context = glue_read_context_sync(channel_context_name);
```

After getting a reference to the Channel context reader, use the Glue42 methods prefixed with `glue_read_` to retrieve data from the Channel context. The following example demonstrates how to read different types of values from a Channel context using [`glue_read_glue_value()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadgluevalue), [`glue_read_s()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereads) and [`glue_read_d()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadd):

```cpp
const void* channel_context = glue_read_context_sync(channel_context_name);

// Read as a `glue_value`, validate the value and use it.
const glue_value instrument_name_gv = glue_read_glue_value(channel_context, "data.instrument.name");

if (instrument_name_gv.type == glue_type::glue_string) {
    std::cout << "Intrument name: " << instrument_name_gv << std::endl;
}

// Read as a `string`, validate the value and use it.
const auto instrument_name_s = glue_read_s(channel_context, "data.instrument.name");

if (instrument_name_s) {
    std::cout << "Intrument name: " << instrument_name_s << std::endl;
}

// Read as a `double`, validate the value and use it.
const auto instrument_price_d = glue_read_d(channel_context, "data.instrument.price");

if (instrument_price_d) {
    std::cout << "Intrument price: " << instrument_price_d << std::endl;
}
```

### Async

To read Channel context data asynchronously, use [`glue_read_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadcontext). It accepts as arguments the Channel context name, a dot-separated field path designating the Channel context field from which to read the data, a Channel context data handler as a [`context_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-contextfunction), and an optional callback cookie. The [`glue_read_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadcontext) method returns an integer value indicating whether the data has been read successfully (`0` for success):

```cpp
// Callback for handling the Channel data.
void channel_data_callback (const char* context_name, const char* field_path, const glue_value* glue_value, COOKIE cookie) {
    if (glue_value->type == glue_type::glue_double) {

        // Extract and use the Channel data.
        const auto new_price = glue_value->d;

        std::cout << "Channel: " << context_name << std::endl;
        std::cout << "New price: " << new_price << std::endl;
    }
}

// Read the Channel context data asynchronously.
const int read_status = glue_read_context(channel_context_name, "data.instrument.price", &channel_data_callback);

// Check the status of reading the data.
if (read_status != 0) {
    std::cout << "Error reading data from Channel." << std::endl;
}
```

## Updating a Context

You can update a Channel context directly or by using a context writer.

### Directly

To update a Channel context directly, use [`glue_write_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluewritecontext). Pass the Channel context name, a dot-separated field path designating the Channel context field in which to write, and the value to write as a [`glue_value`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluevalue). The [`glue_write_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluewritecontext) method returns an integer value indicating whether the data has been written successfully (`0` for success):

```cpp
// Create a `glue_value` to pass as a Channel update.
const glue_value new_price = glv_d(42.42);

// Update the Channel.
const int write_status = glue_write_context(channel_context_name, "data.instrument.price", new_price);

// Check the status of writing the data.
if (write_status != 0) {
    std::cout << "Error updating the Channel." << std::endl;
}
```

### Using a Writer

To get a reference to a writer for a Channel context, use [`glue_get_context_writer()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluegetcontextwriter). Pass the Channel context name and a dot-separated field path designating the Channel context field in which the data will be written.

```cpp
// Get a reference to a Channel context writer.
const void* writer = glue_get_context_writer(channel_context_name, "data.instrument.price");
```

After getting a reference to a Channel context writer, use the Glue42 methods prefixed with `glue_push_` to push data to the Channel. The following example demonstrates how to update a field in a Channel context with JSON data by using [`glue_push_json_payload()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushjsonpayload):

```cpp
const void* writer = glue_get_context_writer(channel_context_name, "data.instrument");

// String in JSON format containing the data.
const std::string json_data = "{name: \"VOD.L\", price: 235.42}";

// Update the Channel context field with the JSON data.
glue_push_json_payload(writer, json_data);
```

To destroy the Channel context writer, use [`glue_destroy_resource()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource) and pass the reference to the Channel context writer:

```cpp
glue_destroy_resource(writer);
```