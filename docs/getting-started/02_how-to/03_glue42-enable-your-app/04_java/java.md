## Installation

The Glue42 Java library provides an easy way to make your enterprise Java applications Glue42 enabled. You can use the Glue42 Java library in the same way as any standard Java library.

The Glue42 Java library is available as a package in the [Maven Central Repository](https://search.maven.org/search?q=g:com.glue42) and as a part of the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer. It is highly recommended to use the shaded version of the library from the `java-glue42-shaded` artifact.

Glue42 Java requires JDK 8+ (Java SE 8+) and is JDK 9+ ready.

For Glue42 Java to work with Java SE 17+, you must pass the following parameters to the JVM:

```cmd
--add-opens java.desktop/java.awt=ALL-UNNAMED
--add-opens java.desktop/sun.awt.windows=ALL-UNNAMED
--add-exports java.desktop/java.awt.peer=ALL-UNNAMED
```

### Maven

```xml
<?xml version="1.0" encoding="utf-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.glue42</groupId>
            <artifactId>java-glue42-shaded</artifactId>
            <version>1.4.8</version>
        </dependency>
    </dependencies>
</project>
```

### Gradle

```groovy
apply plugin: 'java'

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile 'com.glue42:java-glue42-shaded:1.4.8'
}
```

## Referencing and Initialization

To use Glue42 Java in your applications, add the Glue42 Java library and its dependencies to your application classpath, import the `Glue` class and create a `Glue` instance:

```java
import com.tick42.glue.Glue;

try (Glue glue = Glue.builder().build())
{
    System.out.println(glue.version());
}
```

`Glue` is the main entry point of the SDK. It is thread-safe and you should create a single instance (per Glue42 application) and share it throughout your code.

Always close the `Glue` instance once you are done with it, in order to free up underlying resources (`ws` connections, thread pools, etc.). This example uses a `try-with-resources` block, because `Glue` extends `AutoCloseable` - in a real application, you will probably call `close()` or `closeAsync()` explicitly.

### Application Shutdown

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

When initializing Glue42, you can pass an event listener that will notify your application when it is about to be stopped. This is useful when your app/process isn't started directly by Glue42, but rather by Glue42 invoking a script/batch file or another application that in turn starts your application. With this listener you can properly shut down your application, free resources, etc.

Below is an example of passing a shutdown request listener when initializing Glue42:

```java
Glue.builder()
        .withShutdownRequestListener(glue -> {
            System.out.println("Starting shutdown procedure...");
            glue.closeAsync();
        })
        .build();
```

## App Configuration

To configure your Glue42 enabled Java app and add it to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you can use CONF, PROPERTIES, JSON files and system properties to externalize your configuration. For example, if you want to specify a name for your application, you can do so in a `glue.conf` file:

```java
glue {
    application: "My Java App"
}
```

Glue42 Java will look for a Glue42 configuration file in the application classpath. If you are using Maven or Gradle as a build tool, you can place the `glue.conf` file for your application in the `\src\main\resources` folder of your project directory.

If you don't specify an application name in a `glue.conf` file, as in the example above, the name of the application will be taken from the [**Glue42 Enterprise**](https://glue42.com/enterprise/) starting context which contains application configurations from JSON files.

You can also set the application name runtime when initializing the Glue42 Java library, which will override any previous configurations:

```java
Glue.builder().withApplicationName("My Java App").build();
```

JSON application configuration files must be placed in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

The following is an example JSON configuration for a Java app:

``` json
[
    {
        "title": "Java Example",
        "type": "exe",
        "name": "java-example",
        "icon": "https://enterprise-demos.tick42.com/resources/icons/w2.jpg",
        "details": {
            "path": "",
            "command": "java",
            "parameters": "-jar example.jar",
            "mode": "tab"
        }
    }
]
```

*Note that if you want to pass parameters to the JVM - e.g., to ensure compatibility of Glue42 Java with Java SE 17+ (see [Installation](#installation)), you must pass them before the command for running the executable JAR file - e.g., before `"-jar example.jar"`.*

| Property | Description |
|----------|-------------|
| `"type"` | Must be `"exe"`. |
| `"path"` | The path to the application - relative or absolute. |
| `"command"` | The actual command to execute (`java`). |
| `"parameters"` | Specifies command line arguments. |

*To be able to start Glue42 Java on a dual core machine, you have to pass the `-Dglue.gateway.ws.max-pool-size=3` parameter to the JVM by adding it to the `"parameters"` property described above.*

*For more details, see the [Application Configuration](../../../../developers/configuration/application/index.html#app_configuration-exe) section.*

## Glue42 Java Concepts

Once you have created a `Glue` instance, your application has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [App Management](../../../../glue42-concepts/application-management/java/index.html)
- [Intents](../../../../glue42-concepts/intents/java/index.html)
- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/java/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/java/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/java/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/java/index.html)
- [Layouts](../../../../glue42-concepts/windows/layouts/java/index.html)
- [Notifications](../../../../glue42-concepts/notifications/java/index.html)