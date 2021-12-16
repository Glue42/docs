## Overview

The purpose of this tutorial is to introduce [**Glue42 Enterprise**](https://glue42.com/enterprise/) and some of its features to Java developers and to show the Glue42 way of developing applications. The tutorial starts with <!-- NUMBER OF APPS --> working independently from each other and when completed, the same applications will be integrated with each other and will interoperate as a unified user-friendly system.

## Prerequisites

- [**Glue42 Enterprise**](https://glue42.com/enterprise/)
- Experience with Java.
- Basic understanding of asynchronous programming.

## Resources

- See the Java examples distributed with [**Glue42 Enterprise**](https://glue42.com/enterprise/) and located in the `%LocalAppData%/Tick42/Demos/Java` folder.

- See how to [Glue42 Enable Your Java App](../../getting-started/how-to/glue42-enable-your-app/java/index.html#installation) - here you will find information on how to install, reference and initialize the Glue42 Java library, as well as how to create application configuration files for your Java apps.

- Each chapter of the tutorial provides a link to the documentation for the respective Glue42 Concept it explores.

- The reference documentation for the Glue42 Java APIs is distributed with [**Glue42 Enterprise**](https://glue42.com/enterprise/) and is located in the `Local/Tick42/GlueSDK/Glue42Java/docs/apidocs/` folder.

## Setup

Clone the [Java tutorial repository](<!-- Link to GitHub. The repo must be in the Glue42 organization and must be named "java-tutorial". -->) and start [**Glue42 Enterprise**](https://glue42.com/enterprise/).

Repository structure:
<!-- Repository structure - directories and files and short descriptions for each. -->

The tutorial contains the following applications:
<!-- List of applications and short description for each. -->

Here are some common issues you may encounter:

- An app is missing from the Glue42 Toolbar: check the [application configuration file](../../developers/configuration/application/index.html#application_configuration-exe), it must be a valid JSON.

- An app doesn't start after being clicked: make sure that the `"path"` property of the application configuration file points to the folder which contains the application executable file.

## Initializing the Glue42 Library

To use Glue42 Java library in your application, you must first initialize it.

<!-- Modify the initialization example as necessary. -->
```java
import com.tick42.glue.Glue; 

try (Glue glue = Glue.builder().build()) 
{
    System.out.println(glue.version()); 
}
```

<!-- TODO - Chapters with the available Glue42 Java functionalities. -->

## Congratulations!

You should now have the foundation to start working on Glue42 enabling your real applications.

Next step?

Build awesome Glue42 enabled apps!!