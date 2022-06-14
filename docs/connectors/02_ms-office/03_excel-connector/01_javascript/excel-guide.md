## Introduction

The Glue42 Excel Connector allows applications to use Excel as a local data editor. The application uploads tabular data into Excel, so that the user may view it and edit it in a familiar environment. Any changes can then be transmitted back to the application for validation, processing, auditing and/or long-term storage.

## Initialization

As shown in the [Set Up Your Application](../../set-up-your-application/javascript/index.html) section, you need to initialize the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) library and set the [`excel`](../../../../reference/glue4office/latest/excel/index.html) property of the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) configuration object to `true`:

```javascript
const config = {
    // ...,
    excel: true // enable Excel integration
}
```

After that, get a reference to the [Excel Connector API](../../../../reference/glue4office/latest/excel/index.html#ExcelAPI):

```javascript
Glue4Office(config)
    .then(g4o => {
        const excel = g4o.excel
        // interact with Excel
    })
    .catch(console.error)
```

## Tracking Excel Connector Status Changes

When [**Glue42 Enterprise**](https://glue42.com/enterprise/) is initialized, you can check whether Excel is running and the Glue42 Excel Connector is loaded:

```javascript
console.log(`Glue42 Excel Connector is ${excel.addinStatus ? "available" : "unavailable"}`);
```

You can use the [`onAddinStatusChanged()`](../../../../reference/glue4office/latest/excel/index.html#API-onAddinStatusChanged) method to track the availability of the Glue42 Excel Connector. You may find this useful if you need to track when to enable or disable certain elements of your app user interface.

```javascript
const unsubscribe = excel.onAddinStatusChanged(available => {
    console.log(`Glue42 Excel Connector is ${available ? "available" : "not available"}`)
});
```

The `available` argument passed to your callback will be `true` if and only if:

- Excel is running.
- The Glue42 Excel Connector is installed and enabled in Excel.
- The Glue42 Excel Connector and your app are using the same connectivity configuration and are connected to the same Glue42 Gateway.

In any other case the `available` flag will be false.

To stop listening for connection status changes, simply call the returned function:

```javascript
unsubscribe();
```

## Sending Data to Excel

To send a table to Excel, you need to call the [`openSheet()`](../../../../reference/glue4office/latest/excel/index.html#API-openSheet) method.

You must specify the shape of your app data, optionally pass the row data and certain [customization options](#customization_options). All of these need to be packaged in a single [`OpenSheetConfig`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetConfig) object, passed to the [`openSheet()`](../../../../reference/glue4office/latest/excel/index.html#API-openSheet) call.

The example below assumes your app is displaying a financial portfolio (e.g., a list of stocks a person owns). Here is how to push the data in Excel:

```javascript
const config = {
    columnConfig: [
        { header: "Symbol", fieldName: "symbol" },
        { header: "Quantity", fieldName: "quantity" }
    ],
    data: [
        { quantity: 100, symbol: "AAPL" },
        { quantity: 200, symbol: "GOOG" }
    ]
}

excel.openSheet(config)
    .then(sheet => console.log("Sent data to Excel"))
```

The example above will open a new Excel workbook, create an empty sheet and will create the following table and place it at position `A1` (row 1, column A):

|Symbol|Quantity|
|-------|---------|
|AAPL   |      100|
|GOOG   |      200|

The [`header`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig-header) in the [`ColumnConfig`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig) is optional and specifies the caption of the column in Excel. If omitted, the field name will be used.

If you don't need to send any data but just set up a table in Excel for the user to populate, then you can omit the [`data`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetConfig-data) property. You can still populate the table later using the [`update()`](../../../../reference/glue4office/latest/excel/index.html#Sheet-update) method on the [`Sheet`](../../../../reference/glue4office/latest/excel/index.html#Sheet) object.

Note that the `Promise` returned by the [`openSheet()`](../../../../reference/glue4office/latest/excel/index.html#API-openSheet) method resolves with a reference to a [`Sheet`](../../../../reference/glue4office/latest/excel/index.html#Sheet) object.

## Customization Options

By default, when your application sends data to Excel, the Glue42 Connector will create a new workbook, a new spreadsheet in the workbook, and place the unformatted data in the 1st row and column (A1).

There are certain customizations you can apply by specifying the [`options`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetConfig-options) property in the [`OpenSheetConfig`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetConfig) object.

### Column Customization

If you are planning to take updates from the user into your app, you shouldn't rely on the ordering of the rows when accepting the data because the user might have filtered or sorted the data before returning it to your application. This means that:

- all your rows must be keyed somehow, so you can track what has changed
- you must not accept changes to columns containing keys

You can also specify a background and/or a foreground color and a column width:

```javascript
const columns = [
    {
        header: "Symbol",
        fieldName: "symbol",
        width: 80
    }
    // ...
]
```

### Preventing Saving Temporary Workbooks

If your users need to message data to Excel, but aren't allowed to save it locally, and should instead return the data to be saved in your application, you can set the [`inhibitLocalSave`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-inhibitLocalSave) flag to `true` to prevent the users from saving temporary workbooks.

### Custom Workbook, Sheet and Range

If your app needs to create (or re-open and re-use) a specific workbook, or place data in a specific spreadsheet and location in the spreadsheet, you can use the [`workbook`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-workbook), [`worksheet`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-worksheet) and [`namedRange`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-namedRange) options:

```javascript
const config = {
    columnConfig,
    data,
    options: {
        workbook: "ClientData.xls",
        worksheet: "John Doe",
        namedRange: "B2",
        dataRangeName: "ClientData",
        clearGrid: true
    }
}
```

Note that all these settings are optional. If the workbook doesn't exist, it is going to be created, otherwise re-opened. If there is data in the specified spreadsheet, it's going to be wiped, unless you've set [`clearGrid`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-clearGrid) to `false`. In the example above the data will be placed in the `John Doe` spreadsheet, starting on the 2nd row and column (B2). You can use also use Row/Column references, e.g., `R2C2` (row 2 column 2). Specifying [`dataRangeName`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-dataRangeName) names the range of cells which starts at `B2` and spans your data to `ClientData`.

### Using Templates

Excel Templates are workbooks that can be used to create a framework that the Glue42 Excel Connector should use when displaying a new set of data to the user. When the application invokes a Glue42 Excel API method, it may request that the data be added to a copy of an existing workbook (the template) that has been formatted to present the data correctly for the user. The Glue42 Excel Connector will then make a copy of the workbook and paste the data into it, instead of using a new blank workbook.

If you want to send formatted data to Excel or you want to include more than just the data (e.g. headers, footers, charts and maps), you can use [`templateWorkbook`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-templateWorkbook) and [`templateSheet`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-templateWorksheet):

```javascript
const config = {
    columnConfig,
    data,
    options: {
        templateWorkbook: "ReportTemplate.xls",
        templateWorksheet: "Data with Chart"
    }
}
```

## Receiving Updates from Excel

Once you have obtained a reference to the opened spreadsheet, you can subscribe for and start tracking updates made by the user, receive and validate them in your application, using the [`onChanged()`](../../../../reference/glue4office/latest/excel/index.html#Sheet-onChanged) method on the [`Sheet`](../../../../reference/glue4office/latest/excel/index.html#Sheet) object:

```javascript
const config = {
    columnConfig: [
        { header: "Symbol", fieldName: "symbol" },
        { header: "Quantity", fieldName: "quantity" }
    ],
    data: [
        { quantity: 100, symbol: "AAPL" },
        { quantity: 200, symbol: "GOOG" },
    ],
    options: {
        // Configure Excel to trigger sheet change events when the user changes
        // one or more cells in a given row and then selects a cell in a different row.
        // By default, sheet change events will be triggered when the user clicks a predefined button in Excel.
        updateTrigger: ["row"]
    }
};

excel.openSheet(config)
    .then(sheet => {
        console.log("Sent data to Excel", sheet);

        sheet.onChanged((data, errorCallback, doneCallback, delta) => {
            console.log("data: ", data);
            console.log("delta: ", delta);

            doneCallback();
        });
    });
});
```

The function passed to [`onChanged()`](../../../../reference/glue4office/latest/excel/index.html#Sheet-onChanged) will be called with the data sent from Excel. The `data` parameter holds an array of objects where each object corresponds to a row in Excel. Each object will have a number of properties populated with data, where each property will correspond to the [`fieldName`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig-fieldName) property you have passed in the respective column definition when calling [`openSheet()`](../../../../reference/glue4office/latest/excel/index.html#API-openSheet).

You can optionally validate the data by calling the `errorCallback` function or accept the changes by calling the `doneCallback` function. Declarative and imperative validation is explained in details in the [Data Validation](#data_validation) section below.

The `delta` argument is a [`DeltaItem`](../../../../reference/glue4office/latest/excel/index.html#DeltaItem) object which holds the changes made to the row(s).

The [`onChanged()`](../../../../reference/glue4office/latest/excel/index.html#Sheet-onChanged) method call creates a subscription for changes done by the user. To unsubscribe, simply call the returned `unsubscribe()` function:

```javascript
const unsubscribe = sheet.onChanged(...)
unsubscribe()   // sheet no longer tracked for changes
```

## Data Validation

There are 2 types of validation you can perform before you accept data from Excel:

- preventing users from typing incompatible data (e.g., column is numeric but the user types in some text), which you can do using Declarative Validation;
- preventing users from breaking the integrity of your data (which can span all data), which you can do using Programmatic Validation;

### Declarative Validation

You can specify what kind of data the user is allowed to type in a given column using the [`validation`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig-validation) property from the [`ColumnConfig`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig):

```javascript
const quantityColumn = {
    header: "Quantity",
    fieldName: "quantity",
    // whole numbers only
    validation: {
        alert: "Stop",
        type: "WholeNumber"
    }
}
```

The possible [`alert`](../../../../reference/glue4office/latest/excel/index.html#Validation-alert) types are:

|Alert|Description|
|---|---|
|`Information`|Inform users that the data they entered is invalid, without preventing them from entering it. This type of error alert is the most flexible. When an Information alert message appears, users can click OK to accept the invalid value or Cancel to reject it.|
|`Stop`|Prevent users from entering invalid data in a cell. A Stop alert message has two options: Retry or Cancel.|
|`Warning`|Warn users that the data they entered is invalid, without preventing them from entering it. When a Warning alert message appears, users can click Yes to accept the invalid entry, No to edit the invalid entry, or Cancel to remove the invalid entry.|

The possible values for the [`type`](../../../../reference/glue4office/latest/excel/index.html#Validation-type) property are:

|Type|Description|
|---|---|
|`Date`|Only dates are allowed|
|`Time`|Only times are allowed|
|`Decimal`|Only decimal numbers are allowed. Once the decimal number option is selected, other options become available to further limit the input.|
|`TextLength`|Validates the input based on a number of characters or digits|
|`WholeNumber`|Only whole numbers are allowed. Once the whole number option is selected, other options become available to further limit input.|
|`List`|Only values from a predefined list are allowed. The values are presented to the user as a dropdown menu control.|

The [`list`](../../../../reference/glue4office/latest/excel/index.html#Validation-list) validation type requires you to pass one more property holding the allowed set of values:

```javascript
const stateColumn = {
    header: "State",
    fieldName: "state",
    validation: {
        alert: "Stop",
        type: "List",
        list: ["Arizona", "California", ...]
    }
}
```

When your application has set the [`validation`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig-validation) property and the [`alert`](../../../../reference/glue4office/latest/excel/index.html#Validation-alert) is `Stop`, Excel won't even attempt to return data back to your application if the user input is invalid.

### Programmatic Validation

When your app needs a more sophisticated validation than what the [`validation`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig-validation) property can offer, you can write code to completely control the validation of the data sent from Excel to your application.

Here is again how subscribing for user updates looks like:

```javascript
sheet.onChanged((data, errorCallback, doneCallback, delta) => {
	// add your custom validation here
})
```

When your app receives an update from Excel, you can signal Excel back by calling the `errorCallback` and let Excel know that there were errors in the user's input which need to be corrected in Excel before the data is accepted by your web application.

The `errorCallback` accepts a list of [validation errors](../../../../reference/glue4office/latest/excel/index.html#ValidationError), where a validation error specifies which cell (row and column) is in error and what the problem is. Here is an example of handling multiple validation errors:

```javascript
sheet.onChanged((data, errorCallback, doneCallback, delta) => {
    // ...
    const errors = data.reduce(
        (errors, rowData, rowIndex) => {
            if (!rowData["firstName"]) {
                errors.push({
                    row: rowIndex + 1,
                    column: 0,
                    description: "First name is mandatory"
                })
            }
            if (Number(rowData["subscriptionMonths"]) < 6) {
                errors.push({
                    row: rowIndex + 1,
                    column: 2,
                    description: "Subscription period must be at least 6 months",
                    text: "6"   // replacing what the user typed
                })
            }
        }, [])

    // if during the validation there are any errors accumulated
    // you need to call the errorCallback, otherwise the doneCallback
    if (errors.length > 0) {
        errorCallback(errors)
    }
    else {
        doneCallback()
    }
})
```

The [`column`](../../../../reference/glue4office/latest/excel/index.html#ValidationError-column) property in the validation error can either be a `Number` (the 0-based column index) or a `String` (the [`fieldName`](../../../../reference/glue4office/latest/excel/index.html#ColumnConfig-fieldName) specified in the column config).

You can also customize the way Excel displays the errors by setting the [`foregroundColor`](../../../../reference/glue4office/latest/excel/index.html#ValidationError-foregroundColor), [`backgroundColor`](../../../../reference/glue4office/latest/excel/index.html#ValidationError-backgroundColor) and some other properties of the [`validationError`](../../../../reference/glue4office/latest/excel/index.html#ValidationError) object.

### Controlling Update Frequency

If you are expecting updates from Excel, you can specify how often your app should get updates by setting a value for the [`updateTrigger`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-updateTrigger) property in the [`OpenSheetOptions`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions) object.

The possible values are:

|Trigger Type|Description|
|---|---|
|`button`|When the user clicks the Return Data button on the Glue42 ribbon|
|`row`|When the user changes one or more cells in a given row and then selects a cell in a different row|
|`save`|When the user tries to save the worksheet|

The `row` options is the most interactive, since your application gets updated as the user moves through the spreadsheet.

When using the `button` option, you can also customize the caption of the button and where it is placed by using the [`buttonText`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-buttonText) and [`buttonRange`](../../../../reference/glue4office/latest/excel/index.html#OpenSheetOptions-buttonRange) properties:

```javascript
const config = {
    columnConfig,
    data,
    {   // options follow
        updateTrigger: "Button",
        buttonText: "Send back data",
        buttonRange: "A1:C1"    // button spans 3 cells
    }
}

excel.openSheet(config).then(sheet => ...)
```

## Excel Scripting

Upon launching, the Glue42 Excel Connector registers the following streams and methods:

|Method|Description|
|------|-----------|
|T42.ExcelScript.Grid.AddRow|Method: Add Row to Sheet|
|T42.ExcelScript.Table.AddRow|Method: Add Row to Table|
|T42.ExcelScript.Workbook.Create|Method: Create Workbook|
|T42.ExcelScript.Workbook.Open|Method: Open Workbook/Get Sheets|
|T42.ExcelScript.Worksheet.Write|Method: Update/Insert Row in Grid/Table|
|T42.ExcelScript.Grid.OnRowAdded|Stream: Row Added to Sheet|
|T42.ExcelScript.Table.OnRowAdded|Stream: Row Added to Table|
|T42.ExcelScript.Grid.OnRowUpdated|Stream: Row Updated in Sheet|
|T42.ExcelScript.Table.OnRowUpdated|Stream: Row Updated in Table|
|T42.ExcelScript.Worksheet.OnAdded|Stream: Worksheet Added|
|T42.ExcelScript.Grid.FindRow|Method: Find Row in Sheet|
|T42.ExcelScript.Table.FindRow|Method: Find Row in Table|
|T42.ExcelScript.Workbook.IsOpen|Method: Find Workbook|
|T42.ExcelScript.Workbook.GetFolders|Method: Get Folders|
|T42.ExcelScript.Workbook.GetFiles|Method: Get Files|
|T42.ExcelScript.Workbook.Open|Method: Get Sheets|
|T42.ExcelScript.Grid.ReadRow|Method: Get Columns in Sheet|
|T42.ExcelScript.Table.ReadRow|Method: Get Columns in Table|
|T42.ExcelScript.Table.GetInformation|Method: Get Tables|
|T42.ExcelScript.Workbook.GetTemplates|Method: Get Templates|

## Reference

For a complete list of the available Excel Connector API methods and properties, see the [Excel Connector Reference Documentation](../../../../reference/glue4office/latest/excel/index.html).