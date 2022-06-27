## Metric System

A metrics system groups one or more metrics. For example, a Hardware Metric System is a metric system that groups all metrics, which measure the hardware performance – CPU usage, HDD Disk usage etc.

Metrics can be added or removed from the system programmatically.

The metric system is a composite structure. One metric system may contain other metric systems - e.g., a Databases metric system can be defined to contain the *Oracle_1* and *MySql_2* metric sub-systems. In this example, the *Oracle_1* and *MySql_2* sub-systems are specific metric systems that measure the performance and the load of the database servers *Oracle_1* and *MySql_2*.

Every metric system has one system state metric. The purpose of this state metric is to provide information about the condition of the metric system. The state can be:
- `GREEN` - when the system is working properly;
- `AMBER` - when the system can't work properly because of a third-party component, but is trying to repair;
- `RED` - when the system can't continue working;

## Metric System Repository

The metric system repository is a holder for metric systems. The repository has one root metric system, which is created automatically when the
repository is created. If you add another metric system to the repository, then this metric system will be attached to the root metric system as a
child.

## Metric Repository Setup

When the `GlueFacade` is initialized, a metrics repository is automatically created for you. To access the repository, use:

```csharp
var metrics = glue.Metrics.MetricsRepository;
```

## Metric Systems Setup

When the metric repository is created, a root metric system is created by default. You should use this root metric system to create your own
metrics systems:

```csharp
IPublishingMetricSystem root = metrics.Root;

IPublishingMetricSystem databasesMetricSystem = root.AddChild("Databases", "The root metric system for all databases metric systems");

IPublishingMetricSystem oracleMetricSystem = databasesMetricSystem.AddChild("Oracle_1", "Measurements of Oracle server");

IPublishingMetricSystem mysqlMetricSystem = databasesMetricSystem.AddChild("MySql_2", "Measurements of MySql server");
```

## Accessing and Removing Metric Systems

You can always access any of the created metric system by their name:

```csharp
root.GetChild("Databases");
```

You can also remove a metrics system:

```csharp
databasesMetricSystem.RemoveChild(mysqlMetricSystem);
```

## Creating Metrics

Now that you have created metric systems, you can start defining metrics. Our database metrics systems will have the following metrics:

- address metric - database server name;
- count metric - number of all inserted records;

```csharp
IAddressMetric oracleServerAddress = oracleMetricSystem.GetOrCreateAddressMetric("databaseServer", new AddressMetricOptions().WithDescription("database server name"));

ICountMetric oracleInsertedRecords = oracleMetricSystem.GetOrCreateCountMetric("allInsertedRecords", new CountMetricOptions().WithDescription("number of all inserted records"));

IAddressMetric mySqlServerAddress = mysqlMetricSystem.GetOrCreateAddressMetric("databaseServer", new AddressMetricOptions().WithDescription("database server name"));

ICountMetric mySqlInsertedRecords = mysqlMetricSystem.GetOrCreateCountMetric("allInsertedRecords", new CountMetricOptions().WithDescription("number of all inserted records"));
```

## Accessing and Removing Metrics

You can access a metric by its name:

```csharp
mySqlServerAddress = mysqlMetricSystem.GetStatisticMetric("databaseServer");
```

And you can also remove the metric:

```csharp
mysqlMetricSystem.RemoveMetric(mySqlServerAddress);
```

## Setting Metric Values

The following code sets the values of the metrics. Note that the different metric types provide different methods for updating their values

```csharp
// sets the IP address of the Oracle database server
oracleServerAddress.SetValue("192.168.0.111", 1521);

// increase the metric whenever you insert a database record
oracleInsertedRecords.Increment();
```