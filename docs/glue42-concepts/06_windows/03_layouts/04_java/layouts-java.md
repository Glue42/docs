## Global Layouts

Global saving and restoring is an operation in which all apps running on a user's desktop are saved to a named Layout which can later be restored.

### Saving a Global Layout

The Layouts API is accessible through `glue.layouts()`.

To save a global Layout, use the `save()` method and specify a name for the Layout. Note that if a Layout with the same name already exists, it will be replaced. If a name isn't specified, a random name will be generated:

```java
CompletionStage<Void> layout = glue.layouts().save(options -> options
        .withName("Name of Layout")
        .withType(LayoutType.GLOBAL)
);
```

### Restoring a Global Layout

To restore a global Layout, use the `restore()` method:

```java
LayoutRestoreOptions restoreOptions = LayoutRestoreOptions.builder("LayoutToRestore").build();
CompletionStage<Void> restore = glue.layouts().restore(restoreOptions);
```

## Managing Layouts

### Listing Layouts

To get a collection of all Layouts, use the `list()` method:

```java
Collection<Layout> layouts = glue.layouts().list();
```

### Exporting Layouts

To export all Layouts for the current user, use the `exportLayouts()` method:

```java
CompletionStage<Collection<Layout>> exportedLayouts = glue.layouts().exportLayouts();
```

### Importing Layouts

You can import collections of Layouts by either merging them with the existing ones, or replacing the existing ones. Use the `importLayouts()` method and pass a collection of Layouts and import mode (`MERGE` or `REPLACE`):

```java
CompletionStage<Void> importLayouts = glue.layouts().importLayouts(Collections.emptyList(), LayoutImportMode.REPLACE);
```

### Removing Layouts

To remove a Layout, use the `delete()` method and specify the type and name of the Layout to remove. The method returns a `CompletionStage<Void>` which completes when the Layout has been deleted:

```java
glue.layouts().delete(LayoutType.GLOBAL, "Name of Layout");
```

## Saving Custom Data

Apps can store custom data in a saved Layout. When the Layout is restored, the custom data is also restored and returned to the apps. Currently, the custom data can only be the window context. When the Layout is restored, the context of the window in the Layout will also be restored if it has been saved previously.

*Note that saving large volumes of custom data as window context (e.g., thousands of lines of table data) can lead to significant delays when saving a Layout. A Layout usually contains several (in some cases - many) apps and/or Workspaces (which can also contain many apps) and if one or more of the apps saves large amounts of context data each time a Layout is saved, this will significantly slow down the saving process. The methods for saving custom context work best with smaller amounts of data. If your app needs to save large amounts of data, you have to think about how to design this process better - for instance, you may store IDs, indices, etc., as context data, save the actual data to a database and when you restore the Layout, fetch the data using the data IDs saved as window context.*

To save custom data, apps can subscribe for Layout save requests using the `addSaveListener()` method. The `LayoutSaveHandler` passed as an argument will be invoked when a Layout save is requested. The handler receives a `LayoutSaveRequest` object as an argument from which the Layout name and type can be extracted. It must return a `Map<String, Object>` containing pairs of context property names and their values:

```java
Map<String, Object> context = new HashMap<>();
context.put("gridWidth", 420);
context.put("gridHeight", 42);

glue.layouts().addSaveListener((request) -> context);
```