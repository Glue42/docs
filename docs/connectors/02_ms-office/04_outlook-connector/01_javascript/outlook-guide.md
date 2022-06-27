## Introduction

The Glue42 Outlook Connector allows apps to use Outlook for creating emails, tasks, meetings and appointments, so that the user may view and edit these in a familiar environment. Once the items are sent or saved by the user, they can then be transmitted back to the app for processing, auditing and/or long-term storage.

The Glue42 Outlook Connector API lets your app interact with Outlook, and enables it to:

- Initiate new emails with rich HTML body and attachments and get notified when the email has been sent.
- Monitor certain folders in Outlook and get notified when an email has been received.
- Save complete email messages with attachments in your web app.
- Show a saved email in Outlook.
- Initiate the creation of new tasks and get notified when a task is created.
- Save complete tasks with attachments in your web app.
- Show a saved task in Outlook.

## Initialization

As shown in the [Set Up Your App](../../set-up-your-app/javascript/index.html) section, you need to initialize the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) library and set the [`outlook`](../../../../reference/glue4office/latest/outlook/index.html#OutlookAPI) property of the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) configuration object to `true`:

```javascript
const config = {
    // ...,
    outlook: true // enable Outlook integration
}
```

After that, get a reference to the [Glue42 Outlook Connector API](../../../../reference/glue4office/latest/outlook/index.html#OutlookAPI):

```javascript
Glue4Office(config)
    .then(g4o => {
        const outlook = g4o.outlook
        // interact with Outlook
    })
    .catch(console.error)
```

## Tracking Outlook Connector Status Changes

When [**Glue42 Enterprise**](https://glue42.com/enterprise/) is initialized, you can check whether Outlook is running and the Glue42 Outlook Connector is loaded:

```javascript
console.log(`The Glue42 Outlook Connector is ${outlook.addinStatus ? "available" : "unavailable"}`);
```

You can use the [`onAddinStatusChanged()`](../../../../reference/glue4office/latest/outlook/index.html#API-onAddinStatusChanged) method to track the availability of the Glue42 Outlook Connector. You may find this useful if you need to track when to enable or disable certain elements of your app user interface.

```javascript
const unsubscribe = outlook.onAddinStatusChanged(({ connected }) => {
    console.log(`The Glue42 Outlook Connector is ${connected ? "available" : "unavailable"}.`)
});
```

The `connected` argument passed to your callback will be `true` if and only if:

- Outlook is running.
- The Glue42 Outlook Connector is installed and enabled in Outlook.
- The Glue42 Outlook Connector and your app are using the same connectivity configuration and are connected to the same Glue42 Gateway.

In any other case, the `connected` flag will be `false`.

To stop listening for connection status changes, simply call the returned function:

```javascript
unsubscribe();
```

## Working with Emails

### Default Settings

**Default Email Account**

By default, the Glue42 Outlook Connector picks the first email account and uses that for all API calls.

**Default Monitored Folder in Outlook**

By default, the Glue42 Outlook Connector is set up to monitor a folder at the level of your Inbox, called `GlueHandleEmail` (you have to create that folder yourself in Outlook). You can configure the Outlook folder monitoring (rename the default folder, add more custom monitored folders) by changing the settings in the `foldermonitor.yaml` located in `%LocalAppData%\Tick42\GlueOutlook\UserConfig`.

**Whitelisted Folders**

Due to security reasons, [**Glue42 Enterprise**](https://glue42.com/enterprise/) is configured to have access only to specific folders. Keep that in mind when you want to create attachments, save files, etc. Folder access can be customized be changing the settings in the `whitelist.yaml` file located in `%AppData%\Tick42\Common`

### Creating a New Email

While there is no technical limitation for the Glue42 Outlook Connector to send an email, there are many reasons while this isn't a good idea. So, "creating a new email" actually means that the Glue42 Outlook Connector will create a new email window and populate it, but it won't send the email automatically and will instead let the user press the "Send" button. You can, however, track whether the user sends or cancels the email, as explained below in this document.

To create an email, use the [`newEmail()`](../../../../reference/glue4office/latest/outlook/index.html#API-newEmail) method and pass one or more email properties in an [`EmailParams`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams) object. The example below creates a new email with plain text. As with any email client, everything is optional - recipients, subject, body, etc.:

```javascript
outlook.newEmail({
        to: "someone@domain.com",
        cc: ["another@domain.com", "yetanother@domain.com"],
        subject: "Interesting topic",
        body: "Some plain text"
    })
    .then(() => console.log("A New Email window has been shown"))
    .catch(console.error)
```

Note that [`to`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-to), [`cc`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-cc) and [`bcc`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-bcc) allow you to set a single recipient (in a single string) or multiple recipients in an array.

Also note that you can't specify the sender of the email. The Glue42 Outlook Connector will automatically use your email account and set it up as a sender.

### Construct Emails with HTML

If you want to create an email with an HTML body, instead of setting [`body`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-body), set the [`bodyHtml`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-bodyHtml) property of the [`EmailParams`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams) object:

```javascript
outlook.newEmail({
        to: "manager@domain.com",
        subject: "Interesting report",
        bodyHtml: document.getElementById("reportTable").innerHTML
    })
```

If you set both the [`body`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-body) and [`bodyHtml`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-bodyHtml) properties, the `bodyHtml` property will take precedence.

### Tracking Sent or Canceled Emails

Note that when the `Promise` of [`newEmail()`](../../../../reference/glue4office/latest/outlook/index.html#API-newEmail) resolves, it means that the call from your web app to the Glue42 Outlook Connector has succeeded and it is guaranteed that the user will see a new email window populated with the parameters, as in the example above. However, this doesn't necessarily mean that the user will press the "Send" button.

In order to track whether the user has sent the email or dismissed the window and discarded the email, you need to pass one or two optional callbacks ([`onSent`](../../../../reference/glue4office/latest/outlook/index.html#NewEmailOptions-onSent) and [`onCanceled`](../../../../reference/glue4office/latest/outlook/index.html#NewEmailOptions-onCanceled)) in the [`NewEmailOptions`](../../../../reference/glue4office/latest/outlook/index.html#NewEmailOptions) object to get notified whether the user has sent or canceled the email:

```javascript
outlook.newEmail({
        to: "someone@domain.com",
        subject: "Interesting topic",
        body: "Some plain text"
    },
    {
        onSent: email => console.log("The user has sent the email", email),
        onCanceled: () => console.warn("The user has canceled the email")
    })
    .then(() => console.log("A New Email window has been shown"))
    .catch(console.error)
```

The [`onSent`](../../../../reference/glue4office/latest/outlook/index.html#NewEmailOptions-onSent) callback will pass you the email object which is of type [`T42Email`](../../../../reference/glue4office/latest/outlook/index.html#T42Email). This object has a property [`ids`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-ids) which contains a set of IDs uniquely identifying the email in multiple systems (e.g., CRMs), including Outlook.

If you save the email IDs in your app, you can later use them to display the email in Outlook. Take a look at [Showing Emails](#working_with_emails-showing_emails) to see how.

### Creating Attachments

To attach files to a new email, you need to set the [`attachments`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-attachments) property. There are two ways to add attachments to your emails:

- creating attachments directly from your app content;
- attaching existing files;

#### App Content

Suppose your app is displaying a list of customers, received as a `JSON` from a backend call. You can create a file from this data (e.g., CSV or HTML) and attach it to an email:

```javascript
outlook.newEmail({
        to: "someone@domain.com",
        subject: "Interesting topic",
        body: "Some plain text",
        attachments: [
            {
                // the "data" property expects base64 encoded data
                data: window.btoa(document.getElementById("customerList").innerHTML),
                fileName: "customers.html"
            }
        ]
    })
    .then(() => console.log("A New Email window has been shown"))
    .catch(console.error)
```

#### Existing Files

If you are using The Glue42 Excel or Word Connectors and have created a file for which you know the file path, you can attach the file by specifying the file paths in the [`attachments`](../../../../reference/glue4office/latest/outlook/index.html#EmailParams-attachments) property:

```javascript
outlook.newEmail({
        to: "someone@domain.com",
        // ...
        attachments: [pathToExcelWorkbook, pathToWordFile]
    })
    .then(() => console.log("A New Email window has been shown"))
    .catch(console.error)
```

### Email Monitoring and Notifications

Whenever an email arrives in a monitored folder, (e.g., from an Outlook rule which copies it from "Inbox"), your app can be notified about it. In order to subscribe for such events, you can use [`onEmailReceived()`](../../../../reference/glue4office/latest/outlook/index.html#API-onEmailReceived):

```javascript
outlook.onEmailReceived(email => console.log("An email arrived", email));
```

The [`email`](../../../../reference/glue4office/latest/outlook/index.html#Email) parameter is of type [`T42Email`](../../../../reference/glue4office/latest/outlook/index.html#T42Email) and contains the standard [`sender`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-sender), [`to`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-to), [`subject`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-subject), etc. properties. This object has a property [`ids`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-ids) which contains a set of IDs uniquely identifying the email in multiple systems (e.g., CRMs), including Outlook.

If you save the email IDs in your app, you can later use them to display the email in Outlook. Take a look at [Showing Emails](#working_with_emails-showing_emails) to see how.

When an email arrives in the monitored folder, the Glue42 Outlook Connector will try to map the email addresses to Outlook contacts, so [`sender`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-sender), [`to`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-to), [`cc`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-cc) and [`bcc`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-bcc) will contain instances of [`T42Contact`](../../../../reference/glue4office/latest/outlook/index.html#T42Contact).

In addition, any attachments in the [`attachments`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-attachments) property will be instances of [`T42Attachment`](../../../../reference/glue4office/latest/outlook/index.html#T42Attachment). Note that the data in the attachments can be quite large and is delivered to your app on demand. If you need to get hold of the attachment data (encoded in Base64), you can call the [`getData()`](../../../../reference/glue4office/latest/outlook/index.html#Attachment-getData) method:

```javascript
outlook.onEmailReceived(email => {
    const imageAttachment = email.attachments.filter(a => a.name === "icon.png")[0];

    imageAttachment.getData().then(imageData => {
        const img = document.getElementById("image");
        img.src = `data:image/png;base64,${imageData}`;
    }
});
```

The attachment data is received in chunks, visible to your app, as it could be quite large. You might want to let the user view the progress or even cancel the download. The [`getData()`](../../../../reference/glue4office/latest/outlook/index.html#Attachment-getData) method allows you to pass an optional callback to report the progress and cancel the download if you return `true` from the callback:

```javascript
attachment.getData(percent => {
            console.log(`Downloaded ${percent}%`)
            return userPressedCancelButton
        })
    .then(data => { ... })
    .catch(console.error)
```

### Saving Emails

If you have created a new email, the user has sent it and you want to save it in your app backend database, you can call the [`getAsMsg()`](../../../../reference/glue4office/latest/outlook/index.html#Email-getAsMsg) method of the [`Email`](../../../../reference/glue4office/latest/outlook/index.html#Email) object and retrieve the message, including its attachments, as an Outlook MSG file, encoded in Base64:

```javascript
outlook.newEmail({
        subject: "...",
        attachments: [ ... ],
        // ...
    },
    {
        onSent: email =>
            email.getAsMsg()
                .then(msgData => saveEmail(msgData))
                .catch(console.error)
    })
    .then(console.log)
    .catch(console.error)
```

You can use the same method if you want to save an email received from a monitored folder:

```javascript
outlook.onEmailReceived(email => email.getAsMsg().then(saveEmail));
```

### Showing Emails

If you have saved the IDs (the [`ids`](../../../../reference/glue4office/latest/outlook/index.html#T42Email-ids) property in [`T42Email`](../../../../reference/glue4office/latest/outlook/index.html#T42Email)) of an email either after the user has sent a new email, or when your app has received an email from a monitored folder, you can instruct Outlook to display this email using the [`showEmail()`](../../../../reference/glue4office/latest/outlook/index.html#API-showEmail) method. The `Promise` will resolve with the loaded email object:

```javascript
outlook.showEmail(ids)
    .then(email => console.log("Email shown", email))
    .catch(console.error)
```

## Working with Tasks

### Creating a New Task

The call for creating a new task is very similar to the one for creating emails. You just need to call [`newTask()`](../../../../reference/glue4office/latest/outlook/index.html#API-newTask) and pass one or more task properties in a [`TaskParams`](../../../../reference/glue4office/latest/outlook/index.html#TaskParams) object:

```javascript
outlook.newTask({
        subject: "Go to gym tomorrow",
        startDate: new Date("2019-07-19"),
        dueDate: new Date("2019-07-19"),
        reminderTime: new Date("2019-07-19 12:45 UTC"),
        body: "Sweat is fat crying!",
        priority: "high"
    })
    .then(() => console.log("A New Task window has been shown"))
    .catch(console.error)
```

Similarly to the [`newEmail()`](../../../../reference/glue4office/latest/outlook/index.html#API-newEmail) call, by default the Outlook Connector won't create and save the task automatically but will display the task creation window. You can change the behavior and create and save the task automatically by setting the `noUI` to `true`.

If you want to be notified when the user saves or cancels the task, you need to pass one or two optional callbacks ([`onSaved`](../../../../reference/glue4office/latest/outlook/index.html#NewTaskOptions-onSaved) and [`onCanceled`](../../../../reference/glue4office/latest/outlook/index.html#NewTaskOptions-onCanceled)) in the [`NewTaskOptions`](../../../../reference/glue4office/latest/outlook/index.html#NewTaskOptions) object to be notified if the user has saved the task or this or another user has canceled it:

```javascript
outlook.newTask({
        subject: "Go to gym tomorrow",
        startDate: new Date("2019-07-19"),
        dueDate: new Date("2019-07-19"),
        reminderTime: new Date("2019-07-19 12:45 UTC"),
        body: "Sweat is fat crying!",
        priority: "high"
    },
    {
        onSaved: task => console.log("The user saved the task", task),
        onCanceled: () => console.warn("The user canceled the task")
    })
    .then(() => console.log("A New Task window has been shown"))
    .catch(console.error)
```

The [`onSaved`](../../../../reference/glue4office/latest/outlook/index.html#NewTaskOptions-onSaved) callback will pass you the task object which is of type [`T42Task`](../../../../reference/glue4office/latest/outlook/index.html#NewTaskOptions). This object has a property [`ids`](../../../../reference/glue4office/latest/outlook/index.html#T42Task-ids) which contains a set of IDs uniquely identifying the task in multiple systems (e.g., CRMs), including Outlook.

If you save the task IDs in your app, you can later use them to display the task in Outlook. Take a look at [Showing Tasks](#working_with_tasks-showing_tasks) to see how.

### Task Attachments

Task attachments work in the same way as email ones - to attach files to a new task, you need to set the [`attachments`](../../../../reference/glue4office/latest/outlook/index.html#TaskParams-attachments) property. Again, just like for emails, the property supports:

- creating attachments directly from your app content;
- attaching existing files;

There are at least three kinds of attachments that can be added to a task - Word documents, created using the Glue42 Word Connector; Excel documents, created using the Glue42 Excel Connector; and Outlook MSG files, created by the Glue42 Outlook Connector.

Below is an example which saves an email as a task attachment:

```javascript
// task creation rule
function createTaskByCEO(email) {
    if (email.sender.emails.includes("company.ceo@company.com")) {
        email.getAsMsg()
        .then(data => {
            outlook.newTask({
                subject: `Review ${email.subject}`,
                body: email.body,
                startDate: new Date(),
                attachments: [{ data, fileName: `${email.subject}.msg` }],
                priority: "high"
            })
        })
    }
}

outlook.onEmailReceived(email => {
    createTaskByCEO(email);
})
```

### Saving Tasks

If you have created a new task, the user has saved it and you want to save it in your app backend database, you can call the [`saveToFile()`](../../../../reference/glue4office/latest/outlook/index.html#Task-saveToFile) method of the [`Task`](../../../../reference/glue4office/latest/outlook/index.html#Task) object and retrieve the task, including its attachments, as an Outlook MSG file, encoded in Base64:

```javascript
outlook.newTask({
        subject: "...",
        attachments: [ ... ],
        // ...
    },
    {
        onSaved: task => {
                task.saveToFile()
                .then(uri => console.log(uri))
                .catch(console.error);
        }
    })
    .then(console.log)
    .catch(console.error)
```

### Showing Tasks

If you have saved the IDs (the [`ids`](../../../../reference/glue4office/latest/outlook/index.html#T42Task-ids) property in [`T42Task`](../../../../reference/glue4office/latest/outlook/index.html#T42Task)) of a task after the user has saved it, you can instruct Outlook to display this task using the [`showTask()`](../../../../reference/glue4office/latest/outlook/index.html#API-showTask) method. The `Promise` will resolve with the loaded task object:

```javascript
outlook.showTask(ids)
    .then(task => console.log("Task shown", task))
    .catch(console.error)
```

## Reference

For a complete list of the available Outlook Connector API methods and properties, see the [Outlook Connector Reference Documentation](../../../../reference/glue4office/latest/outlook/index.html).