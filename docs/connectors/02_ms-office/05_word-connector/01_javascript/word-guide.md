## Introduction

The Glue42 Word Connector allows applications to use MS Word as a document editor. The application sends document text and images to Word in the form of HTML, so that the user may edit it using a familiar tool. When the user saves the changes, a new HTML document is created and sent back to the application to display and/or store in the backend where it can be properly secured and audited.

Summary of features:

- The HTML document is sent to the application every time the user saves the document.
- Images may be embedded within the document HTML, as long as they are in one of the following formats: `PNG`, `ICO`, `BMP`, or `JPEG`.
- Embedded objects, such as images, charts and spreadsheets can be added to the Word document, but these will be converted to inline HTML images before they are returned to the application. The original objects can be preserved in Microsoft `DOCX` format, however, and can be saved to the backend so the user can edit the document later.
- Documents are created in the folder `%APPDATA%\Tick42\GlueWordPad\Files` (configurable).

## Initialization

As shown in the [Set Up Your Application](../../set-up-your-application/javascript/index.html) section, you need to initialize the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) library and set the [`word`](../../../../reference/glue4office/latest/word/index.html) property of the [`glue4office`](../../../../reference/glue4office/latest/glue4office/index.html) configuration object to `true`:

```javascript
const config = {
    // ...,
    word: true // enable Word integration
}
```

After that, get a reference to the [Glue42 Word Connector API](../../../../reference/glue4office/latest/word/index.html):

```javascript
Glue4Office(config)
    .then(g4o => {
        const word = g4o.word
        // interact with Word
    })
    .catch(console.error)
```

## Tracking Word Connector Status Changes

When [**Glue42 Enterprise**](https://glue42.com/enterprise/) is initialized, you can check whether Word is running and the Glue42 Word Connector is loaded:

```javascript
console.log(`The Glue42 Word Connector is ${word.addinStatus ? "available" : "unavailable"}`);
```

You can use the `onAddinStatusChanged()` method to track the availability of the Glue42 Word Connector. You may find this useful if you need to track when to enable or disable certain elements of your app user interface.

```javascript
const unsubscribe = word.onAddinStatusChanged(available => {
    console.log(`The Glue42 Word Connector is ${available ? "available" : "unavailable"}`)
});
```

The `available` argument passed to your callback will be `true` if and only if:

- Word is running.
- The Glue42 Word Connector is installed and enabled in Word.
- The Glue42 Word Connector and your app are using the same connectivity configuration and are connected to the same Glue42 Gateway.

In any other case, the `available` flag will be `false`.

To stop listening for connection status changes, simply call the returned function:

```javascript
unsubscribe();
```

## Sending a Document to Word

Typically, you would have some contents formatted as HTML in your web application and would like to let the user edit this in Word. When the user updates the document, you may want to get the changes back into your application.

Most of the time this is works, because HTML can express lots in terms of formatting. However, certain objects in Word, such as WordArt, embedded Excel spreadsheets, etc., don't have a round-trip representation in HTML and are automatically converted to images by Word. That is why the Glue42 Word Connector API lets you send dually formatted documents back and forth between your application and Word - one in HMTL format, which is used to display the document, and another in `DOCX` (the Word `XML` document format) which you send from your application to Word so you can keep the original objects editable.

### Creating a New Document

To create a document with your application contents, call the [`openDocument()`](../../../../reference/glue4office/latest/word/index.html#API-openDocument) method of the Word Connector API, passing the name of the document and its HTML representation packed in a [`OpenDocumentConfig`](../../../../reference/glue4office/latest/word/index.html#OpenDocumentConfig) object:

```javascript
const htmlContents = document.getElementById("contents").innerHTML;

word.openDocument({
    data: htmlContents
    })
    .then(wordDoc => console.log("Document created"))
    .catch(console.error)
```

The `Promise` returned by the [`openDocument()`](../../../../reference/glue4office/latest/word/index.html#API-openDocument) method resolves with a reference to a [`Document`](../../../../reference/glue4office/latest/word/index.html#OpenDocumentConfig) object. If you are planning on tracking any updates performed by the user, you will need to keep a reference to the opened document.

### Preventing Save of Temporary Documents

If your users need to edit data in Word, but aren't allowed to save it locally and should instead return the data to be saved in your application, you can set the `inhibitLocalSave` flag to `true`, which will prevent the users from saving the temporary document.

### Opening Existing Documents

If you already have a document in your application which you want to let the user edit, you would most certainly want to send the `DOCX` version to Word, not the HTML one, so that Word can work with the original document and preserve WordArt, embedded Excel sheets and so on, as the document circulates between Word and your app:

```javascript
fetchDocument()
.then(docxData => word.openDocument({
    data: docxData
    })
)
.then(wordDoc => console.log("Document loaded"))
.catch(console.error)
```

## Receiving Updates from Word

Once you have obtained a reference to the opened document, you can subscribe for and start tracking updates by the user using the [`onChanged()`](../../../../reference/glue4office/latest/word/index.html#DocumentApi-onChanged) method of the [`Document`](../../../../reference/glue4office/latest/word/index.html#OpenDocumentConfig) object:

```javascript
word.openDocument({
        data: document.getElementById("contents").innerHTML
    })
    .then(wordDoc => {
        console.log("Document created");

        wordDoc.onChanged((html, docx) => {
            document.getElementById("contents").innerHTML = html;
            postSave(docx)   // save document to backend
        })
    })
    .catch(console.error)
```

You can see from the example above that the [`onChanged()`](../../../../](../../../../reference/glue4office/latest/word/index.html#DocumentApi-onChanged)) callback will be called with both an HMTL representation of the document, which you can render in your application, as well as the `DOCX` version of the document (represented as Base64 string) which you can send to your backend for audit/storage purposes.

## Tracking Closing a Document

There are some occasions when you want to be notified when the user closes the document in Word or even exits Word. You can subscribe for such a change using the [`onClose()`](../../../../reference/glue4office/latest/word/index.html#DocumentApi-onClose) method of the [`Document`](../../../../reference/glue4office/latest/word/index.html#OpenDocumentConfig) object received by the [`openDocument()`](../../../../reference/glue4office/latest/word/index.html#API-openDocument) call:

```javascript
wordDoc.onClosed(doc => closeDocumentPreviewFor(doc));
```

## Reference

For a complete list of the available Word Connector API methods and properties, see the [Word Connector Reference Documentation](../../../../reference/glue4office/latest/word/index.html).