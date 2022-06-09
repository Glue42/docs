## Method Registration

Registering Interop methods enables your app to offer functionality to other Glue42 enabled apps.

*See the C++ [Console example](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/cpp-console-example) on GitHub.*

### Registering Methods

To expose a Glue42 method that can be invoked by other Glue42 enabled applications, use [`glue_register_endpoint()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterendpoint) and provide a method name and an invocation handler as the first two arguments. The third argument is an optional callback cookie.

The function for handling method invocations must be an [`invocation_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-invocationcallbackfunction) which will receive as arguments the name of the invoked Interop method, an optional callback cookie, a [`glue_payload`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluepayload) value holding an array with the invocation arguments and a correlational endpoint which may be used to send an invocation result back to the invoking application.

The following example demonstrates how to register an Interop method that prints the method name, the invocation origin point and the length of the invocation arguments array to the console:

```cpp
// Callback for handling invocation requests.
void invocation_callback(const char* endpoint_name, COOKIE cookie, const glue_payload* payload, const void* result_endpoint) {
    std::cout << "Method " << endpoint_name << " invoked by " << payload->origin << "with " << payload->args_len << " arguments." << std::endl;
}

// Register the Interop method.
glue_register_endpoint("PrintToConsole", &invocation_callback, "InvocationCookie");
```

### Returning Results

To return a result to the invoking app, you must use the Glue42 methods prefixed with `glue_push_`:

- to push a result, use either [`glue_push_payload()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushpayload), or [`glue_push_json_payload()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushjsonpayload);

- to push a failure message, use [`glue_push_failure()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushfailure);

The following example demonstrates registering an Interop method that returns the sum of two or more whole numbers:

```cpp
void invocation_callback (const char* endpoint_name, COOKIE cookie, const glue_payload* payload, const void* result_endpoint) {

    // Extract the invocation arguments.
    const auto args = payload->args;

    // Validate the arguments.
    if (payload->args_len < 2) {

        // Send a failure message in case of invalid arguments.
        glue_push_failure(result_endpoint, "Incorrect number of arguments.");
        return;
    }

    long long sum = 0;
    for (int i = 0; i < payload->args_len; i++) {
        // Check the value type and extract the value.
        switch (args[i].value.type) {
            case glue_type::glue_int:
                sum += args[i].value.i;
                break;
            case glue_type::glue_long:
                sum += args[i].value.l;
                break;
            default:
                // Filter out other value types.
                break;
        }
    }

    // Wrap the result as a `glue_arg` value and send it back
    // to the invoking app as a `glue_arg` array.
    const glue_arg result[] = { glarg_l("sum", sum) };

    glue_push_payload(result_endpoint, result, 1);
}

glue_register_endpoint("Addition", &invocation_callback);
```

## Method Invocation

You can invoke any already registered Interop method from your app. If multiple applications or application instances have registered the same Interop method, you can choose whether to invoke the method only on the best Interop server (usually the first that has registered it), or on all Interop servers offering it (see [Targeting](#method_invocation-targeting)). After invoking an Interop method, you must [handle the returned result](#method_invocation-handling_invocation_results), if any.

### Invoking Methods

The following example demonstrates invoking the previously registered "Addition" Interop method using [`glue_invoke()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueinvoke). It accepts as arguments the method name, an array of invocation arguments, the length of the arguments array, a result handler and an optional callback cookie. The invocation arguments must be a [`glue_arg`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluearg) array and the callback function for handling the invocation result must be a [`payload_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-payloadfunction):

```cpp
// Callback for handling the result returned from the invocation.
void result_callback (const char* origin, COOKIE cookie, const glue_payload* glue_payload) {
    std::cout << "Invoked method " << origin << " and received " << glue_payload->args[0].value.l << " as a result." << std::endl;
}

// Wrap the invocation arguments as `glue_arg` values in a `glue_arg` array.
const glue_arg invocation_args[] = { glarg_i("b", 7), glarg_l("a", 35) };

// Invoke the Interop method.
glue_invoke("Addition", invocation_args, 2, &result_callback);
```

### Targeting

You can choose whether to invoke an Interop method only on the best server (usually the first that has registered it) or on all servers offering it.

To invoke an Interop method only on the best server, use [`glue_invoke()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueinvoke).

To invoke an Interop method on all servers offering it, use [`glue_invoke_all()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueinvokeall).

The following example demonstrates how to invoke the previously registered "Addition" Interop method using [`glue_invoke_all()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueinvokeall). It accepts as arguments the method name, an array of invocation arguments, the length of the arguments array, a multiple results handler and an optional callback cookie. The invocation arguments must be a [`glue_arg`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluearg) array and the callback function for handling the invocation results must be a [`multiple_payloads_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-multiplepayloadsfunction):

```cpp
// Callback for handling the results from multiple invocations.
void multiple_results_callback (const char* origin, COOKIE cookie, const glue_payload* payloads, int len) {
    std::cout << "Invoked method " << origin << " on " << len << " Interop targets." << std::endl;
}

const glue_arg invocation_args[] = { glarg_i("b", 7), glarg_l("a", 35) };

// Invoke all registered Interop methods with the same name.
glue_invoke_all("Addition", invocation_args, 2, &multiple_results_callback);
```

### Handling Invocation Results

If the invoked Interop method returns a result, use the [`payload_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-payloadfunction) or the [`multiple_payloads_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-multiplepayloadsfunction) passed respectively to [`glue_invoke()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueinvoke) and [`glue_invoke_all()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueinvokeall) to handle the result from single or multiple invocations.

The following example demonstrates handling multiple results from the invocation of the previously registered "Addition" Interop method invoked on all servers offering it:

```cpp
// Callback for handling the results from multiple invocations.
void multiple_results_callback (const char* origin, COOKIE cookie, const glue_payload* payloads, int len) {

    std::cout << "Invoked method " << origin << " on " << len << " Interop targets." << std::endl;

    for (int i = 0; i < len; i++) {
        // Check whether the invocation has been successful, and if so, extract the result.
        const auto result = payloads[i]->status == 0 ? std::to_string(payloads[i]->args[0].value.l) : "No result."

        std::cout << "Result from app " << payloads[i]->origin << ": " << result << std::endl;
    }
}

const glue_arg invocation_args[] = { glarg_i("b", 7), glarg_l("a", 35) };

// Invoke all registered Interop methods with the same name.
glue_invoke_all("Addition", invocation_args, 2, &multiple_results_callback);
```

## Discovery

To get notified when an Interop method has been registered or unregistered by an app, or when the method has been removed because the Interop server offering it has been shut down, use [`glue_subscribe_endpoints_status()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribeendpointsstatus). It accepts a [`glue_endpoint_status_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-glueendpointstatuscallbackfunction) and an optional callback cookie as arguments.

The following example demonstrates how to get notified when any Interop method has been registered or unregistered by any Interop server and check for the method you are interested in:

```cpp
// Callback for handling notifications about registered or unregistered Interop methods.
void method_status_handler (const char* endpoint_name, const char* origin, bool state, COOKIE cookie) {
    std::cout << "Method " << endpoint_name << (state ? " added " : " removed ") << " by app " << origin << std::endl;

    if (endpoint_name == "Addition" && state) {
        std::cout << "The \"Addition\" Interop method is available." << std::endl;
        // The method is available and you can invoke it.
    }
    else if (endpoint_name == "Addition" && !state) {
        std::cout << "The \"Addition\" Interop method is unavailable." << std::endl;
        // The method has been removed either because the Interop server offering it
        // has unregistered it, or because the server itself has been shut down.
    }
}

// Subscribe for notifications about registered or unregistered Interop methods.
glue_subscribe_endpoints_status($method_status_handler);
```

## Streaming

### Overview

Your application can publish events that can be observed by other applications, or it can provide real-time data (e.g., market data, news alerts, notifications, etc.) to other applications by publishing an Interop stream. Your application can also receive and react to these events and data by creating an Interop stream subscription.

Applications that create and publish to Interop Streams are called *publishers*, and applications that subscribe to Interop Streams are called *subscribers*. An application can be both.

<glue42 name="diagram" image="../../../../images/interop/interop-streaming.gif">

Interop Streams are used extensively in [**Glue42 Enterprise**](https://glue42.com/enterprise/) products and APIs:

- in Glue42 Windows - to publish notifications about window status change (events);
- in application configuration settings - to publish application configuration changes, and notifications about application instance state change (events);
- in the Glue42 Notification Service (GNS) Desktop Manager and GNS Interop Servers - to publish Notifications (real-time data);
- in the Window Management and App Management APIs (events);

## Publishing Stream Data

To expose a data stream to which other applications can subscribe, you must register a stream and provide a callback function for handling the server side streaming events (accepting or rejecting subscription requests, assigning subscribers to stream branches). Once a stream has been successfully registered, the publishing application can start pushing data to it.

### Creating Streams

To create an Interop stream, use [`glue_register_streaming_endpoint()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterstreamingendpoint). It accepts as arguments a name for the stream, an optional subscription request handler of type [`stream_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-streamcallbackfunction), an optional method invocation handler of type [`invocation_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-invocationcallbackfunction) (in case the stream is to be treated as a regular Interop method) and an optional callback cookie.

```cpp
// Callback for handling stream subscription requests.
bool stream_subscription_callback (const char* endpoint_name, COOKIE cookie, const glue_payload* payload, const char*& branch) {
    // Accept the subscription.
    return true;
}

// Register the Interop stream and get a reference to it.
const auto my_stream = glue_register_streaming_endpoint("MarketData", &stream_subscription_callback);
```

The [`glue_register_streaming_endpoint()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueregisterstreamingendpoint) method returns a reference to the registered Interop stream which you can use as a correlational endpoint to [push data](#publishing_stream_data-pushing_data) to that stream.

### Accepting or Rejecting Subscription Requests

To accept a subscription request, return `true` from the [`stream_callback_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-streamcallbackfunction), to reject it - return `false`.

The following example demonstrates how to accept or reject a subscription based on a required condition:

```cpp
bool stream_subscription_callback (const char* endpoint_name, COOKIE cookie, const glue_payload* payload, const char*& branch) {

    // Validate the subscription request based on a required condition.
    const bool is_valid_request = payload->args[0].name == "instrument" ? true : false;

    // Accept or reject the subscription based on the condition.
    return is_valid_request;
}

const auto my_stream = glue_register_streaming_endpoint("MarketData", &stream_subscription_callback);
```

### Pushing Data

To push data to an Interop stream, you must use the Glue42 methods prefixed with `glue_push_`:

- to push data, use either [`glue_push_payload()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushpayload), or [`glue_push_json_payload()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushjsonpayload);

- to push a failure message, use [`glue_push_failure()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluepushfailure);

The following example demonstrates how to push data to a previously registered Interop stream as [`glue_arg`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluearg) values or as a string encoded in JSON format:

```cpp
const glue_arg glue_arg_payload[] = { glarg_s("instrument", "VOD.L"), glarg_f("price", 42.42) };
const std::string json_payload = "{instrument: \"VOD.L\", price: 42.42}";

// Pushing data to the stream as `glue_arg` values.
glue_push_payload(my_stream, glue_arg_payload, 2);

// Pushing data to the stream as a JSON-encoded string.
glue_push_json_payload(my_stream, json_payload);
```

### Using Stream Branches

Using stream branches allows you to group subscribers by any criterion and target stream data at specific groups of subscribers. Branches are distinguished by their name (key). Each Glue42 stream has a default (unnamed) branch on which it accepts subscribers and to which it pushes data if no branch is specified.

To accept a subscription on a branch, specify a string as the branch name when accepting the subscription. If the branch doesn't exist, it will be automatically created:

```cpp
bool stream_subscription_callback (const char* endpoint_name, COOKIE cookie, const glue_payload* payload, const char*& branch) {

    // For clarity, the branch is named explicitly with a string literal,
    // but you can assign subscribers to a branch based on the subscription arguments, e.g.:
    // branch = payload.args[0].value.s;
    branch = "VOD.L";

    // Accept the subscription.
    return true;
}

const auto my_stream = glue_register_streaming_endpoint("MarketData", &stream_subscription_callback);
```

To assign a subscriber to the default stream branch, either don't specify a branch name, or use `nullptr`.

To push data only to a specific branch, use [`glue_open_streaming_branch()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-glueopenstreamingbranch) to open an already existing stream branch.

```cpp
// Open the branch of interest.
const auto my_branch = glue_open_streaming_branch(my_stream, "VOD.L");
const std::string json_payload = "{instrument: \"VOD.L\", price: 235.42}";

// Push data as a JSON-encoded string to the specified branch only.
glue_push_json_payload(my_branch, json_payload);
```

## Consuming Stream Data

To receive data published on an Interop stream, your app must subscribe to it and provide a handler for the data. You can subscribe to a single Interop stream created by the best Interop server (usually the first that has registered it), or to all streams with the same name offered by different Interop servers.

### Subscribing to Streams

To subscribe to a single Interop stream offered by the best Interop server, use [`glue_subscribe_single_stream()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribesinglestream). Provide the name of the stream, stream data handler as a [`payload_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-payloadfunction), subscription arguments as a [`glue_arg`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#structs-gluearg) array, the length of the subscription arguments, and an optional callback cookie.

The following example demonstrates how to subscribe to the previously registered "MarketData" Interop stream:

```cpp
// Callback for handling the data received from the Interop stream.
void stream_data_callback (const char* origin, COOKIE cookie, const glue_payload* payload) {
    std::cout << "Received data from the " << origin << " Interop stream." << std::endl;
}

// Subscription arguments.
const glue_arg subscription_args[] = { glarg_s("instrument", "VOD.L") };

// Subscribe to a single Interop stream.
const void* my_subscription = glue_subscribe_single_stream("MarketData", $stream_data_callback, subscription_args, 1);
```

Subscribing to an Interop stream returns a reference to the created stream subscription. To unsubscribe from the stream, use [`glue_destroy_resource()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource) and pass the reference to the stream subscription:

```cpp
// Unsubscribe from the Interop stream.
glue_destroy_resource(my_subscription);
```

To subscribe to multiple Interop streams with the same name offered by different Interop servers, use [`glue_subscribe_stream()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribestream). It accepts the same arguments as [`glue_subscribe_single_stream()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribesinglestream):

```cpp
void stream_data_callback (const char* origin, COOKIE cookie, const glue_payload* payload) {
    std::cout << "Received data from the " << origin << " Interop stream." << std::endl;
}

const glue_arg subscription_args[] = { glarg_s("instrument", "VOD.L") };

// Subscribe to all Interop streams with the same name.
const void* my_subscription = glue_subscribe_stream("MarketData", $stream_data_callback, subscription_args, 1);
```

To unsubscribe from the streams, use [`glue_destroy_resource()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluedestroyresource).

### Handling Subscriptions Client Side

The [`payload_function`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#types-payloadfunction) passed to [`glue_subscribe_single_stream()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribesinglestream) or [`glue_subscribe_stream()`](../../../../getting-started/how-to/glue42-enable-your-app/c-exports/index.html#functions-gluesubscribestream) will be invoked each time data has been pushed to the stream. It allows you to extract and handle the received data:

```cpp
// Handle the received data in the callback passed when subscribing to the stream.
void stream_data_callback (const char* origin, COOKIE cookie, const glue_payload* payload) {

    // Extract and use the stream data.
    const auto instrument = payload->args[0].value.s;
    const auto price = payload->args[1].value.f;

    std::cout << "Received data from stream " << origin << " for instrument " << instrument << " at price: " << price << std::endl;
}

const glue_arg subscription_args[] = { glarg_s("instrument", "VOD.L") };

const void* my_subscription = glue_subscribe_stream("MarketData", $stream_data_callback, subscription_args, 1);
```