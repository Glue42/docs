## Creating Metrics Sub-Systems

The Metrics API is accessible through the [`glue.metrics`](../../../reference/glue/latest/metrics/index.html) object. Metrics are organized in a hierarchical structure using [System](../../../reference/glue/latest/metrics/index.html#System) objects. Systems can have sub-systems and metrics. Аll metrics and metric sub-systems are created under a common `App` system, which sits under the root (`/`) system.

To create a sub-system, you need to have a reference to a system and call its [`subSystem()`](../../../reference/glue/latest/metrics/index.html#System-subSystem) method. Initially, there is one root system available.

The following example demonstrates how to create a sub-system by passing a name and description for it:

```javascript
const module = glue.metrics.subSystem("Module", "App Module");
```

You can create almost arbitrary in structure and in depth metrics systems:

```javascript
const module = glue.metrics.subSystem("A").subSystem("B"); 

// Or

const module = glue.metrics.subSystem("A/B/C");
```

## Setting System State

The value of creating sub-systems is that it allows you to independently group attributes of your system and to flag its state. To set the state, use the [`setState()`](../../../reference/glue/latest/metrics/index.html#System-setState) method which accepts a state number and a state description as arguments:

```javascript
// State RED, nothing works.
module.setState(100, "Disconnected from backend.");

// State GREEN, everything works.
module.setState(0, "Connected to backend.");

// State AMBER, some problems.
module.setState(50, "Backend operational but some endpoints aren't available.");
```

The state is a number between 0 and 100 and the description argument is a string explaining the state.

| Value | Color Code | Status |
|-------|------------|--------|
| `0` | GREEN | Everything works. |
| `50` | AMBER | Some features are operational, but some problems. |
| `100` | RED | Critical error. |

## Traversing Sub-Systems

The root system is [`glue.metrics`](../../../reference/glue/latest/metrics/index.html), but it can also be accessed via the [`root`](../../../reference/glue/latest/metrics/index.html#System-root) property of any system: 

```javascript
const root = module.root;
```

To find the parent of any sub-system, use the [`parent`](../../../reference/glue/latest/metrics/index.html#System-parent) property. To retrieve the child systems of a system, use the [`subSystems`](../../../reference/glue/latest/metrics/index.html#System-subSystems) property.

## Working with Metrics

### Metric Types

The Metrics library offers the following subsets of Glue42 Metrics:

| Type | Description |
|------|-------------|
| FAV metric | FAV ("Feature, Action, Value") metrics allow you to cover a wide range of scenarios for collecting and sorting metrics data by using the `Feature` and `Action` fields to describe the metric in any way suitable to your case, and the `Value` field to pass the value you are interested in. |
| `StringMetric` | Holds any string as a value. |
| `NumberMetric` | Holds a number as a value. |
| `TimestampMetric` | Holds any date/time value. |
| `ObjectMetric` | A set of key/value pairs that allows you to create metrics with non-scalar values. |

### Creating Metrics

The following examples demonstrate how to create different types of metrics. The methods for creating metrics are available at top level of the API, as well as on the [System](../../../reference/glue/latest/metrics/index.html#System) instance, except for the FAV metric method which is available only at top level.

To create a FAV metric, use the [`featureMetric()`](../../../reference/glue/latest/metrics/index.html#API-featureMetric) method:

```javascript
// The FAV metric method is available only at top level of the API.
glue.metrics.featureMetric("Info View", "Selected", "grid");
```

To create a string metric, use the [`stringMetric()`](../../../reference/glue/latest/metrics/index.html#API-stringMetric) method:

```javascript
// String metric created under a metric sub-system.
const stringMetric = module.stringMetric("Glue42 Version", glue.version);
```

To create a number metric, use the [`numberMetric()`](../../../reference/glue/latest/metrics/index.html#API-numberMetric) method:

```javascript
// Number metric.
const numberMetric = module.numberMetric("Failed Requests", 1);
```

To create a timestamp metric, use the [`timestampMetric()`](../../../reference/glue/latest/metrics/index.html#API-timestampMetric) method:

```javascript
// Timestamp metric.
const timestampMetric = module.timestampMetric("Time", Date.now());
```

To create an object metric, use the [`objectMetric()`](../../../reference/glue/latest/metrics/index.html#API-objectMetric) method:

```javascript
const error = {
    message: "Critical module error.",
    time: Date.now()
};

// Object metric.
const objectMetric = module.objectMetric("Last Error", error);
```

### Updating Metrics

Metric values are updated using the `update()` method of the metric instance and the [`NumberMetric`](../../../reference/glue/latest/metrics/index.html#NumberMetric) also has methods for incrementing and decrementing its value:

```javascript
// Updating string metrics.
await stringMetric.update(glue.version);

// Updating number metrics.
await numberMetric.update(2);
numberMetric.increment();
numberMetric.decrement();
numberMetric.incrementBy(1);
numberMetric.decrementBy(2);

// Updating timestamp metrics.
await timestampMetric.update(Date.now());

// Updating object metrics.
await objectMetric.udpate({ glue: 42 });
```

## Best Practices

Try to instrument your application as early as possible. Almost any web application performs a lot of AJAX requests, typically over REST calls to manipulate resources. Avoid simply using your favorite AJAX library function everywhere. Instead, you should create a wrapper function which calls it. 

Maintain a metrics system for each endpoint, set its state to 100 (RED) when the request fails, and back to 0 (GREEN) when the request succeeds. Then, under each system:

- use number metrics to count the number of requests, the number of failures and the number of successes;
- use have timestamp metrics to record min/max/average latency;
- use an object metric to record the last reason for failure (time of occurrence, message, and stack trace, if possible);

Wrapping the function also gives you the ability to mock responses for testing, show and hide progress widgets in a consistent way, etc.

If you have instrumented your application properly, you should be able to debug any problem without having to look at log files, even if the problem is with the backend.

Avoid leaking personal information (IDs are OK, names aren't, URLs should be fine).

## Reference

For a complete list of the available Metrics API methods and properties, see the [Metrics API Reference Documentation](../../../reference/glue/latest/metrics/index.html).