# M Extensions

**Table of Contents**
* [1 - Overview](#overview)
  * [1.1 - Additional Resources](#additional-resources)
  * [1.2 - Developer Tools](#developer-tools)
  * [1.3 - Extension Files (MEZ)](#extension-files-mez)
* [2 - Power Query SDK](#power-query-sdk)
  * [2.1 - Creating a Data Connector in Visual Studio](#creating-a-new-extension-in-visual-studio)
  * [2.2 - Testing in Visual Studio](#testing-in-visual-studio)
  * [2.3 - Build and Deploy from Visual Studio](#build-and-deploy-from-visual-studio)
* [3 - Technical Reference](#technical-reference)
  * [3.1 - Data Source Kind](#data-source-kind)
  * [3.2 - Publish to UI](#publish-to-ui)
  * [3.3 - Data Source Functions](#data-source-functions)
  * [3.4 - Authentication and Credentials](#authentication-and-credentials)
* [4 - Next Steps](#next-steps)

# Overview
Power BI extensions are created using M (also known as the Power Query Formula Language). This is the same language used by the Power Query (PQ) user experience found in Power BI Desktop (PBID) and Excel 2016. Extensions allow you to define new functions for the M language, and can be used to enable connectivity to new data sources. While this document will focus on defining new connectors, much of the same process applies to defining general purpose M functions. Extensions can vary in complexity, from simple wrappers that essentially just provide "branding" over existing data source functions, to rich connectors that support Direct Query (DQ).

The general process is:
1. Install the Power Query SDK from the Visual Studio Marketplace (TODO - provide link)
2. Create a new Data Connector project
3. Define your connector logic
4. Build the project to produce a .mez file
5. Set a **PQ_ExtensionDirectory** environment variable, set it to `"c:\program files\microsoft power bi desktop\bin\extensions"`
6. Copy the .mez file in your c:\program files\microsoft power bi desktop\bin\extensions directory
7. Restart Power BI Desktop 

**Note:** Setting the environment variable (Step 5) is temporary. Extensibility can be enabled as a Preview Feature in Power BI Desktop starting the June release.

## Additional Resources
* [M Library Functions](https://msdn.microsoft.com/en-US/library/mt253322.aspx)
* [M Language Specification](http://pqreference.azurewebsites.net/PowerQueryFormulaLanguageSpecificationAugust2015.pdf)
* [Power BI Developer Center](https://powerbi.microsoft.com/en-us/developers/)

## Developer Tools 

The following tools are recommended for developing PQ extensions.

| Tool                              | Description                                                                                                                                   | Location                                                   |
|:----------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------|
| Power Query SDK for Visual Studio | Visual studio extension (vsix) which provides the M Project templates, as well as syntax highlighting, intellisense, and build capabilities.  | TODO - marketplace link                                    |
| Power BI Desktop                  | Used to visually build M expressions and test out your data source extension.                                                                 | [Download](https://powerbi.microsoft.com/en-us/desktop/)   |

## Extension Files (MEZ)

PQ extensions are bundled in a zip file and given a .mez file extension. These are typically referred to as MEZ files. A MEZ will contain the following files:

* Script file containing your function and model definition (i.e. MyConnector.m)
* Icons of various sizes (i.e. *.png)
* Resource file(s) containing strings for localization (i.e. resources.resx)

At runtime, PBI Desktop will load extensions from directory defined by the PQ_ExtensionDirectory environment variable.
PBI Desktop will load files with both the .m and .mez format, however, use of a .mez is required if you want to include icons or localized strings for your extension.

# Power Query SDK

## Creating a New Extension in Visual Studio
Installing the Power Query SDK for Visual Studio will create a new Data Connector project template in Visual Studio.

(TODO - picture)

This creates a new project containing the following files:
1. Connector definition file (&lt;connectorName&gt;.m)
2. A query test file (&lt;connectorName&gt;.query.m)
3. A string resource file (resources.resx)
4. PNG files of various sizes

Your connector definition file will start with an empty Data Source Kind description. Please see the [Data Source Kind](#data-source-kind) section later in this document for details.

(TODO - picture)

## Testing in Visual Studio

&lt;TODO&gt;

## Build and Deploy from Visual Studio
Building your project will produce your .mez file.

Data Connector projects do not support custom post build steps to copy the output .mez file to your `PQ_ExtensionDirectory`. If this is something you want to do, you may want to use a third party visual studio extension, such as [Auto Deploy](https://visualstudiogallery.msdn.microsoft.com/9f7165ab-eef6-4576-8733-b630db1a59c0?SRC=VSIDE).

# Technical Reference

## Data Source Kind

The Resource record defines a specific data source extension. This record is where you'll define your exports (i.e. which functions will be exposed to end users), branding information, supported authentication types, and how to uniquely identify a credential (known as a "Resource Path").

Each Resource has a unique name (Description) used to identify and distinguish it from other data sources.

After defining your Resource, it is registered using the Extension.Module function.

### Properties

The following table lists the fields for your Resource definition. All fields are required unless indicated otherwise.

| Field              | Type     | Details                                                                                                                                                                                                                                                                   |
|:-------------------|:---------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Label              | text     | Friendly display name for this extension.                                                                                                                                                                                                                                 |
| Authentication     | record   | Specifies one or more types of authentication supported by your data source. At least one kind is required. Each kind will be displayed as an option in the Power Query credential prompt. For more information, see [Authentication Kinds](#authentication-kinds) below. |
| SupportsEncryption | logical  | **(optional)** When true, the UI will present the option to connect to the data source using an encrypted connection. This is typically used for data sources with a non-encrypted fallback mechanism (generally ODBC or ADO.NET based sources).                          |

## Publish to UI

This record provides the Power Query UI the information it needs to expose this extension in the Get Data dialog.

| Field               | Type    | Details                                                                                                                                                                                                                                                                                                                    |
|---------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ButtonText          | list    | List of text items that will be displayed next to the data source's icon in the Power BI Get Data dialog.                                                                                                                                                                                                                  |
| Category            | text    | Where the extension should be displayed in the Get Data dialog. Currently the only category values with special handing are `Azure` and `Database`. All other values will end up under the Other category.                                                                                                               |
| LearnMoreUrl        | text    | **(optional)** Url to website containing more information about this data source or connector.                                                                                                                                                                                                                             |
| Beta                | logical | **(optional)** When set to true, the UI will display a Preview/Beta identifier next to your connector name and a warning dialog that the implementation of the connector is subject to breaking changes.                                                                                                                   |
| SupportsDirectQuery | logical | **(optional)** Enables Direct Query for your extension.<br>**This is currently only supported for ODBC extensions.**                                                                                                                                                                                                       |
| SourceImage         | record  | **(optional)** A record containing a list of binary images (sourced from the .mez file using the **Extension.Contents** method). The record contains two fields (Icon16, Icon32), each with its own list. Each icon should be a different size.                                                                            |                                                                                                                                                                                                                              |
| SourceTypeImage     | record  | **(optional)** Similar to SourceImage, except the convention for many out of the box connectors is to display a sheet icon with the source specific icon in the bottom right corner. Having a different set of icons for SourceTypeImage is optional - many extensions simply reuse the same set of icons for both fields. |

## Data Source Functions
A Data Connector wraps and customizes the behavior of a [data source function in the M Library](https://msdn.microsoft.com/en-us/library/mt253322.aspx#Anchor_15). For example, an extension for a REST API would make use of the Web.Contents() function to make HTTP requests. Currently, a limited set of data source functions have been enabled to support extensibility.
- [Web.Contents](https://msdn.microsoft.com/en-us/library/mt260892.aspx)
- [OData.Feed](https://msdn.microsoft.com/en-us/library/mt260868.aspx)
- [Odbc.DataSource](https://msdn.microsoft.com/en-us/library/mt708843.aspx)
- [AdoDotNet.DataSource](https://msdn.microsoft.com/en-us/library/mt736964)
- [OleDb.DataSource](https://msdn.microsoft.com/en-us/library/mt790573.aspx)

## Authentication and Credentials

### Authentication Kinds

An extension can support one or more AuthenticationKinds. Each authentication kind is a different type of credential. The authentication UI displayed to end users in Power Query is driven by the type of credential(s) that an extension supports.

The list of supported authentication types is defined as part of an extension's [Resource](#resource) definition. Each AuthenticationKind value is a record with specific fields. The table below lists the expected fields for each kind. All fields are required unless marked otherwise.


| Authentication Kind | Field         | Description                                                                                                                 |
|:--------------------|:--------------|:----------------------------------------------------------------------------------------------------------------------------|
| Implicit            |               | The Implicit (anonymous) authentication kind does not have any fields.                                                      |
| OAuth               | StartLogin    | Function which provides the URL and state information for initiating an OAuth flow.<br><br>See [Implementing an OAuth Flow](#implementing-an-oauth-flow) below.|
|                     | FinishLogin   | Function which extracts the access\_token and other properties related to the OAuth flow.                                   |
|                     | Refresh       | **(optional)** Function that retrieves a new access token from a refresh token.                                             |
|                     | Logout        | **(optional)** Function that invalidates the user's current access token.                                                   |
|                     | Label         | **(optional)** A text value that allows you to override the default label for this AuthenticationKind.                      |
| UsernamePassword    | UsernameLabel | **(optional)** A text value to replace the default label for the *Username* text box on the credentials UI.                 |
|                     | PasswordLabel | **(optional)** A text value to replace the default label for the *Password* text box on the credentials UI.                 |
|                     | Label         | **(optional)** A text value that allows you to override the default label for this AuthenticationKind.                      |
| Windows             | UsernameLabel | **(optional)** A text value to replace the default label for the *Username* text box on the credentials UI.                 |
|                     | PasswordLabel | **(optional)** A text value to replace the default label for the *Password* text box on the credentials UI.                 |
|                     | Label         | **(optional)** A text value that allows you to override the default label for this AuthenticationKind.                      |
| Key                 | KeyLabel      | **(optional)** A text value to replace the default label for the *API Key* text box on the credentials UI.                  |
|                     | Label         | **(optional)** A text value that allows you to override the default label for this AuthenticationKind.                      |
| Parameterized       | Name          | A text value which identifies the type of parameterized credential.                                                         |
|                     | Fields        | A record containing the custom fields for this type of credential.<br><br>See [Implementing a Parameterized Authentication Kind](#implementing-a-parameterized-authentication-kind) for more details.|
|                     | Label         | **(optional)** A text value that allows you to override the default label for this AuthenticationKind.                      |

The sample below shows the Authentication record for a connector that supports OAuth, Windows, Basic (Username and Password), and anonymous credentials.

```
Authentication = [
    OAuth = [
        StartLogin = StartLogin,
        FinishLogin = FinishLogin,
        Refresh = Refresh,
        Logout = Logout
    ],
    Key = [],
    UsernamePassword = [],
    Windows = [],
    Implicit = []
]
```

### Accessing the Current Credentials
The current credentials can be retrieved using the **Extension.CurrentCredential**() function.

M data source functions that have been enabled for extensibility will automatically inherit your extension's credential scope. In most cases, you will not need to explicitly access the current credentials, however, there are exceptions, such as:

-   Passing in the credential in a custom header or query string parameter (such as when you are using the API Key auth type)

-   Setting connection string properties for ODBC or ADO.NET extensions

-   Checking custom properties on an OAuth token

-   Using the credentials as part of an OAuth v1 flow

The Extension.CurrentCredential() function returns a record object. The fields it contains will be authentication type specific. See the table below for details.

| Field              | Description                                                                                                                                                                                                                                                                                                                                          | Used By                        |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| AuthenticationKind | Contains the name of the authentication kind assigned to this credential (UsernamePassword, OAuth, etc).                                                                                                                                                                                                                                             | All                            |
| Username           | Username value                                                                                                                                                                                                                                                                                                                                       | UsernamePassword, Windows      |  
| Password           | Password value. Typically used with UsernamePassword, but it is also set for Key.                                                                                                                                                                                                                                                                    | Key, UsernamePassword, Windows |
| access_token       | OAuth access token value.                                                                                                                                                                                                                                                                                                                            | OAuth                          |
| Properties         | A record containing other custom properties for a given credential. Typically used with OAuth to store additional properties (such as the refresh\_token) returned with the access\_token during the authentication flow.                                                                                                                            | OAuth                          |
| Key                | The API key value. Note, the key value is also available in the Password field as well. By default the mashup engine will insert this in an Authorization header as if this value were a basic auth password (with no username). If this is not the behavior you want, you must specify the ManualCredentials = true option in the options record.   | Key                            |
| EncryptConnection  | A logical value that determined whether to require an encrypted connection to the data source. This value is available for all Authentication Kinds, but will only be set if EncryptConnection is specified in the [Data Source](#data-source-kind) definition.                                                                                      | All                            |

The following is an example of accessing the current credential for an API key and using it to populate a custom header (`x-APIKey`).

```
MyConnector.Raw = (_url as text) as binary =>
let
    apiKey = Extension.CurrentCredential()[Key],
    headers = [

        #"x-APIKey" = apiKey,
        Accept = "application/vnd.api+json",
        #"Content-Type" = "application/json"
    ],
    request = Web.Contents(_url, [ Headers = headers, ManualCredentials = true ])
in
    request
```

### Implementing an OAuth Flow
Please see the [full Github sample](../samples/github). 

### Implementing a Parameterized Authentication Kind
The following example implements a custom parameterized authentication kind for the Spark connector. It contains three fields – Username, Password, and authmech.

Note, a data source may only have a single Parameterized authentication kind.

(TODO)

# Next Steps
(TODO)
* Samples and walkthroughs
* Using navigation tables
* Enabling Direct Query for an ODBC based connector
* OData based connectors
* Advanced connector scenarios with Table.View