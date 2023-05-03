# pdf-generator-api

PDF Generator API
- API version: 4.0.2
  - Build date: 2023-05-03T15:08:54.021423+03:00[Europe/Tallinn]

# Introduction
[PDF Generator API](https://pdfgeneratorapi.com) allows you easily generate transactional PDF documents and reduce the development and support costs by enabling your users to create and manage their document templates using a browser-based drag-and-drop document editor.

The PDF Generator API features a web API architecture, allowing you to code in the language of your choice. This API supports the JSON media type, and uses UTF-8 character encoding.

## Base URL
The base URL for all the API endpoints is `https://us1.pdfgeneratorapi.com/api/v4`

For example
* `https://us1.pdfgeneratorapi.com/api/v4/templates`
* `https://us1.pdfgeneratorapi.com/api/v4/workspaces`
* `https://us1.pdfgeneratorapi.com/api/v4/templates/123123`

## Editor
PDF Generator API comes with a powerful drag & drop editor that allows to create any kind of document templates, from barcode labels to invoices, quotes and reports. You can find tutorials and videos from our [Support Portal](https://support.pdfgeneratorapi.com).
* [Component specification](https://support.pdfgeneratorapi.com/en/category/components-1ffseaj/)
* [Expression Language documentation](https://support.pdfgeneratorapi.com/en/category/expression-language-q203pa/)
* [Frequently asked questions and answers](https://support.pdfgeneratorapi.com/en/category/qanda-1ov519d/)

## Definitions

### Organization
Organization is a group of workspaces owned by your account.

### Workspace
Workspace contains templates. Each workspace has access to their own templates and organization default templates.

### Master Workspace
Master Workspace is the main/default workspace of your Organization. The Master Workspace identifier is the email you signed up with.

### Default Template
Default template is a template that is available for all workspaces by default. You can set the template access type under Page Setup. If template has "Organization" access then your users can use them from the "New" menu in the Editor.

### Data Field
Data Field is a placeholder for the specific data in your JSON data set. In this example JSON you can access the buyer name using Data Field `{paymentDetails::buyerName}`. The separator between depth levels is :: (two colons). When designing the template you don’t have to know every Data Field, our editor automatically extracts all the available fields from your data set and provides an easy way to insert them into the template.
```
{
    "documentNumber": 1,
    "paymentDetails": {
        "method": "Credit Card",
        "buyerName": "John Smith"
    },
    "items": [
        {
            "id": 1,
            "name": "Item one"
        }
    ]
}
```

## Rate limiting
Our API endpoints use IP-based rate limiting and allow you to make up to 2 requests per second and 60 requests per minute. If you make more requests, you will receive a response with HTTP code 429.

Response headers contain additional values:

| Header   | Description                    |
|--------|--------------------------------|
| X-RateLimit-Limit    | Maximum requests per minute                   |
| X-RateLimit-Remaining    | The requests remaining in the current minute               |
| Retry-After     | How many seconds you need to wait until you are allowed to make requests |

## Postman Collection
We have created a [Postman](https://www.postman.com) Collection so you can easily test all the API endpoints without developing and code. You can download the collection [here](https://god.gw.postman.com/run-collection/11578263-c6546175-de49-4b35-904b-29bb52a5a69a?action=collection%2Ffork&collection-url=entityId%3D11578263-c6546175-de49-4b35-904b-29bb52a5a69a%26entityType%3Dcollection%26workspaceId%3D5900d75f-c45d-4e61-9fb7-63aca23580df) or just click the button below.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://god.gw.postman.com/run-collection/11578263-c6546175-de49-4b35-904b-29bb52a5a69a?action=collection%2Ffork&collection-url=entityId%3D11578263-c6546175-de49-4b35-904b-29bb52a5a69a%26entityType%3Dcollection%26workspaceId%3D5900d75f-c45d-4e61-9fb7-63aca23580df)

*  *  *  *  *

# Authentication
The PDF Generator API uses __JSON Web Tokens (JWT)__ to authenticate all API requests. These tokens offer a method to establish secure server-to-server authentication by transferring a compact JSON object with a signed payload of your account’s API Key and Secret.
When authenticating to the PDF Generator API, a JWT should be generated uniquely by a __server-side application__ and included as a __Bearer Token__ in the header of each request.


<SecurityDefinitions />

## Accessing your API Key and Secret
You can find your __API Key__ and __API Secret__ from the __Account Settings__ page after you login to PDF Generator API [here](https://pdfgeneratorapi.com/login).

## Creating a JWT
JSON Web Tokens are composed of three sections: a header, a payload (containing a claim set), and a signature. The header and payload are JSON objects, which are serialized to UTF-8 bytes, then encoded using base64url encoding.

The JWT's header, payload, and signature are concatenated with periods (.). As a result, a JWT typically takes the following form:
```
{Base64url encoded header}.{Base64url encoded payload}.{Base64url encoded signature}
```

We recommend and support libraries provided on [jwt.io](https://jwt.io/). While other libraries can create JWT, these recommended libraries are the most robust.

### Header
Property `alg` defines which signing algorithm is being used. PDF Generator API users HS256.
Property `typ` defines the type of token and it is always JWT.
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
The second part of the token is the payload, which contains the claims  or the pieces of information being passed about the user and any metadata required.
It is mandatory to specify the following claims:
* issuer (`iss`): Your API key
* subject (`sub`): Workspace identifier
* expiration time (`exp`): Timestamp (unix epoch time) until the token is valid. It is highly recommended to set the exp timestamp for a short period, i.e. a matter of seconds. This way, if a token is intercepted or shared, the token will only be valid for a short period of time.

```
{
  "iss": "ad54aaff89ffdfeff178bb8a8f359b29fcb20edb56250b9f584aa2cb0162ed4a",
  "sub": "demo.example@actualreports.com",
  "exp": 1586112639
}
```

### Signature
To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that. The signature is used to verify the message wasn't changed along the way, and, in the case of tokens signed with a private key, it can also verify that the sender of the JWT is who it says it is.
```
HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    API_SECRET)
```

### Putting all together
The output is three Base64-URL strings separated by dots. The following shows a JWT that has the previous header and payload encoded, and it is signed with a secret.
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhZDU0YWFmZjg5ZmZkZmVmZjE3OGJiOGE4ZjM1OWIyOWZjYjIwZWRiNTYyNTBiOWY1ODRhYTJjYjAxNjJlZDRhIiwic3ViIjoiZGVtby5leGFtcGxlQGFjdHVhbHJlcG9ydHMuY29tIn0.SxO-H7UYYYsclS8RGWO1qf0z1cB1m73wF9FLl9RCc1Q

// Base64 encoded header: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
// Base64 encoded payload: eyJpc3MiOiJhZDU0YWFmZjg5ZmZkZmVmZjE3OGJiOGE4ZjM1OWIyOWZjYjIwZWRiNTYyNTBiOWY1ODRhYTJjYjAxNjJlZDRhIiwic3ViIjoiZGVtby5leGFtcGxlQGFjdHVhbHJlcG9ydHMuY29tIn0
// Signature: SxO-H7UYYYsclS8RGWO1qf0z1cB1m73wF9FLl9RCc1Q
```

## Temporary JWTs
You can create a temporary token in [Account Settings](https://pdfgeneratorapi.com/account/organization) page after you login to PDF Generator API. The generated token uses your email address as the subject (`sub`) value and is valid for __15 minutes__.
You can also use [jwt.io](https://jwt.io/) to generate test tokens for your API calls. These test tokens should never be used in production applications.
*  *  *  *  *

# Error codes

| Code   | Description                    |
|--------|--------------------------------|
| 401    | Unauthorized                   |
| 402    | Payment Required               |
| 403    | Forbidden                      |
| 404    | Not Found                      |
| 422    | Unprocessable Entity           |
| 429    | Too Many Requests              |
| 500    | Internal Server Error          |

## 401 Unauthorized
| Description                                                             |
|-------------------------------------------------------------------------|
| Authentication failed: request expired                                  |
| Authentication failed: workspace missing                                |
| Authentication failed: key missing                                      |
| Authentication failed: property 'iss' (issuer) missing in JWT           |
| Authentication failed: property 'sub' (subject) missing in JWT          |
| Authentication failed: property 'exp' (expiration time) missing in JWT  |
| Authentication failed: incorrect signature                              |

## 402 Payment Required
| Description                                                             |
|-------------------------------------------------------------------------|
| Your account is suspended, please upgrade your account                  |

## 403 Forbidden
| Description                                                             |
|-------------------------------------------------------------------------|
| Your account has exceeded the monthly document generation limit.        |
| Access not granted: You cannot delete master workspace via API          |
| Access not granted: Template is not accessible by this organization     |
| Your session has expired, please close and reopen the editor.           |

## 404 Entity not found
| Description                                                             |
|-------------------------------------------------------------------------|
| Entity not found                                                        |
| Resource not found                                                      |
| None of the templates is available for the workspace.                   |

## 422 Unprocessable Entity
| Description                                                             |
|-------------------------------------------------------------------------|
| Unable to parse JSON, please check formatting                           |
| Required parameter missing                                              |
| Required parameter missing: template definition not defined             |
| Required parameter missing: template not defined                        |

## 429 Too Many Requests
| Description                                                             |
|-------------------------------------------------------------------------|
| You can make up to 2 requests per second and 60 requests per minute.   |

*  *  *  *  *


  For more information, please visit [https://support.pdfgeneratorapi.com](https://support.pdfgeneratorapi.com)

*Automatically generated by the [OpenAPI Generator](https://openapi-generator.tech)*


## Requirements

Building the API client library requires:
1. Java 1.8+
2. Maven (3.8.3+)/Gradle (7.2+)

## Installation

To install the API client library to your local Maven repository, simply execute:

```shell
mvn clean install
```

To deploy it to a remote Maven repository instead, configure the settings of the repository and execute:

```shell
mvn clean deploy
```

Refer to the [OSSRH Guide](http://central.sonatype.org/pages/ossrh-guide.html) for more information.

### Maven users

Add this dependency to your project's POM:

```xml
<dependency>
  <groupId>org.pdfgeneratorapi</groupId>
  <artifactId>pdf-generator-api</artifactId>
  <version>4.0.2</version>
  <scope>compile</scope>
</dependency>
```

### Gradle users

Add this dependency to your project's build file:

```groovy
  repositories {
    mavenCentral()     // Needed if the 'pdf-generator-api' jar has been published to maven central.
    mavenLocal()       // Needed if the 'pdf-generator-api' jar has been published to the local maven repo.
  }

  dependencies {
     implementation "org.pdfgeneratorapi:pdf-generator-api:4.0.2"
  }
```

### Others

At first generate the JAR by executing:

```shell
mvn clean package
```

Then manually install the following JARs:

* `target/pdf-generator-api-4.0.2.jar`
* `target/lib/*.jar`

## Getting Started

Please follow the [installation](#installation) instruction and execute the following Java code:

```java

// Import classes:
import org.pdfgeneratorapi.client.ApiClient;
import org.pdfgeneratorapi.client.ApiException;
import org.pdfgeneratorapi.client.Configuration;
import org.pdfgeneratorapi.client.auth.*;
import org.pdfgeneratorapi.client.models.*;
import org.pdfgeneratorapi.client.api.ConversionApi;

public class Example {
  public static void main(String[] args) {
    ApiClient defaultClient = Configuration.getDefaultApiClient();
    defaultClient.setBasePath("https://us1.pdfgeneratorapi.com/api/v4");
    
    // Configure HTTP bearer authorization: JSONWebTokenAuth
    HttpBearerAuth JSONWebTokenAuth = (HttpBearerAuth) defaultClient.getAuthentication("JSONWebTokenAuth");
    JSONWebTokenAuth.setBearerToken("BEARER TOKEN");

    ConversionApi apiInstance = new ConversionApi(defaultClient);
    ConvertHTML2PDFRequest convertHTML2PDFRequest = new ConvertHTML2PDFRequest(); // ConvertHTML2PDFRequest | 
    try {
      GenerateDocument200Response result = apiInstance.convertHTML2PDF(convertHTML2PDFRequest);
      System.out.println(result);
    } catch (ApiException e) {
      System.err.println("Exception when calling ConversionApi#convertHTML2PDF");
      System.err.println("Status code: " + e.getCode());
      System.err.println("Reason: " + e.getResponseBody());
      System.err.println("Response headers: " + e.getResponseHeaders());
      e.printStackTrace();
    }
  }
}

```

## Documentation for API Endpoints

All URIs are relative to *https://us1.pdfgeneratorapi.com/api/v4*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*ConversionApi* | [**convertHTML2PDF**](docs/ConversionApi.md#convertHTML2PDF) | **POST** /conversion/html2pdf | HTML to PDF
*ConversionApi* | [**convertURL2PDF**](docs/ConversionApi.md#convertURL2PDF) | **POST** /conversion/url2pdf | URL to PDF
*DocumentsApi* | [**generateDocument**](docs/DocumentsApi.md#generateDocument) | **POST** /documents/generate | Generate document
*DocumentsApi* | [**generateDocumentAsynchronous**](docs/DocumentsApi.md#generateDocumentAsynchronous) | **POST** /documents/generate/async | Generate document (async)
*DocumentsApi* | [**generateDocumentBatch**](docs/DocumentsApi.md#generateDocumentBatch) | **POST** /documents/generate/batch | Generate document (batch)
*DocumentsApi* | [**generateDocumentBatchAsynchronous**](docs/DocumentsApi.md#generateDocumentBatchAsynchronous) | **POST** /documents/generate/batch/async | Generate document (batch + async)
*DocumentsApi* | [**getDocuments**](docs/DocumentsApi.md#getDocuments) | **GET** /documents | Get documents
*TemplatesApi* | [**copyTemplate**](docs/TemplatesApi.md#copyTemplate) | **POST** /templates/{templateId}/copy | Copy template
*TemplatesApi* | [**createTemplate**](docs/TemplatesApi.md#createTemplate) | **POST** /templates | Create template
*TemplatesApi* | [**deleteTemplate**](docs/TemplatesApi.md#deleteTemplate) | **DELETE** /templates/{templateId} | Delete template
*TemplatesApi* | [**getTemplate**](docs/TemplatesApi.md#getTemplate) | **GET** /templates/{templateId} | Get template
*TemplatesApi* | [**getTemplateData**](docs/TemplatesApi.md#getTemplateData) | **GET** /templates/{templateId}/data | Get template data fields
*TemplatesApi* | [**getTemplates**](docs/TemplatesApi.md#getTemplates) | **GET** /templates | Get templates
*TemplatesApi* | [**openEditor**](docs/TemplatesApi.md#openEditor) | **POST** /templates/{templateId}/editor | Open editor
*TemplatesApi* | [**updateTemplate**](docs/TemplatesApi.md#updateTemplate) | **PUT** /templates/{templateId} | Update template
*WorkspacesApi* | [**createWorkspace**](docs/WorkspacesApi.md#createWorkspace) | **POST** /workspaces | Create workspace
*WorkspacesApi* | [**deleteWorkspace**](docs/WorkspacesApi.md#deleteWorkspace) | **DELETE** /workspaces/{workspaceIdentifier} | Delete workspace
*WorkspacesApi* | [**getWorkspace**](docs/WorkspacesApi.md#getWorkspace) | **GET** /workspaces/{workspaceIdentifier} | Get workspace
*WorkspacesApi* | [**getWorkspaces**](docs/WorkspacesApi.md#getWorkspaces) | **GET** /workspaces | Get workspaces


## Documentation for Models

 - [AsyncOutputParam](docs/AsyncOutputParam.md)
 - [CallbackParam](docs/CallbackParam.md)
 - [Component](docs/Component.md)
 - [ConvertHTML2PDFRequest](docs/ConvertHTML2PDFRequest.md)
 - [ConvertURL2PDFRequest](docs/ConvertURL2PDFRequest.md)
 - [CopyTemplateRequest](docs/CopyTemplateRequest.md)
 - [CreateTemplate200Response](docs/CreateTemplate200Response.md)
 - [CreateWorkspace200Response](docs/CreateWorkspace200Response.md)
 - [CreateWorkspaceRequest](docs/CreateWorkspaceRequest.md)
 - [DataBatchInner](docs/DataBatchInner.md)
 - [DeleteTemplate204Response](docs/DeleteTemplate204Response.md)
 - [DeleteTemplate204ResponseResponse](docs/DeleteTemplate204ResponseResponse.md)
 - [Document](docs/Document.md)
 - [FormatParam](docs/FormatParam.md)
 - [GenerateDocument200Response](docs/GenerateDocument200Response.md)
 - [GenerateDocument200ResponseMeta](docs/GenerateDocument200ResponseMeta.md)
 - [GenerateDocumentAsynchronous200Response](docs/GenerateDocumentAsynchronous200Response.md)
 - [GenerateDocumentAsynchronous200ResponseResponse](docs/GenerateDocumentAsynchronous200ResponseResponse.md)
 - [GenerateDocumentAsynchronousRequest](docs/GenerateDocumentAsynchronousRequest.md)
 - [GenerateDocumentBatchAsynchronousRequest](docs/GenerateDocumentBatchAsynchronousRequest.md)
 - [GenerateDocumentBatchRequest](docs/GenerateDocumentBatchRequest.md)
 - [GenerateDocumentRequest](docs/GenerateDocumentRequest.md)
 - [GetDocuments200Response](docs/GetDocuments200Response.md)
 - [GetTemplateData200Response](docs/GetTemplateData200Response.md)
 - [GetTemplates200Response](docs/GetTemplates200Response.md)
 - [GetTemplates401Response](docs/GetTemplates401Response.md)
 - [GetTemplates402Response](docs/GetTemplates402Response.md)
 - [GetTemplates403Response](docs/GetTemplates403Response.md)
 - [GetTemplates404Response](docs/GetTemplates404Response.md)
 - [GetTemplates422Response](docs/GetTemplates422Response.md)
 - [GetTemplates429Response](docs/GetTemplates429Response.md)
 - [GetTemplates500Response](docs/GetTemplates500Response.md)
 - [GetWorkspaces200Response](docs/GetWorkspaces200Response.md)
 - [OpenEditor200Response](docs/OpenEditor200Response.md)
 - [OpenEditorRequest](docs/OpenEditorRequest.md)
 - [OpenEditorRequestData](docs/OpenEditorRequestData.md)
 - [OutputParam](docs/OutputParam.md)
 - [PaginationMeta](docs/PaginationMeta.md)
 - [Template](docs/Template.md)
 - [TemplateDefinition](docs/TemplateDefinition.md)
 - [TemplateDefinitionDataSettings](docs/TemplateDefinitionDataSettings.md)
 - [TemplateDefinitionEditor](docs/TemplateDefinitionEditor.md)
 - [TemplateDefinitionNew](docs/TemplateDefinitionNew.md)
 - [TemplateDefinitionNewLayout](docs/TemplateDefinitionNewLayout.md)
 - [TemplateDefinitionNewLayoutMargins](docs/TemplateDefinitionNewLayoutMargins.md)
 - [TemplateDefinitionNewLayoutRepeatLayout](docs/TemplateDefinitionNewLayoutRepeatLayout.md)
 - [TemplateDefinitionNewPagesInner](docs/TemplateDefinitionNewPagesInner.md)
 - [TemplateDefinitionNewPagesInnerMargins](docs/TemplateDefinitionNewPagesInnerMargins.md)
 - [TemplateParam](docs/TemplateParam.md)
 - [TemplateParamData](docs/TemplateParamData.md)
 - [Workspace](docs/Workspace.md)


## Documentation for Authorization

Authentication schemes defined for the API:
### JSONWebTokenAuth

- **Type**: HTTP basic authentication


## Recommendation

It's recommended to create an instance of `ApiClient` per thread in a multithreaded environment to avoid any potential issues.

## Author

support@pdfgeneratorapi.com

