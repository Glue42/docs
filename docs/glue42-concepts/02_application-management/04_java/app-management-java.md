## Listing Applications

To list all applications available to the current user, use the `applications()` method:

```java
glue.appManager().applications();
```

The `applications()` method returns a `Map<String, ApplicationInstance>` result containing the available application instances keyed by application name.

## Starting Applications

To start an application, use the `start()` method:

```java
glue.appManager().start("clientlist")
        .whenComplete((instance, error) -> {
            if (error != null) {
                // application failed to start
            }
        });
```

You can also pass a context object (an application-specific object that will be available in the new app) or override any of the pre-configured window settings:

```java
glue.appManager().start("clientcontact", Collections.singletonMap("selectedUser", 2));
```

## Listing Running Instances

To list all running instances of an application, use the `instances()` method:

```java
glue.appManager().instances();
```

The `instances()` method returns a `Collection<ApplicationInstance>` result containing all running application instances.

## Stopping Instances

To stop a running instance, use the `close()` or `closeAsync()` method:

```java
instance.closeAsync();
```

## Multi Window Apps

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

Glue42 Java offers support for applications consisting of multiple windows. Each child window of an application can be registered in the context of a child Glue42 Application that you can save and restore in a [Layout](../../windows/layouts/java/index.html), start directly from the Glue42 Toolbar, etc.

The following example demonstrates how to register an `ApplicationInstanceHandler` using the `registerInstanceHandler()` method. It will be invoked when a child window of the specified application is started. The handler in the example registers the child window as a Glue42 Window (for more details, see [Window Management](../../windows/window-management/java/index.html)) using a `WindowRegistration` builder. When the child window has been registered, it starts to listen for context updates using its `onContextUpdated()` method. Finally, when a [Layout](../../windows/layouts/java/index.html) save is requested, the child window will save its current context using the `addSaveListener()` method in order to be able to retrieve it later when the Layout is restored:

```java
glue.appManager().registerInstanceHandler("my-child-window", applicationInstance -> {

    glue.windows().register(
            WindowRegistration.builder(glue.windows().getWindowHandle(childFrame))
                    .withInstanceId(applicationInstance.getId())
                    .build()
    ).thenAccept(window ->
            window.onContextUpdated(e ->
                selector.setSelectedIndex(selectedIndex);
            {
                int selectedIndex = (Integer) e.getContext().getOrDefault("SelectedIndex", -1);
            }));
    glue.layouts().addSaveListener(applicationInstance.getId(), request ->
            Collections.singletonMap("SelectedIndex", selector.getSelectedIndex()));
});
```