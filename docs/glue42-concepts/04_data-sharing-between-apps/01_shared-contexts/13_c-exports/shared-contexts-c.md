## Subscribing for Context Updates

*See the C++ [Console example](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/cpp-console-example) on GitHub.*

To subscribe for context updates, use [`glue_subscribe_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribecontext). It accepts as arguments the shared context name, an optional dot-separated field path (in case you are interested in a specific context field), a context update handler as a [`context_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-contextfunction), and an optional callback cookie.

The following example demonstrates how to subscribe to a shared context named "MyContext":

```cpp
// Callback for handling context updates.
void context_update_callback (const char* context_name, const char* field_path, const glue_value* glue_value, COOKIE cookie) {
        std::cout << "Received update from context: " << context_name << std::endl;
}

// Subscribe for updates of a shared context.
const void* my_subscription = glue_subscribe_context("MyContext", nullptr, &context_update_callback);
```

Subscribing to a shared context returns a reference to the created context subscription. To unsubscribe from the shared context, use [`glue_destroy_resource()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource) and pass the reference to the context subscription:

```cpp
// Unsubscribe from the shared context.
glue_destroy_resource(my_subscription);
```

## Handling Context Updates

The [`context_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-contextfunction) passed to [`glue_subscribe_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribecontext) will be invoked each time the shared context is updated. It allows you to extract and handle the received data.

The following example demonstrates how to subscribe to a specific field in a shared context and handle the updates:

```cpp
// Handle the received data in the callback passed when subscribing to the context.
void context_update_callback (const char* context_name, const char* field_path, const glue_value* glue_value, COOKIE cookie) {

    if (glue_value == nullptr) {
        std::cout << "No update available for field \"" << field_path << "\" of context \"" << context_name << "\"." std::endl;

        return;
    }
    else {
        // Extract and use the new context data.
        const auto new_price = glue_value->f;

        std::cout << "New price: " << new_price << std::endl;
    }
}

const void* my_subscription = glue_subscribe_context("MyContext", "data.instrument.price", &context_update_callback);
```

## Reading Context Data

You can access shared context data synchronously, as well as asynchronously.

### Sync

To read shared context data synchronously, use [`glue_read_context_sync()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadcontextsync) and pass the context name as an argument. This returns a reference to the reader for the specified shared context:

```cpp
// Get a reference to a context reader.
const void* context = glue_read_context_sync("MyContext");
```

After getting a reference to the context reader, use the Glue42 methods prefixed with `glue_read_` to retrieve data from the shared context. The following example demonstrates how to read different types of values from a shared context using [`glue_read_glue_value()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadgluevalue), [`glue_read_s()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereads) and [`glue_read_d()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadd):

```cpp
const void* context = glue_read_context_sync("MyContext");

// Read as a `glue_value`, validate the value and use it.
const glue_value instrument_name_gv = glue_read_glue_value(context, "data.instrument.name");

if (instrument_name_gv.type == glue_type::glue_string) {
    std::cout << "Intrument name: " << instrument_name_gv << std::endl;
}

// Read as a `string`, validate the value and use it.
const auto instrument_name_s = glue_read_s(context, "data.instrument.name");

if (instrument_name_s) {
    std::cout << "Intrument name: " << instrument_name_s << std::endl;
}

// Read as a `double`, validate the value and use it.
const auto instrument_price_d = glue_read_d(context, "data.instrument.price");

if (instrument_price_d) {
    std::cout << "Intrument price: " << instrument_price_d << std::endl;
}
```

### Async

To read shared context data asynchronously, use [`glue_read_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadcontext). It accepts as arguments the shared context name, a dot-separated field path designating the context field from which to read the data, a context data handler as a [`context_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-contextfunction), and an optional callback cookie. The [`glue_read_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluereadcontext) method returns an integer value indicating whether the data has been read successfully (`0` for success):

```cpp
// Callback for handling the context data.
void context_data_callback (const char* context_name, const char* field_path, const glue_value* glue_value, COOKIE cookie) {
    if (glue_value->type == glue_type::glue_double) {

        // Extract and use the context data.
        const auto new_price = glue_value->d;

        std::cout << "Shared context: " << context_name << std::endl;
        std::cout << "New price: " << new_price << std::endl;
    }
}

// Read the shared context data asynchronously.
const int read_status = glue_read_context("MyContext", "data.instrument.price", &context_data_callback);

// Check the status of reading the data.
if (read_status != 0) {
    std::cout << "Error reading data from context." << std::endl;
}
```

## Updating a Context

You can update a shared context directly or by using a context writer.

### Directly

To update a shared context directly, use [`glue_write_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluewritecontext). Pass the shared context name, a dot-separated field path designating the context field in which to write, and the value to write as a [`glue_value`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluevalue). The [`glue_write_context()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluewritecontext) method returns an integer value indicating whether the data has been written successfully (`0` for success):

```cpp
// Create a `glue_value` to pass as a context update.
const glue_value new_price = glv_d(42.42);

// Update the shared context.
const int write_status = glue_write_context("MyContext", "data.instrument.price", new_price);

// Check the status of writing the data.
if (write_status != 0) {
    std::cout << "Error updating the context." << std::endl;
}
```

### Using a Writer

To get a reference to a writer for a shared context, use [`glue_get_context_writer()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluegetcontextwriter). Pass the shared context name and a dot-separated field path designating the context field in which the data will be written.

```cpp
// Get a reference to a context writer.
const void* writer = glue_get_context_writer("MyContext", "data.instrument.price");
```

After getting a reference to a context writer, use the Glue42 methods prefixed with `glue_push_` to push data to the shared context. The following example demonstrates how to update a field in a shared context with JSON data by using [`glue_push_json_payload()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushjsonpayload):

```cpp
const void* writer = glue_get_context_writer("MyContext", "data.instrument");

// String in JSON format containing the data.
const std::string json_data = "{name: \"VOD.L\", price: 235.42}";

// Update the context field with the JSON data.
glue_push_json_payload(writer, json_data);
```

To destroy the context writer, use [`glue_destroy_resource()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource) and pass the reference to the context writer:

```cpp
glue_destroy_resource(writer);
```