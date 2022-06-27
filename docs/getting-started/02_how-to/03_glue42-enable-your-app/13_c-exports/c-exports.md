## Overview

The Glue42 C Exports library offers easy access to Glue42 functionalities via exported C functions. It can be used by any native language that supports external C functions.

All examples of using the Glue42 C Exports library are in C++.

*See the C++ [Console](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/cpp-console-example) and [MFC](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/mfc-example) examples on GitHub demonstrating various [**Glue42 Enterprise**](https://glue42.com/enterprise/) features.*

## Referencing

The [Glue42 C Exports](https://github.com/Glue42/native-examples/tree/main/glue-c-exports/glue-cli-lib) library is available on GitHub.

To use the Glue42 C Exports library, include its header file in your project:

```cpp
#include "GlueCLILib.h"
```

*Note that you must add the `GlueCLILib.lib` file to your project dependencies in the IDE (e.g., `Linker > Input > Additional Dependencies` for Visual Studio). All necessary dependencies (`GlueCLILib.lib`, `GlueCLILib.dll`, `Glue42.dll`) must be located in the directory where you will build the EXE file.*

## Initialization

To initialize the Glue42 library in your app, use the [`glue_init()`](#functions-glueinit) method passing a name for your app, a callback for handling Glue42 connection status events and an optional callback cookie. The connection status handler must be a [`glue_init_callback_function`](#types-glueinitcallbackfunction) that will be invoked with the current connection state in the form of a [`glue_state`](#enums-gluestate) value, a message related to the current state, a Glue42 starting context in the form of a [`glue_payload`](#structs-gluepayload) value and an optional callback cookie as arguments.

```cpp
// Callback for handling Glue42 connection status events.
void connection_status_callback (glue_state state, const char* message, const glue_payload* glue_payload, COOKIE cookie) {

    std::cout << "Glue42 state: " << enum_as_int(state) << " Message: " << message << std::endl;

    if (state == glue_state::connected) {
        // Glue42 has been initialized successfully and is ready for use.
    }
}

// Initialize Glue42 in your app.
glue_init("my_cpp_app", &connection_status_callback);
```

## App Configuration

To add your C++ app (or any EXE app) to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you must create a JSON file with app configuration. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

The following is an example configuration for a C++ app:

```json
{
    "title": "My C++ App",
    "type": "exe",
    "name": "my-cpp-app",
    "icon": "https://example.com/cpp-logo.jpg",
    "details": {
        "path": "%GDDIR%/../Demos/CPPDemo/",
        "command": "MyCPPApp.exe",
        "parameters": " --mode=1"
    }
}
```

| Property | Description |
|----------|-------------|
| `"type"` | Must be `"exe"`. |
| `"path"` | The path to the app - relative or absolute. You can also use the `%GDDIR%` environment variable, which points to the [Glue42 Enterprise](https://glue42.com/enterprise/) installation folder. |
| `"command"` | The actual command to execute (the EXE file name). |
| `"parameters"` | Specifies command line arguments. |

## Glue42 C Exports Concepts

Once the Glue42 library has been initialized, your app has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [App Management](../../../../glue42-concepts/application-management/c-exports/index.html)
- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/c-exports/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/c-exports/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/c-exports/index.html)

The following sections explain concepts related to using the Glue42 C Exports library.

### Using Glue42 Values

The Glue42 C Exports library offers defined structures for the data that is sent or received via Glue42, as well as functions for reading these data structures. The results from [invoking Interop methods](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html#method_invocation), the [Interop streaming](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html#streaming) data, as well as the context values when using [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/c-exports/index.html) and [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/c-exports/index.html) are in the form of Glue42 data structures.

#### Glue42 Data Structures

- `glue_value`

The [`glue_value`](#structs_gluevalue) structure encapsulates all possible values that can be sent or received via Glue42:

```cpp
struct glue_value {
	union {
		bool b;
		int i;
		long long l;
		double d;
		const char* s;

		bool* bb;
		int* ii;
		long long* ll;
		double* dd;
		const char** ss;

		glue_arg* composite;
		glue_value* tuple;
	};

	glue_type type;
	int len;
};
```

- `glue_arg`

The [`glue_arg`](#structs-gluearg) structure represents a named [`glue_value`](#structs_gluevalue):

```cpp
struct glue_arg {
	const char* name;
	glue_value value;
};
```

- `glue_payload`

The [`glue_payload`](#structs-gluepayload) structure represents data payloads received when using [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html) methods and streams, [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/c-exports/index.html) and [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/c-exports/index.html). Contains a reader to help consume the data, the origin of the data payload, a status indicating the state of the method invocation (when the payload is a result returned from [Interop method invocation](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html#method_invocation)), a [`glue_arg`](#structs-gluearg) array holding the data payload and the length of the data payload array:

```cpp
struct glue_payload {
	const void* reader;
	const char* origin;
	int status;
	const glue_arg* args;
	int args_len;
};
```

#### Creating Glue42 Values

Use the Glue42 methods prefixed with `glv_` to create [`glue_value`](#structs_gluevalue) values:

```cpp
// Signatures of the `glv_` methods for creating Glue42 values.

// Creates a Glue42 `bool` value.
glv_b(bool b) -> glue_value;

// Creates a Glue42 `bool[]` value.
glv_bb(bool* bb, int len) -> glue_value;

// Creates a Glue42 `double` value.
glv_d(double d) -> glue_value;

// Creates a Glue42 `double[]` value.
glv_dd(double* dd, int len) -> glue_value;

// Creates a Glue42 `int` value.
glv_i(int i) -> glue_value;

// Creates a Glue42 `int[]` value.
glv_ii(int* ii, int len) -> glue_value;

// Creates a Glue42 `long long` value.
glv_l(long long l) -> glue_value;

// Creates a Glue42 `long long[]` value.
glv_ll(long long* ll, int len) -> glue_value;

// Creates a Glue42 `string` value.
glv_s(const char* s) -> glue_value;

// Creates a Glue42 `string[]` value.
glv_ss(const char** ss, int len) -> glue_value;

// Creates a Glue42 `long long` value representing a date in milliseconds since epoch.
glv_dt(long long l) -> glue_value;

// Creates a Glue42 `long long[]` value representing an array of dates in milliseconds since epoch.
glv_dts(long long* ll, int len) -> glue_value;

// Creates a Glue42 `glue_arg` value (composite) representing an object.
glv_comp(glue_arg* composite, int len) -> glue_value;

// Creates a Glue42 `glue_arg` value (composite) representing
// a map of objects (an interpretational difference with non-array commposites).
glv_comps(glue_arg* composite, int len) -> glue_value;

// Creates a Glue42 `glue_value` representing a tuple that may be heterogeneous.
glv_tuple(glue_value* tuple, int len) -> glue_value;
```

The following example demonstrates some of the Glue42 methods for creating [`glue_value`](#structs_gluevalue) values:

```cpp
new glue_value[] {
    glv_i(42),
    glv_b(true),
    glv_d(3.14),
    glv_s("Glue42"),
    glv_ii([42, 4242]),
    glv_tuple(["glue", 42, 42.42])
};
```

Use the Glue42 methods prefixed with `glarg_` to create named [`glue_value`](#structs_gluevalue) values:

```cpp
// Signatures of the `glarg_` methods for creating named Glue42 values.

// Creates a `bool` value.
glarg_b(const char* name, bool b) -> glue_arg;

// Creates a Glue42 named `bool[]` value.
glarg_bb(const char* name, bool* bb, int len) -> glue_arg;

// Creates a Glue42 named `double` value.
glarg_d(const char* name, double d) -> glue_arg;

// Creates a Glue42 named `double[]` value.
glarg_dd(const char* name, double* dd, int len) -> glue_arg;

// Creates a Glue42 named `int` value.
glarg_i(const char* name, int i) -> glue_arg;

// Creates a Glue42 named `int[]` value.
glarg_ii(const char* name, int* ii, int len) -> glue_arg;

// Creates a Glue42 named `long long` value.
glarg_l(const char* name, long long l) -> glue_arg;

// Creates a Glue42 named `long long[]` value.
glarg_ll(const char* name, long long* ll, int len) -> glue_arg;

// Creates a Glue42 named `string` value.
glarg_s(const char* name, const char* s) -> glue_arg;

// Creates a Glue42 named `string[]` value.
glarg_ss(const char* name, const char** ss, int len) -> glue_arg;

// Creates a Glue42 named `long long` value representing a date in milliseconds since epoch.
glarg_dt(const char* name, long long l) -> glue_arg;

// Creates a Glue42 named `long long[]` value representing an array of dates in milliseconds since epoch.
glarg_dts(const char* name, long long* ll, int len) -> glue_arg;

// Creates a Glue42 named `glue_arg` value (composite) representing an object.
glarg_comp(const char* name, glue_arg* composite, int len) -> glue_arg;

// Creates a Glue42 named `glue_arg` value (composite) representing
// a map of objects (an interpretational difference with non-array commposites).
glarg_comps(const char* name, glue_arg* composite, int len) -> glue_arg;

// Creates a Glue42 named `glue_value` representing a tuple that may be heterogeneous.
glarg_tuple(const char* name, glue_value* tuple, int len) -> glue_arg;
```

The following example demonstrates some of the Glue42 methods for creating named [`glue_value`](#structs_gluevalue) values:

```cpp
// An array of `glue_arg` values.
glue_arg result[] = {
    glarg_i("glue", 42),
    glarg_s("glue", "42"),
    // A structured composite value.
    glarg_comp("sender", new glue_arg[] {
        glarg_comp("contact", new glue_arg[] {
            glarg_s("name", "Support")
        }, 1),
        glarg_s("email", "support@example.com")
    }, 2),
    glarg_dt("date", 1645551286008l),
    // A named heterogeneous tuple.
    glarg_tuple("Glue42", new glue_value[]{ glv_i(42), glv_b(true), glv_d(42.42) }, 3)
};
```

#### Reading Glue42 Values

Use the Glue42 methods prefixed with `glue_read_` to read [`glue_value`](#structs_gluevalue) values. Each method accepts a reader returned by [`glue_payload`](#structs-gluepayload) or by [`glue_read_context_sync()`](#functions-gluereadcontextsync) as a first parameter and a dot-separated field path from which to read.

| Method | Description |
|--------|-------------|
| [`glue_read_b()`](#functions-gluereadb) | Reads a Glue42 `bool` value. |
| [`glue_read_d()`](#functions-gluereadd) | Reads a Glue42 `double` value. |
| [`glue_read_i()`](#functions-gluereadi) | Reads an Glue42 `int` value. |
| [`glue_read_l()`](#functions-gluereadl) | Reads a Glue42 `long long` value. |
| [`glue_read_s()`](#functions-gluereads) | Reads a Glue42 `const char*` value. |
| [`glue_read_json()`](#functions-gluereadjson) | Reads a JSON value. |
| [`glue_read_glue_value()`](#functions-gluereadgluevalue) | Reads a [`glue_value`](#structs_gluevalue) value. |

*Note that all methods return the read value type and the `glue_read_json()` method returns a string encoded as a JSON - e.g., `"{x: 42, y: {a: 42.42, b: \"glue\"}}"`.*

#### Traversing Glue42 Payloads

The following example demonstrates how to traverse Glue42 value structures and extract data:

```cpp
void handle_payload(const char* endpoint, COOKIE cookie, const glue_payload* payload) {

    // Traversing a `glue_value` structure.
	for (int i = 0; i < payload->args_len; i++) {

		const auto glue_val = payload->args[i].value;

        if (glue_val.len < 0) {
            // Non-array Glue42 value.
            std::cout << "Non-array Glue42 value: ";

			switch (glue_val.type) {
                case glue_type::glue_bool:
                    std::cout << glue_val.b << std::endl;
                    break;
                case glue_type::glue_int:
                    std::cout << glue_val.i << std::endl;
                    break;
                case glue_type::glue_long:
                    std::cout << glue_val.l << std::endl;
                    break;
                case glue_type::glue_double:
                    std::cout << glue_val.d << std::endl;
                    break;
                case glue_type::glue_string:
                    std::cout << glue_val.s << std::endl;
                    break;
                // Milliseconds since epoch.
                case glue_type::glue_datetime:
                    std::cout << glue_val.l << std::endl;
                    break;
                // Represents an object - a `glue_arg` value (named `glue_value`).
                // Traverse recursively.
                case glue_type::glue_composite:
                    std::cout << glue_val.composite << std::endl;
                    break;
                default:
                    break;
			}
		}
		else {
            // Array Glue42 value.
            std::cout << "Array Glue42 value length: " << glue_val.len << " Value: ";

			switch (glue_val.type) {
                case glue_type::glue_bool:
                    std::cout << glue_val.bb << std::endl;
                    break;
                case glue_type::glue_int:
                    std::cout << glue_val.ii << std::endl;
                    break;
                case glue_type::glue_long:
                    std::cout << glue_val.ll << std::endl;
                    break;
                case glue_type::glue_double:
                    std::cout << glue_val.dd << std::endl;
                    break;
                case glue_type::glue_string:
                    std::cout << glue_val.ss << std::endl;
                    break;
                case glue_type::glue_datetime:
                    std::cout << glue_val.ll << std::endl;
                    break;
                // Represents an array of `glue_value` (can be non-homogeneous).
                // Traverse recursively.
                case glue_type::glue_tuple:
                    std::cout << glue_val.tuple << std::endl;
                    break;
                // Represents a map of objects (an interpretational difference with non-array commposites).
                // Traverse recursively.
                case glue_type::glue_composite_array:
                    std::cout << glue_val.composite << std::endl;
                    break;
                default:
                    break;
			}
		}
	}
}
```

### Destroying Resources

To destroy Glue42 resources after they have been utilized, use the [`glue_destroy_resource()`](#functions-gluedestroyresource) method. The following example demonstrates how to destroy a writer resource after it has been used to update a field in a [Channel](../../../../glue42-concepts/data-sharing-between-apps/channels/c-exports/index.html) context:

```cpp
const auto writer = glue_get_context_writer("MyContext", "data.glue");

glue_push_json_payload(writer, "{a: { b: 42.42, c: [42, 42.42], d: \"Glue42\"}}");

// Destroying the Glue42 resource.
glue_destroy_resource(writer);
```

## C Exports Reference

This reference describes the exported Glue42 C functions, the available Glue42 data structures, enumerations and types of callback functions.

## Structs

### glue_arg

Represents a named [`glue_value`](#structs-gluevalue). Use the Glue42 methods prefixed with `glarg_` to create named Glue42 values.

*Signature:*

```cpp
struct glue_arg
{
	const char* name;
	glue_value value;
};
```

*See also:*

- [Creating Glue42 Values](#glue42_c_exports_concepts-using_glue42_values-creating_glue42_values) - how to use `glv_` and `glarg_` methods to create Glue42 values.

### glue_payload

Represents data payload received when using [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html) methods and streams, [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/c-exports/index.html), [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/c-exports/index.html). Contains the data origin point and a reader to facilitate consuming the data. Use the Glue42 methods prefixed with `glue_read_` to read the data.

*Signature:*

```cpp
struct glue_payload
{
	const void* reader;
	const char* origin;
	int status;
	const glue_arg* args;
	int args_len;
};
```

*See also:*

- [Using Glue42 Values](#glue42_c_exports_concepts-using_glue42_values) - how to handle Glue42 values when using [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/c-exports/index.html) methods and streams, [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/c-exports/index.html), [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/c-exports/index.html).
- [`glue_arg`](#structs-gluearg) - data structure representing named Glue42 values.

### glue_value

Encapsulates all possible values that can be sent or received via Glue42. Use the Glue42 methods prefixed with `glv_` to create Glue42 values.

*Signature:*

```cpp
struct glue_value
{
	union
	{
		bool b;
		int i;
		long long l;
		double d;
		const char* s;

		bool* bb;
		int* ii;
		long long* ll;
		double* dd;
		const char** ss;

		glue_arg* composite;
		glue_value* tuple;
	};

	glue_type type;
	int len;
};
```

*See also:*

- [Creating Glue42 Values](#glue42_c_exports_concepts-using_glue42_values-creating_glue42_values) - how to use `glv_` and `glarg_` methods to create Glue42 values.
- [`glue_arg`](#structs-gluearg) - data structure representing named Glue42 values.s
- [`glue_type`](#enums-gluetype) - the available Glue42 value types.

## Enums

### glue_app_command

Represents the available commands passed to [`app_callback_function`](#types-appcallbackfunction).

*Signature:*

```cpp
enum class glue_app_command {
    create,
    init,
    save,
    shutdown
};
```

| Command | Description |
|---------|-------------|
| `create` | The app has been created. |
| `init` | The app has been initialized. |
| `save` | The app state has been saved. |
| `shutdown` | The app has been shut down. |

### glue_state

Represents the state of the connection to Glue42.

*Signature:*

```cpp
enum class glue_state {
    none,
    connecting,
    connected,
    initialized,
    disconnected
};
```

| State | Description |
|-------|-------------|
| `connected` | Connected to Glue42. |
| `connecting` | Connecting to Glue42. |
| `disconnected` | Disconnected from Glue42. |
| `initialized` | The Glue42 library has been initialized. |
| `none` | Information about the connection to Glue42 is not yet available. |

### glue_type

Represents the available Glue42 value types.

*Signature:*

```cpp
enum class glue_type {
    glue_none,
    glue_bool,
    glue_int,
    glue_long,
    glue_double,
    glue_string,
    glue_datetime,
    glue_tuple,
    glue_composite,
    glue_composite_array
};
```

*Note that `glue_composite_array` is a map of composite objects.*

### glue_window_command

Represents the available commands passed to [`glue_window_callback_function`](#types-gluewindowcallbackfunction).

```cpp
enum class glue_window_command {
    init,
    channel_switch,
    data_update
};
```

| Command | Description |
|---------|-------------|
| `channel_switch` | The window Channel has been changed. |
| `data_update` | The data of the Channel to which the window is joined has been updated. |
| `init` | The window has been initialized. |

## Types

### app_callback_function

Callback function for handling events related to app instances.

*Signature:*

```cpp
typedef void (*app_callback_function)(
    glue_app_command command,
    const void* callback,
    const glue_payload* payload,
    COOKIE
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `command` | [`glue_app_command`](#enums-glueappcommand) | Use the command to determine the type of the app instance event. |
| `callback` | `const void*` | Correlational callback that can be passed to other Glue42 methods - e.g., to [`glue_app_announce_instance()`](#functions-glueappannounceinstance) when announcing the newly created app instance, or to [`glue_push_payload()`](#functions-gluepushpayload) when saving the app state. |
| `payload` | `const glue_payload` | A [`glue_payload`](#structs-gluepayload) value that can be used for initial or restored state of the app. |
| `cookie` | `COOKIE` | Optional callback cookie. |

### context_function

Callback function for handling shared context updates when subscribing to or reading a shared contexts.

*Signature:*

```cpp
typedef void (*context_function)(
    const char* context_name,
    const char* field_path,
    const glue_value*,
    COOKIE
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context_name` | `const char*` | Name of the shared context. |
| `field_path` | `const char*` | Dot-separated field path in the shared context from where the data has been read or a path to the specific field in the shared context for which you have subscribed. |
| `glue_value` | `const glue_value*` | A [`glue_value`](#structs-gluevalue) value representing the shared context data. |
| `cookie` | `COOKIE` | Optional callback cookie. |

### glue_endpoint_status_callback_function

Callback function for handling status changes of Interop endpoints - e.g., when an Interop method has been registered or unregistered.

*Signature:*

```cpp
typedef void (*glue_endpoint_status_callback_function)(
    const char* endpoint_name,
    const char* origin,
    bool state,
    COOKIE
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | Name of the Interop method. |
| `origin` | `const char*` | ID of the Interop server that has registered or unregistered the Interop method. |
| `state` | `bool` | If `true`, the Interop method has been registered. If `false`, the Interop method has been unregistered. |
| `cookie` | `COOKIE` | Optional callback cookie. |

### glue_init_callback_function

Callback function for handling status changes when initializing Glue42.

*Signature:*

```cpp
typedef void (*glue_init_callback_function)(
    glue_state state,
    const char* message,
    const glue_payload*,
    COOKIE
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `state` | [`glue_state`](#enums-gluestate) | Status of the Glue42 connection. |
| `message` | `const char*` | Message related to the Glue42 status. |
| `glue_payload` | `const glue_payload*` | A [`glue_payload`](#structs-gluepayload) value representing the starting context. |
| `cookie` | `COOKIE` | Optional callback cookie. |

### glue_window_callback_function

Callback function for handling events related to registered [Glue42 Windows](../../../../glue42-concepts/windows/window-management/c-exports/index.html).

*Signature:*

```cpp
typedef void (*glue_window_callback_function)(
    glue_window_command command,
    const char* context_name,
    COOKIE
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `command` | [`glue_window_command`](#enums-gluewindowcommand) | Use the command to determine the type of the Glue42 Window event. |
| `context_name` | `const char*` | Name of the Channel context to which the window is joined. |
| `cookie` | `COOKIE` | Optional callback cookie. |

### invocation_callback_function

Callback function for handling Interop method invocations.

*Signature:*

```cpp
typedef void (*invocation_callback_function)(
    const char* endpoint_name,
    COOKIE,
    const glue_payload* payload,
    const void* endpoint
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | The name of the registered Interop method. |
| `cookie` | `COOKIE` | Optional callback cookie. |
| `payload` | `const glue_payload*` | A [`glue_payload`](#structs-gluepayload) value holding the invocation arguments. |
| `endpoint` | `const void*` | Correlational endpoint that can be used to send the invocation result (e.g., using the [`glue_push_payload()`](#functions-gluepushpayload) or [`glue_push_failure()`](#functions-gluepushfailure) methods). |

### multiple_payloads_function

Callback function for handling payloads received from invoking an Interop method on multiple Interop targets.

*Signature:*

```cpp
typedef void (*multiple_payloads_function)(
    const char* origin,
    COOKIE,
    const glue_payload* payloads,
    int len
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `origin` | `const char*` | The name of the registered Interop method. |
| `cookie` | `COOKIE` | Optional callback cookie. |
| `payloads` | `const glue_payload*` | A [`glue_payload`](#structs-gluepayload) value holding the multiple invocation results. |
| `len` | `int` | The number of the Interop targets from which results have been received. |

### payload_function

Callback function for handling the payload received from an Interop target (method invocation or stream subscription).

*Signature:*

```cpp
typedef void (*payload_function)(
    const char* origin,
    COOKIE,
    const glue_payload* payload
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `origin` | `const char*` | The name of the registered Interop method or stream. |
| `cookie` | `COOKIE` | Optional callback cookie. |
| `payload` | `const glue_payload*` | A [`glue_payload`](#structs-gluepayload) value holding the invocation result or stream data. |

### stream_callback_function

Callback function for handling Interop stream subscription requests. To accept the subscription, return `true`. To reject the subscription, return `false`.

*Signature:*

```cpp
typedef bool (*stream_callback_function)(
    const char* endpoint_name,
    COOKIE,
    const glue_payload* payload,
    const char*& branch
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | The name of the registered Interop stream. |
| `cookie` | `COOKIE` | Optional callback cookie. |
| `payload` | `const glue_payload*` | A [`glue_payload`](#structs-gluepayload) value holding the arguments of the subscription request. |
| `branch` | `const char*&` | The stream branch to which the subscriber is to be assigned. To assign a subscriber to the main branch, use `nullptr`. |

## Functions

### glue_app_announce_instance

Announces the successful creation of an app.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_app_announce_instance(
    const void* app_factory_request,
    HWND hwnd,
    app_callback_function app_callback,
    glue_window_callback_function window_callback,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `app_factory_request` | `const void*` | The correlational callback from the [`app_callback_function`](#types-appcallbackfunction) passed to the [`glue_app_register_factory()`](#functions-glueappregisterfactory) method when registering the app factory. |
| `hwnd` | `HWND` | The window handle. |
| `app_callback` | [`app_callback_function`](#types-appcallbackfunction) | Callback function for handling app instance events. |
| `window_callback` | [`glue_window_callback_function`](#types-gluewindowcallbackfunction) | Callback function for handling Glue42 Window events. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if successful.

### glue_app_register_factory

Registers an app factory.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_app_register_factory(
    const char* app_factory,
    app_callback_function callback,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `app_factory` | `const char*` | The name of the app. |
| `callback` | [`app_callback_function`](#types-appcallbackfunction) | Callback function for handling app instance events. Will be called when an instance of the app is created. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* Reference to the app factory. Call [`glue_destroy_resource()`](#functions-gluedestroyresource) to unregister the app factory.

### glue_destroy_resource

Destroys Glue42 subscriptions, streams, branches, methods, readers, writers and releases any memory obtained by them.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_destroy_resource(const void* resource);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `resource` | `const void*` | The resource to destroy. |

*Return value:* `0` if the resource is successfully destroyed.

*See also:*

- [Destroying Resources](#glue42_c_exports_concepts-destroying_resources) - how to destroy Glue42 resources after they have been utilized.

### glue_get_context_writer

Gets a writer for a Glue42 Channel context or a shared context and a field path. Use the Glue42 methods prefixed with `glue_push_` to write.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_get_context_writer(
    const char* context,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | `const char*` | The name of the the shared or Channel context. |
| `field_path` | `const char*` | Dot-separated field path for the writer designating where the data will be written. |

*Return value:* Reference to the writer for the specified field path.

### glue_get_starting_context_reader

Gets a reader for the Glue42 starting context.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_get_starting_context_reader();
```

*Parameters:* None

*Return value:* Reference to the reader for the Glue42 starting context. Use the Glue42 methods prefixed with `glue_read_` to read.

### glue_init

Initializes Glue42 asynchronously with the specified app name and a callback for handling Glue42 status changes.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_init(
    const char* app_name,
    glue_init_callback_function callback = nullptr,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `app_name` | `const char*` | The app name to be used when announcing to Glue42. |
| `callback` | [`glue_init_callback_function`](#types-glueinitcallbackfunction) | Callback function for handling Glue42 status updates. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if the parameters have been successfully validated.

### glue_invoke

Invokes an Interop method only on the best Interop server (usually the first that has registered the method with that name).

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_invoke(
    const char* endpoint_name,
    const glue_arg* args,
    int len,
    payload_function callback = nullptr,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | The name of the Interop method to invoke. |
| `args` | `const glue_arg*` | An array of [`glue_arg`](#structs-gluearg) values representing the arguments for the method invocation. |
| `len` | `int` | The length of the invocation arguments array. |
| `callback` | [`payload_function`](#types-payloadfunction) | Callback function for handling the result from the method invocation. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if the invocation is successful.

### glue_invoke_all

Invokes all Interop methods with the same name on all servers offering them.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_invoke_all(
    const char* endpoint_name,
    const glue_arg* args,
    int len,
    multiple_payloads_function callback = nullptr,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | The name of the Interop method to invoke. |
| `args` | `const glue_arg*` | An array of [`glue_arg`](#structs-gluearg) values representing the arguments for the method invocation. |
| `len` | `int` | The length of the invocation arguments array. |
| `callback` | [`multiple_payloads_function`](#types-multiplepayloadsfunction) | Callback function for handling the results from the method invocation on all Interop targets. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if the invocation is successful.

### glue_is_launched_by_gd

Checks whether this process has been launched by [**Glue42 Enterprise**](https://glue42.com/enterprise/) (e.g., starting an app or loading a Layout).

*Signature:*

```cpp
extern "C" GLUE_LIB_API bool __cdecl glue_is_launched_by_gd();
```

*Parameters:* None

*Return value:* `true` if the process has been launched by [**Glue42 Enterprise**](https://glue42.com/enterprise/), otherwise - `false`.

### glue_open_streaming_branch

Opens an already existing Interop stream branch by name (e.g., to push data to it).

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_open_streaming_branch(
    const void* stream,
    const char* branch
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | `const void*` | The registered Interop stream. |
| `branch` | `const char*` | The name of the Interop stream branch as specified in the [`stream_callback_function`](#types-streamcallbackfunction) for handling subscription requests. |

*Return value:* Reference to the stream branch. Use the Glue42 methods prefixed with `glue_push_` to push data to that specific branch.

### glue_push_failure

Pushes a failure message to an endpoint (e.g., failed invocation request, failed app factory registration).

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_push_failure(
    const void* endpoint,
    const char* message
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint` | `const void*` | Reference to the endpoint to which to push the message. The reference can be obtained from the corresponding invocation or registration call. |
| `message` | `const char*` | Message describing the failure. |

*Return value:* `0` if the error message is pushed successfully.

### glue_push_json_payload

Pushes a JSON payload to an endpoint (e.g., result from an invocation request, data from an Interop stream or a stream branch).

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_push_json_payload(
    const void* endpoint,
    const char* json
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint` | `const void*` | Reference to the endpoint to which to push the data. The reference can be obtained from the corresponding invocation or registration call. |
| `json` | `const char*` | Data to push as a correctly formatted JSON string - e.g., `"{x: 42, y: {a: 42.42, s: \"glue\"}}"`. |

*Return value:* `0` if the JSON string is parsed and pushed successfully.

### glue_push_payload

Pushes an array of [`glue_arg`](#structs-gluearg) values to an endpoint (e.g., result from an invocation request, data from an Interop stream or a stream branch).

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_push_payload(
    const void* endpoint,
    const glue_arg* args,
    int len
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint` | `const void*` | Reference to the endpoint to which to push the data. The reference can be obtained from the corresponding invocation or registration call. |
| `args` | `const glue_arg*` | An array of [`glue_arg`](#structs-gluearg) values representing the data to push. |
| `len` | `int` | The length of the [`glue_arg`](#structs-gluearg) array. |

*Return value:* `0` if the data is pushed successfully.

### glue_read_b

Reads a `bool` value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API bool __cdecl glue_read_b(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* the read `bool` value.

### glue_read_context

Opens a shared or Channel context asynchronously by name and calls back with a reader in the [`context_function`](#types-contextfunction) callback.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_read_context(
    const char* context,
    const char* field_path,
    context_function callback,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | `const char*` | The name of the shared or Channel context to read. |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the data. |
| `callback` | [`context_function`](#types-contextfunction) | Callback function for handling the context data. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if the data is read successfully.

### glue_read_context_sync

Gets a reader synchronously for a shared or Channel context by name.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_read_context_sync(const char* context);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | `const char*` | The name of the shared or Channel context to read. |

*Return value:* Reference to the reader for the specified shared or Channel context.

### glue_read_d

Reads a `double` value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API double __cdecl glue_read_d(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* the read `double` value.

### glue_read_glue_value

Reads a [`glue_value`](#structs-gluevalue) value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API glue_value __cdecl glue_read_glue_value(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* the read [`glue_value`](#structs-gluevalue) value.

### glue_read_i

Reads an `int` value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_read_i(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* the read `int` value.

### glue_read_json

Reads a JSON value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const char* __cdecl glue_read_json(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* string encoded as a JSON - e.g., `"{x: 42, y: {a: 42.42, s: \"glue\"}}"`.

### glue_read_l

Reads a `long long` value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API long long __cdecl glue_read_l(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* the read `long long` value.

### glue_read_s

Reads a `const char*` (string) value from a field path in a reader.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const char* __cdecl glue_read_s(
    const void* reader,
    const char* field_path
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `reader` | `const void*` | The reader obtained from a [`glue_payload`](#structs-gluepayload) or [`glue_read_context_sync()`](#functions-gluereadcontextsync). |
| `field_path` | `const char*` | Dot-separated field path for the reader designating from where to read the value. |

*Return value:* the read `const char*` value.

### glue_register_endpoint

Registers an Interop method.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_register_endpoint(
    const char* endpoint_name,
    invocation_callback_function callback,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | The name of the Interop method to register. |
| `callback` | [`invocation_callback_function`](#types-invocationcallbackfunction) | Callback function for handling method invocations. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if the method registration is successful.

### glue_register_main_window

Registers the main window of your app simultaneously as a Glue42 Window and as a Glue42 app in the Glue42 environment.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_register_main_window(
	HWND hwnd,
	app_callback_function app_callback,
	glue_window_callback_function window_callback,
	const char* title = nullptr,
	COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | `HWND` | The window handle. |
| `app_callback` | [`app_callback_function`](#types-appcallbackfunction) | Callback function for handling app instance events. |
| `window_callback` | [`glue_window_callback_function`](#types-gluewindowcallbackfunction) | Callback function for handling Glue42 Window updates. |
| `title` | `const char*` | Optional title for the Glue42 Window. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* None

### glue_register_streaming_endpoint

Registers an Interop stream.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_register_streaming_endpoint(
    const char* endpoint_name,
    stream_callback_function stream_callback = nullptr,
    invocation_callback_function invocation_callback = nullptr,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `endpoint_name` | `const char*` | The name of the Interop stream to register. |
| `stream_callback` | [`stream_callback_function`](#types-streamcallbackfunction) | Optional callback function for handling stream subscription requests and assigning subscribers to stream branches. To accept all subscription requests, use `nullptr`. |
| `invocation_callback` | [`invocation_callback_function`](#types-invocationcallbackfunction) | Optional callback function for handling method invocations in case the Interop stream is treated as a regular Interop method. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* Reference to the Interop stream. Use the Glue42 methods prefixed with `glue_push_` to push data to the stream.

### glue_register_window

Registers a window simultaneously as a Glue42 Window and as a Glue42 app in the Glue42 environment. Can be the main or a child window of your app.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_register_window(
    HWND hwnd,
    glue_window_callback_function callback = nullptr,
    const char* title = nullptr,
    COOKIE cookie = nullptr,
    bool startup = false
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `hwnd` | `HWND` | The window handle. |
| `callback` | [`glue_window_callback_function`](#types-gluewindowcallbackfunction) | Callback function for handling Glue42 Window updates. |
| `title` | `const char*` | Optional title for the Glue42 Window. |
| `cookie` | `COOKIE` | Optional callback cookie. |
| `startup` | `bool` | Set to `true` if the window you are registering is your main (startup) window. |

*Return value:* None

### glue_set_save_state

Sets a callback function for persisting any user-specific data for this app instance. Use the Glue42 methods prefixed with `glue_push_` in the [`invocation_callback_function`](#types-invocationcallbackfunction) to push the data to be saved.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_set_save_state(
    invocation_callback_function callback = nullptr,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `callback` | [`invocation_callback_function`](#types-invocationcallbackfunction) | Callback function for handling saving the app state. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* `0` if successful.

### glue_subscribe_context

Subscribes for updates of a shared or a Channel context.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_subscribe_context(
    const char* context,
    const char* field_path,
    context_function callback,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | `const char*` | The name of the shared or Channel context to which to subscribe. |
| `field_path` | `const char*` | Optional dot-separated field path designating to which specific context field to subscribe. |
| `callback` | [`context_function`](#types-contextfunction) | Callback function for handling the context updates. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* Reference to the context subscription. Call [`glue_destroy_resource()`](#functions-gluedestroyresource) to unsubscribe.

### glue_subscribe_endpoints_status

Subscribes for changes in the state of Interop endpoints - e.g., when an Interop method has been registered or unregistered.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_subscribe_endpoints_status(
    glue_endpoint_status_callback_function callback,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `callback` | [`glue_endpoint_status_callback_function`](#types-glueendpointstatuscallbackfunction) | Callback function for handling Interop endpoint status changes. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* Reference to the subscription. Call [`glue_destroy_resource()`](#functions-gluedestroyresource) to unsubscribe.

### glue_subscribe_single_stream

Subscribes to an Interop stream offered by the best Interop server (usually the first that has registered the stream with that name).

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_subscribe_single_stream(
    const char* stream,
    payload_function stream_callback,
    const glue_arg* args,
    int len,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | `const char*` | The name of the Interop stream to which to subscribe. |
| `stream_callback` | [`payload_function`](#types-payloadfunction) | Callback function for handling the data received from the Interop stream. |
| `args` | `const glue_arg*` | An array of [`glue_arg`](#structs-gluearg) values representing the arguments for the subscription request. |
| `len` | `int` | The length of the subscription request arguments array. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* Reference to the stream subscription. Call [`glue_destroy_resource()`](#functions-gluedestroyresource) to unsubscribe.

### glue_subscribe_stream

Subscribes to all Interop streams with the same name offered by different Interop servers.

*Signature:*

```cpp
extern "C" GLUE_LIB_API const void* __cdecl glue_subscribe_stream(
    const char* stream,
    payload_function stream_callback,
    const glue_arg* args,
    int len,
    COOKIE cookie = nullptr
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `stream` | `const char*` | The name of the Interop stream to which to subscribe. |
| `stream_callback` | [`payload_function`](#types-payloadfunction) | Callback function for handling the data received from the Interop streams. |
| `args` | `const glue_arg*` | An array of [`glue_arg`](#structs-gluearg) values representing the arguments for the subscription request. |
| `len` | `int` | The length of the subscription request arguments array. |
| `cookie` | `COOKIE` | Optional callback cookie. |

*Return value:* Reference to the stream subscription. Call [`glue_destroy_resource()`](#functions-gluedestroyresource) to unsubscribe.

### glue_write_context

Writes a value in a shared or a Channel context using a given field path.

*Signature:*

```cpp
extern "C" GLUE_LIB_API int __cdecl glue_write_context(
    const char* context,
    const char* field_path,
    glue_value value
);
```

*Parameters:*

| Name | Type | Description |
|------|------|-------------|
| `context` | `const char*` | The name of the shared or Channel context in which to write. |
| `field_path` | `const char*` | Dot-separated field path designating where to write the data. |
| `value` | [`glue_value`](#structs-gluevalue) | The value to write. |

*Return value:* `0` if the data is written successfully.