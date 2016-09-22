---
[//]: # (Copyright IBM Corp. 2016.  All Rights Reserved.)

layout: reference
title:  "Verse Extensibility Documentation"
categories: reference
---

# {{page.title}}

### Contents

* [Introduction to IBM Verse Extensibility](#introduction-to-ibm-verse-extensibility)
* [Extensibility Concepts](#extensibility-concepts)
* [Verse Action Extensions](#verse-action-extensions)
* [Registering an Application in IBM Verse](#registering-an-application-in-ibm-verse)
* [Sending and Receiving Data](#sending-and-receiving-data)
* [Verse API Data](#verse-api-data)
* [Editing the Manifest](#editing-the-manifest)
* [Troubleshooting](#troubleshooting)

## Introduction to IBM Verse Extensibility

Verse Extensibility allows you to integrate your own web applications with IBM Verse, by registering your application with it. Your application can declare one or more extensions which will enhance Verse with some new functionality.

For example, one of the extension points that Verse supports is an action extension. An action extension is typically displayed as a button or link in the Verse User Interface (UI) which when clicked triggers some logic in your application.

An extension can declare that it requires specific data from Verse and when the extension is activated Verse will send this data to it. For example, if you add a link to a Verse business card your extension can be configured to receive the email address of the person when the link is clicked.

## Extensibility Concepts  
This section introduces extensibility concepts and terminology which is used throughout this document.

* Application: A third-party web application which contributes new functionality to IBM Verse.
  An application can contribute one or more new features to different parts of the IBM Verse UI.
  The URL of the application is registered with Verse and the application will be opened in a separate window.
  The application will have access to Verse data through cross-document messaging or URL query string parameters.

* Extension: A feature contributes to a specific part of IBM Verse. For example, an action extension that adds a button or link to the Verse UI which when clicked will open a new browser window containing a third-party web application.

* `applications.json`: This file contains the details of your application: where in the Verse UI your extension(s) will appear, how your application communicates with Verse, etc. See the `applications.json` [section](#registering-an-application-in-ibm-verse) for more information.

## Verse Action Extensions
Verse supports action extensions in various places in the Verse UI: the mail compose view, the mail read view and the business card, etc.
The `mail.compose` action extension adds an additional action to the `more actions` button when composing a mail.
When an extension is clicked, Verse opens your application in a new window and sends the requested information to it.
For example, if you add a `mail.read` action extension Verse will send the extension the mail subject, body, recipients, date etc. See [here](#action-extensions) for the full reference of action extensions.

## Registering an Application in IBM Verse
To add an application to Verse, you need to register it using the IBM App Registry. For development purposes
you can use the [IBM Verse Developer Extension for Google Chrome][4]{:target="_blank"}. There is a [tutorial](../tutorials/tutorial_verse_developer.html){:target="_blank"} to get you started.

### Your Application
You will need to provide Verse with the URL to your application. Once an action extension is clicked in the Verse UI, the URL will be loaded in a new window.
The page that is loaded needs some Javascript to listen for a window message event which contains a context object. This object has information from Verse for your extension as specified in the `applications.json` file.

When using the Chrome extension you will need to add the URL of your application and the action extension(s) to the `applications.json` file. The Chrome extension will use the extension definitions from this file and register them with Verse.

### File structure of `applications.json`

The `applications.json` file contains a list of Application definitions in JSON format

```javascript
  [
    {
      "app_id": "com.ibm.verse.actions.sample1",
      ...
    },
    {
      "app_id": "com.ibm.verse.actions.sample2",
      ...
    }
  ]
```

Here are three different samples of valid `applications.json`:

* [Sample 1][1]{:target="_blank"}
* [Sample 2][2]{:target="_blank"}
* [Sample 3][3]{:target="_blank"}

### Application Properties

An application definition __must__ contain the following properties:

* `app_id` Unique identifier for the application of the form com.companyName.
* `name` The name of your application. This must be __unique__.
* `url` This is the URL of your application.
* `extensions` An array of of extension definitions. See below for the properties of this object.
* `Payload` Describes the method of communication between Verse and the application as well as display options for the new window.
* `Services` Describes which services the extension is deployed to. `"Verse"` is the only supported value.

### Extension Properties

An extension definition __must__ contain the following properties. __N.B. only one of `object` or `path` is required__:

* `ext_id` The ID of the action extension. This must be __unique__.
* `type` The type property indicates the type of extension being configured (for example, `com.ibm.verse.action` specifies an action contribution).
* `name` The name of the action extension in the UI. This must be __unique__.
* `payload` The payload property indicates optional properties of the extension (for example, you can insert an icon into your action button by adding `"svg": "<your-svg-element-here>"`). _The `payload` property is required but its value may be empty_.
* `object` The object property indicates that the extension displays in a view which provides the specific object.  
Using the person value specifies that the extension displays in a view which provides the person object.  
For example, if the business card view provides the person data type, then the action contribution will be shown on the business card view.  
_This property is not required if you are using the `path` property._
* `path` The path property is used to display an action extension in the mail compose view or the mail read view. Valid values are `"mail.read"` OR `"mail.compose"`. *This property is not required if you are using the `object` property.*
* `title` The title of your action extension which will appear in the Verse UI.

### Payload Properties

* `features` This property indicates which Verse API is called by the application. The list of features is an array, enclosed in square brackets. Currently, the only accepted value is `["core"]` which indicates that we are invoking the Verse core API.
* `renderParams` This is an object that contains properties on how the application window is displayed. This object is passed to `window.open()`. See [here][6]{:target="_blank"} for a complete list of properties.

## Sending and Receiving Data

Verse supports both URL query string parameters or cross-document messaging to communicate
with your application. Both methods are described below.

### Passing data in a URL

Your application can receive data from Verse through URL Query String parameters which are added to the URL specified in the `applications.json` file. Valid parameters are described in the [Verse API Data](#verse-api-data) section.

For Example, to send the name of a user from a business card extension to your application, specify the following URL in the `applications.json` file: `https://<your-domain-here>/extension.html?username=<profile.name>`. `profile.name` is a variable which holds the user's name.

In your application you retrieve the URL query string parameters as normal.

### Passing data through cross-document messaging

If your web application cannot support receiving data from a URL request, you can use cross-document messaging instead. To use this method, you must
add the features property to the manifest with the value of `["core"]` so that your web application can communicate with Verse.

In your application code you must send a `"com.ibm.verse.application.loaded"` message back to the Verse window, so Verse knows your web application is ready to receive data from it. If you have a reference to the Verse window you can do this at the beginning of your code, otherwise wait for the message `"com.ibm.verse.ping.application.loaded"` as the source of this message will be the Verse window.

To handle messages from Verse, your web application needs to register an event listener by using

```javascript
  window.addEventListener("message", function(event) {
    // handle message event code
  });
```

See [here][5]{:target="_blank"} for the complete code source of a sample application that demonstrates the concepts described in this section.

## Verse API Data

Verse is able to send data to your application using what's known as a context object. It is passed in the message event object of the `window.postMessage` function. This section shows the structure of the different context objects that are sent by Verse.

The information contained in the context object depends on the action extension used. For example, adding a mailRead action extension sends information relating to the selected mail: title, subject, body, etc. Conversely, adding a business card action extension sends the person's name, email, phone, etc.

### Parsing the Verse API Data

The message event received by your application contains an object called `data` which has an object called `verseApiData`. You will need to check that the `actionId` property of the `verseApiData` object matches the `ext_id` in your `application.json` file. This will ensure you only run your code for the correct message events.

For example:

```javascript
  window.addEventListener("message", function(event) {
    if (event.data.verseApiData.actionId === "com.ibm.verse.action.sample.person") {
      var verseData = event.data.verseApiData.context;
    }
  }
```

In the code sample above, you can see that the information we need from Verse is stored in the context property, which we check with the if statement. The value of the verseData variable depends on which extension is used.

In the sections below the structure of each of the different context objects is outlined.

#### Action Extensions

* [mail.compose:](#mail-compose) This appears when composing a new mail under the `more actions` button.
* [mail.read:](#mail-read-view) This appears when viewing an existing mail under the `more actions` button.
* [person:](#business-card) This appears on the back of the business card.


#### Mail Compose

```
  {
    "body": "",
    "contextId": "",
    "id": "",
    "recipientCC": [
      {
	"displayName": "",
	"emailAddress": ""
      }
    ],
    "recipientTo": [
      {
	"displayName": "",
	"emailAddress": ""
      }
    ],
    "subject": ""
  }
```

#### Mail Read View

```
  {
    "body": "",
    "contextId": "",
    "id": "",
    "recipientCC": [
      {
        "displayName": "",
        "emailAddress": ""
      }
    ],
    "recipientTo": [
      {
        "displayName": "",
        "emailAddress": ""
      }
    ],
    "sender": {
      "displayName": "",
      "emailAddress": ""
    },
    "subject": "",
    "timeSent": ""
  }
```

#### Business Card

```
  {
    "currentUser": {
      "company": "",
      "displayName": "",
      "fax": "",
      "id": "",
      "jobTitle": "",
      "mobilePhone": "",
      "name": {
	"displayName": "",
	"displayType": "",
	"familyName": "",
	"formatted": "",
	"givenName": "",
	"honorificPrefix": "",
	"honorificSuffix": "",
	"middleName": ""
      },
      "network": "",
      "orgId": "",
      "photo": "",
      "photoUrl": "",
      "primaryAddress": "",
      "primaryEmail": "",
      "primaryPhone": "",
      "status": "",
      "tags": "",
      "website": ""
    },
    "profile": {
      "company": "",
      "displayName": "",
      "fax": "",
      "id": "",
      "jobTitle": "",
      "mobilePhone": "",
      "name": {
	"displayName": "",
	"displaytype": "",
	"familyName": "",
	"formatted": "",
	"givenName": "",
	"honorificPrefix": "",
	"honorificSuffix": "",
	"middleName": ""
      },
      "network": "",
      "orgId": "",
      "photo": "",
      "photoUrl": "",
      "primaryAddress": "",
      "primaryEmail": "",
      "primaryPhone": "",
      "status": "",
      "tags": "",
      "website": ""
    }
  }
```

## Editing The Manifest

If the URL you use to access Verse is specific to your company, you will need to add it to the `manifest.json` file. You can follow our tutorial on [how to update manifest.json][7]{:target="_blank"}.

After you modify this file you will need to reload the Chrome extension and refresh Verse to pick up your latest changes.


## Troubleshooting

This section describes some common issues you may experience as well as providing information on how to debug your application.

### The Chrome Developer Extension will not load

If you are experiencing problems loading the Chrome Developer Extension as an unpacked extension this is most likely an issue with the `manifest.json` or `applications.json` files. When you edit these files ensure they are well-formed: no trailing commas, no missing quotation marks, no syntax errors, etc.

### The Chrome Developer Extension loads but your extensions do not appear in Verse

If your extensions are not appearing in the Verse UI make sure you have specified
the correct path and object properties in the [`applications.json`](#registering-an-application-in-ibm-verse).
Also ensure that you have added the URL you use to access Verse to the manifest file. See
[here][7]{:target="_blank"} to read more on editing the manifest.

### Debugging your application

Your application opens in a new window but is not working as expected or does not appear to have received any data from Verse.

This may be an issue with the `applications.json` file.
If you are using URL query string parameters, check that the variables you added to your URL are valid and correspond to properties in the Verse API data [section](#verse-api-data).
If you are using cross-document messaging check that you have specified the value `["core"]` for the `payload.features` property.

If you are still having problems you will need to debug your application code using the browser developer tools. Since Verse opens your application in a new window which immediately executes your code, you will have no time to open the developer tools and set break points.

The simplest solution to this is to add an `alert();` followed by a `debugger;` statement.

When the new window opens and your application code starts to execute, the alert will appear and will pause the execution of your code until the alert is dismissed. Before dismissing the alert, open the developer tools. Now dismiss the alert and the the code will pause at your debugger statement.

You should inspect the message event listeners and make sure that they are receiving the correct data from Verse. You should also check that you are sending the `com.ibm.verse.application.loaded` message to the Verse window to indicate your application is ready to receive data from the Verse window.

See [here](#sending-and-receiving-data) for more information on communicating with the Verse window.


[1]: {{site.verse-developer-chrome-ext}}/blob/master/src/applications.json
[2]: {{site.verse-developer-chrome-ext}}/blob/master/tutorial/step3/applications.json
[3]: {{site.verse-developer-chrome-ext}}/blob/master/tutorial/step4/applications.json
[4]: {{site.verse-developer-chrome-ext}}
[5]: {{site.verse-developer-chrome-ext}}/blob/gh-pages/samples/actions.html
[6]: https://developer.mozilla.org/en-US/docs/Web/API/Window/open
[7]: ../tutorials/tutorial_verse_developer.html#update-manifestjson