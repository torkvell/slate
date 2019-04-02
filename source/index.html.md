---
title: Usabilla API Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: Node.js
  - python: Python
  - go: Go
  - php: PHP

toc_footers:
  - <a href="mailto:support@usabilla.com">Contact Usabilla Support</a>
  - <a href="https://app.usabilla.com/">Usabilla Login Portal</a>

search: true
---

# Introduction

Welcome to the Usabilla API documentation. Here you will find a detailed guide on how to access the data related to your customers feedback.

You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right. On top of this, we've also included our
client repositories we have on GitHub.

Usabilla maintains the following client libraries:

* [Usabilla API PHP Client](https://github.com/usabilla/api-php)
* [Usabilla API Python Client](https://github.com/usabilla/api-python)
* [Usabilla API Go Client](https://github.com/usabilla/api-go)
* [Usabilla API Node.js Client](https://github.com/usabilla/api-js-node)

Client libraries by third parties:

* [Usabilla API Ruby On Rails Client - maintained by Jon Molinaro (CareerBuilder.com)](https://github.com/JMolinaro/usabilla_api)
* [Usabilla API Shell Script - maintained by Alexander Vellekoop (Aegon.nl)](https://github.com/LexerZz/usabilla_api/blob/master/usabilla_api.sh)

If you wrote a public client library that would be beneficial to fellow coders, we would love to hear about it and include it in our documentation, please send us a link to your project repository.

If you prefer to use a language we do not provide a client library for you can continue to the next section to see how our authentication method works. Please contact [support](mailto:support@usabilla.com) if you need help calling our API.

# Authentication

> The authentication code for each language can be found under the respective GitHub pages:<br>
[GitHub - PHP Client](https://github.com/usabilla/api-php)<br>
[GitHub - Python Client](https://github.com/usabilla/api-python)<br>
[GitHub - Go Client](https://github.com/usabilla/api-go)<br>
[GitHub - Node.js Client](https://github.com/usabilla/api-js-node)<br>
[GitHub - Ruby On Rails Client](https://github.com/JMolinaro/usabilla_api)<br>
[GitHub - Shell Script](https://github.com/LexerZz/usabilla_api/blob/master/usabilla_api.sh)<br>

The credentials consist in an access and a secret key used for authentication purposes. You can acquire the credentials by logging into your Usabilla account and access the following [link](https://app.usabilla.com/member/account/settings#public_api) or navigating to the settings menu under the tab Usabilla API. If you're new to the Usabilla API then we recommend you to read through our support article on [Getting Started with the Usabilla Public API](https://support.usabilla.com/hc/en-us/articles/360019493372-Getting-Started-with-the-Usabilla-Public-API).
Our API authentication system is inspired by the [AWS version 4](http://docs.aws.amazon.com/general/latest/gr/sigv4_signing.html) request signing process.

1. [Create a Canonical Request](#1-create-a-canonical-request)
2. [Create a String to Sign](#2-create-a-string-to-sign)
3. [Calculate the Signature](#3-calculate-the-signature)
4. [Signing the Request](#4-add-the-signing-information-to-the-request)


## 1. Create a Canonical Request

To begin the signing process, you create a string that includes information from your request in a standardized (canonical) format. This step will allow you to create a canonical request string which will be used by our servers to verify the signed request.

Follow the steps here to create a canonical version of the request. Otherwise, your version and the version calculated by our servers won't match, and the request will be denied.

The following example shows the pseudocode to create a canonical request:

<code>
CanonicalRequest =<br>
HTTPRequestMethod + '\n' + <br>
CanonicalURI + '\n' + <br>
CanonicalQueryString + '\n' + <br>
CanonicalHeaders + '\n' + <br>
SignedHeaders + '\n' + <br>
HexEncode(Hash(RequestPayload))
</code>

**Example request**

To show you how to construct the canonical form of an HTTP request, we'll use the following example request.
The original request might look like this as it is sent from the client to our servers, except that this example does not include the signing information yet.

<code>
    GET https://data.usabilla.com/ HTTP/1.1<br>
    Host: data.usabilla.com<br>
    Date: 20150830T123600Z
</code>

**To create a canonical request, concatenate the following components from each step into a single string:**

1.  Start with the GET, HTTP request method written in upper case, followed by a newline character.
    Example request method:

    <code>
        GET
    </code>

2.    Add the canonical URI parameter, followed by a newline character.

    If the absolute path is empty, use a forward slash (/). In the example request, nothing follows the host in the URI, so the absolute path is empty.

    **Example canonical URI:**

    <code>
        /button
    </code>

3.  Add the canonical query string, followed by a newline character.

    If the request does not include a query string, use an empty string (essentially, a blank line). The example request does not contain a query string:

    **Example canonical query string:**

    <code>
        ?limit=1&since=123456789
    </code>

    To construct the canonical query string, complete the following steps:

    a. URI-encode each parameter name and value according to the following rules:
    
    *Do not URL-encode any of the unreserved characters that RFC 3986 defines: A-Z, a-z, 0-9, hyphen ( - ), underscore ( \_ ), period ( . ), and tilde ( ~ ).

    *Percent-encode all other characters with %XY, where X and Y are hexadecimal characters (0-9 and uppercase A-F).
      
    b. Sort the encoded parameter names by character code.

    c. Build the canonical query string by starting with the first parameter name in the sorted list.

    d. For each parameter, append the URI-encoded parameter name, followed by the character '=' (ASCII code 61), followed by the URI-encoded parameter value. Use an empty string for parameters that have no value.

    e. Append the character '&' (ASCII code 38) after each parameter value except for the last value in the list.

    **Example authentication parameters in a query string:**

    The following example shows a query string that includes authentication information.

    <code>
    Algorithm=USBL1-HMAC-SHA256&<br>
    Version=2010-05-08&<br>
    Credential=AKIAIOSFODNN7EXAMPLE%2F20150112%2Fusbl1_request&<br>
    Date=20150112T135139Z&<br>
    SignedHeaders=date%3Bhost
    </code>

4.  Add the canonical headers, followed by a newline character. The canonical headers consist of a list of all the HTTP headers. The required headers are host and date.

    **Example canonical headers:**

    <code>
    host:data.usabilla.com\n<br>
    date:20150111T003735Z\n<br>
    </code>

    To create the canonical headers list, convert all header names to lowercase and trim excess white space characters out of the header values. When you trim, remove leading spaces and trailing spaces, and convert sequential spaces in the value to a single space. However, do not remove extra spaces from any values that are inside quotation marks.

    The following pseudocode describes how to construct the canonical list of headers:

    <code>
    CanonicalHeaders =<br>
    CanonicalHeadersEntry0 + CanonicalHeadersEntry1 + ... +<br>
    CanonicalHeadersEntryN <br>
    CanonicalHeadersEntry =<br>
    Lowercase(HeaderName) + ':' + Trimall(HeaderValue) + '\n'
    </code>

    Lowercase represents a function that converts all characters to lowercase. The Trimall function removes excess white space before and after values and from inside non-quoted strings, per <a href="https://www.ietf.org/rfc/rfc2616.txt">RFC 2616</a> Section 4.2.

    Build the canonical headers list by iterating through the collection of header names and sorting the headers lowercase character code. Construct each header by using the following rules:

        Append the lowercase header name followed by a colon.

        Append a comma-separated list of values for that header. If there are duplicate headers, the values are comma-separated. Do not sort the values in headers that have multiple values.

        Append a new line ('\n').

    The following two samples compare a more complex set of headers with their canonical form:

    **Example original headers:**

    <code>
    host:data.usabilla.com\n<br>
    date:20150111T003735Z\n
    </code>

    **Example canonical form:**

    <code>
    date:20150111T003735Z\n<br>
    host:data.usabilla.com\n
    </code>

5.  To create the signed headers list, convert all header names to lowercase, sort them by character code, and use a semicolon to separate the header names. The following pseudocode describes how to construct a list of signed headers. LOWERCASE represents a function that converts all characters to lowercase.

     <code>
    SignedHeaders =<br>
    Lowercase(HeaderName0) + ';' + Lowercase(HeaderName1) + ";" <br> + ... + Lowercase(HeaderNameN)
    </code>

    Build the signed headers list by iterating through the collection of header names, sorted by lowercase character code. For each header name except the last, append a semicolon (';') to the header name to separate it from the following header name.

    Only the host and date headers are required. Optionally if the users have added additional headers to the CanonicalHeaders these will also be required. Note: Certain API calls require additional headers to be present in the signed headers list.

    **Example request: Signed headers**

    <code>
    host;date
    </code>

6.  Use the hash (digest) function SHA256 and create a hashed value from the RequestPayload in the body of the HTTP or HTTPS request. Since none of the current API calls support a request body the RequestPayload must always be the empty string. Therefore the RequestPayload will always be the hexadecimal SHA-256 hash of an empty string.

    <code>
    HashedPayload = Lowercase(HexEncode(Hash(requestPayload)))
    </code>

    **Example request: RequestPayload**

    <code>
    empty string
    </code>

    **Example request: Hashed RequestPayload**

    The following example shows the result of using SHA-256 to hash the example RequestPayload.

    <code>
    3511de7e95d28ecd39e9513b642aee07e54f4941150d8df8bf94b328ef7e55e2
    </code>

7.  To construct the finished canonical request, combine all the components from each step as a single string. As noted, each component ends with a newline character. If you follow the canonical request pseudocode explained earlier, the resulting canonical request is shown in the following example:

    **Example canonical request:**

    <code>
    GET<br>
    /live/website/button<br>
    date:Mon, 12 Jan 2015 13:57:54 GMT<br>
    host:data.usabilla.dev<br>
    date;host<br>
    3511de7e95d28ecd39e9513b642aee07e54f4941150d8df8bf94b328ef7e55e2
    </code>

8.  Create a digest (hash) of the canonical request by using the same algorithm that you used to hash the RequestPayload. The hashed canonical request must be represented as a string of lowercase hexademical characters.

    **Example hashed canonical request:**

    <code>
    3511de7e95d28ecd39e9513b642aee07e54f4941150d8df8bf94b328ef7e55e2
    </code>

## 2. Create a String to Sign

The string to sign includes meta information about your request and about the canonical request. To create the string to sign, concatenate the algorithm, date, credential scope, and the digest of the canonical request, as shown in the following pseudocode:

**Structure of the String to Sign:**

<code>
StringToSign  = <br>
Algorithm + <br>
RequestDate + <br>
CredentialScope + <br>
HashedCanonicalRequest<br>
</code>

**Example HTTPS request:**

<code>
    GET<br>
    /live/website/button<br>
    date:Mon, 12 Jan 2015 13:57:54 GMT<br>
    host:data.usabilla.dev<br>
    date;host<br>
    3511de7e95d28ecd39e9513b642aee07e54f4941150d8df8bf94b328ef7e55e2
</code>

**To create the string to sign:**

1.  Start with the algorithm designation, followed by a newline character. This value is the hashing algorithm that you're using to calculate the digests in the canonical request.

    <code>
    USBL1-HMAC-SHA256\n
    </code>

2.  Append the request date value, followed by a newline character. The date is specified by using the ISO8601 Basic format via the Date header in the YYYYMMDD'T'HHMMSS'Z' format. This value must match the value you used in any previous steps.

    <code>
    20150112T135139Z\n
    </code>

3.  Append the credential scope value, followed by a newline character. This value is a string that includes the date and termination string ("usbl1_request") in lowercase characters.

    <code>
    20150112/usbl1_request\n
    </code>

    The date must be in the YYYYMMDD format. Note that the date does not include a time value.

4.  Append the hash of the canonical request that you created in task 1; [Create a canonical Request](#1-create-a-canonical-request). This value is not followed by a newline character. The hashed canonical request must be lowercase base-16 encoded, as defined by [Section 8 of RFC 4648](https://tools.ietf.org/html/rfc4648#section-8).

    <code>
    daf6b02d8ae6809bf3b2445c480c1ee6038da1a19a2bd5ad21b7a30acd0fdec2
    </code>

**Example string to sign:**

<code>
    USBL1-HMAC-SHA256<br>
    20150112T135139Z<br>
    20150112/usbl1_request<br>
    daf6b02d8ae6809bf3b2445c480c1ee6038da1a19a2bd5ad21b7a30acd0fdec2
</code>

## 3. Calculate the Signature

Before you calculate a signature, you derive a signing key from your Usabilla secret key.

**To calculate a signature:**

1.  Derive your signing key. To do this, you use your secret access key to create a series of hash-based message authentication codes (HMACs) as shown by the following pseudocode, where HMAC(key, data) represents an HMAC-SHA256 function that returns output in binary format. The result of each hash function becomes input for the next one.

    **Pseudocode for deriving a signing key:**

    <code>kSecret = Your Usabilla Secret Key<br>
    kDate = HMAC("USBL1" + kSecret, Date)<br>
    kSigning = HMAC(kDate, "usbl1_request")</code>

    Use the digest for the key derivation. Most languages have functions to compute either a binary format hash, commonly called a digest, or a hex-encoded hash, called a hexdigest. The key derivation requires that you use a digest.

    **Example inputs:**

    <code>HMAC(HMAC("USBL1" + kSecret,"20110909"),"usbl1_request")</code>

    The following example shows the derived signing key that results from this sequence of HMAC hash operations, using the inputs shown in the previous example. This representation shows the digit representation of each byte in the binary derived key.

    **Example signing key:**

    <code>152 241 216 137 254 196 244 66 26 220 82 43 171 12 225 248 46 105 41 194 98 237 21 229 169 76 144 239 209 227 176 231</code>

2.  Calculate the signature. To do this, use the signing key that you derived and the string to sign as inputs to the keyed hash function. After you've calculated the signature as a digest, convert the binary value to a hexadecimal representation.

    **Pseudocode for calculating the signature:**

    <code>signature = HexEncode(HMAC(derived-signing-key, string-to-sign))</code>

    **Note:** Make sure you specify the HMAC parameters in the correct order for the programming language you are using. This example shows the key as the first parameter and the data (message) as the second parameter, but the function that you use might specify the key and data in a different order.

    **Example signature:**

    <code>ced6826de92d2bdeed8f846f0bf508e8559e98e4b0199114b84c54174deb456c</code>

## 4. Signing the Request

After you have calculated the signature, add it to the Authorization header of your request.

The following pseudocode shows the construction of the Authorization header:

<code>Authorization: algorithm <br>
Credential=access key ID/credential scope, <br>
SignedHeaders=SignedHeaders, <br>
Signature=signature</code>

The following example shows a finished Authorization header:

<code>Authorization: USBL1-HMAC-SHA256 <br>
Credential=AccessKey/20130524/usbl1_request,<br>
SignedHeaders=date;host;user-agent,<br>
Signature=fe5f80f77d5fa3beca038a248ff027d0<br>
445342fe2855ddc963176630326f1024</code>

**Note the following:**

- There is a space between the algorithm and Credential, not a comma. On the other hand, the SignedHeaders and Signature values are separated from the earlier values using a comma.

- The Credential value starts with the access key, which is prefixed with a slash (/) onto the credential scope value that you calculated earlier. In contrast, the secret access key is used to derive the signing key for the signature, but is not included in the signing information that's sent in the request.

#Request URL structure

`https://data.usabilla.com/live/<product-name>/<resource-type>/<id>/<nested-resource>?params`

### Parameters

| Field  | Type   | Description            |
| ------ | ------ | ---------------------- |
| id     | String | The id of the resource |
| params | String | Query URL parameters   |

### Success 200

```json
Example Success 200 Response

HTTP/1.1 200 OK

{
  "items": [
    ...
  ],
  "count": 0,
  "hasMore": false,
  "lastTimestamp": 1419340238645
}
```

| Field         | Type      | Description                                                                                                                         |
| ------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| items         | Array     | An array containing items of the requested resource type                                                                            |
| count         | Integer   | The number of items returned                                                                                                        |
| hasMore       | Integer   | A flag that indicates whether more items can be requested                                                                           |
| lastTimestamp | Timestamp | The timestamp of the last item in the <code>items</code> array. Can be used to request more items when <code>hasMore</code> is true |

# Errors

###403

```json
HTTP/1.1 403 Bad Request
{
  "error": "Invalid client key"
}

```

| Field                 | Description                              |
| --------------------- | ---------------------------------------- |
| DateError             | Date is not valid                        |
| SignatureDoesNotMatch | The signature does not match the content |
| SignatureMissing      | Request doesn't have a signature         |
| TokenMismatch         | Token not found                          |
| HostError             | Requested host is not valid              |

###404

| Field    | Description |
| -------- | ----------- |
| NotFound | Invalid URL |

###429

| Field             | Description                                                         |
| ----------------- | ------------------------------------------------------------------- |
| TooManyRequests   | Request limit reached, please contact your Usabilla account manager |
| RateLimitExceeded | Request rate too high                                               |

###500

```json
HTTP/1.1 500 Server Error
{
  "error": "The server has encountered an error."
}

```

| Field                | Description                    |
| -------------------- | ------------------------------ |
| DatabaseError        | Account Provider not available |
| WrongCodec           | Signed algorithm is not valid  |
| QueryFailed          | Could not query                |
| AuthenticationFailed | Please re-authenticate         |

# Usabilla for Websites

## Feedback Button

This section covers the resources that can be requested regarding Usabilla's Feedback buttons. The feedback button is a button widget that is shown on your website at a visible but non-obtrusive location to enable visitors to leave feedback on their experience.

The authorization code for each programming language can be found under the [corresponding GitHub page](#introduction) we have for each client library.

##  Get Buttons

```javascript
/* eslint-disable no-console */
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all buttons for this account.
usabilla.websites.buttons
  .get()
  .then(buttons => {
    const button = buttons[0];
    if (!button) {
      return;
    }
  })
  .catch(reason => {
    // If the usabilla call fails, we want to see the error message
    console.error(reason);
  });
```

```python
import usabilla as ub

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Get all buttons for this account
    buttons = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_WEBSITES,api.RESOURCE_BUTTON)

    # Print first button
    first_button = buttons['items'][0]
    print first_button['name']
```

```php
// Under construction
```

```go
package main

import (
    "os"
    "fmt"
    "github.com/usabilla/api-go"
)

func main() {
    key := os.Getenv("USABILLA_API_KEY")
    secret := os.Getenv("USABILLA_API_SECRET")

    // Pass the key and secret which should be defined as ENV vars
    usabilla := usabilla.New(key, secret, nil)

    resource := usabilla.Buttons()

    // Get the first ten buttons
    params := map[string]string{"limit": "10"}
    buttons, _ := resource.Get(params)
}
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "8d73568ac2be",
  "name": "My website feedback button"
}
```
### HTTPS Request

`https://data.usabilla.com/live/websites/button`

This request returns all your feedback buttons in JSON format as shown to the right.

###Request example

`https://data.usabilla.com/live/websites/button`

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier for a button. 12 characters long.                                 |
| name      | String  | The name of the button given when created. Can be updated in the button setup page. |

##  Get Feedback Items

```javascript
/* eslint-disable no-console */
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);
    // Use a button id to get feedback for a specific button.
    let buttonFeedbackQuery = {
      id: button.id,
      params: {
        limit: 10
      }
    };
    usabilla.websites.buttons.feedback
      .get(buttonFeedbackQuery)
      .then(feedback => {
        console.log('# button feedback', feedback.length);
      })
      .catch(reason => {
        console.error(reason);
      });
  })
  .catch(reason => {
    // If the usabilla call fails, we want to see the error message
    console.error(reason);
  });
```

```python
import usabilla as ub

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Get the feedback items from a specific button
    feedback_items = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_WEBSITES, api.RESOURCE_FEEDBACK, 'button_id', iterate=True)
    print len([item for item in feedback_items])
```

```php
require 'vendor/autoload.php';

use Usabilla\API\Client\UsabillaClient;

// Creates a new Usabilla client.
$client = new UsabillaClient('ACCESS_KEY', 'SECRET_KEY');

// Specify the parameters as mentioned in the service description.
$params = ['id' => 'BUTTON_ID', 'limit' => 5];

// Gets the command.
$command = $client->getCommand('GetWebsiteFeedbackItems', $params);

// Executes the command.
$response = $client->execute($command);

// Prints the received response.
print_r($response);
```

```go
package main

import (
    "os"
    "fmt"
    "github.com/usabilla/api-go"
)

func main() {
    key := os.Getenv("USABILLA_API_KEY")
    secret := os.Getenv("USABILLA_API_SECRET")

    // Pass the key and secret which should be defined as ENV vars
    usabilla := usabilla.New(key, secret, nil)

    resource := usabilla.Buttons()

    // Get the first ten buttons
    params := map[string]string{"limit": "10"}
    buttons, _ := resource.Get(params)

    // Print all feedback items for each button
    for _, button := range buttons.Items {
        feedback, _ := resource.Feedback().Get(button.ID, nil)
        fmt.Printf("Feedback for button: %s\n%v\n", button.ID, feedback.Items)
    }
}
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "buttonId": "07f5ec0c3b93",
  "comment": "asd",
  "custom": null,
  "date": "2015-04-28T14:48:39.289Z",
  "email": "",
  "id": "553f9dc7d350221532dfa715",
  "image": "",
  "labels": [],
  "location": "Netherlands",
  "nps": 10,
  "publicUrl": "https://www.usabilla.com/feedback/item/538ce51b93804a7f568324fbef81e6559743ba7e",
  "rating": 5,
  "tags": [],
  "url": "https://staging.usabilla.com/",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36"
}
```

After a visitor leaves feedback on a button it becomes a feedback item that consists of several data fields.

###HTTPS Request

`https://data.usabilla.com/live/websites/button/<id>/feedback`

###Request example

`https://data.usabilla.com/live/websites/button/8d73568ac2be/feedback/?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | The button id. If you wish to get all available feedback items use * (asterisk) as value.                       |
| limit      | Integer  | Optional URL query parameter that limits the number of retrieved results between 1 and 100. Default value is 100. |
| since      | Timestamp  | [Optional] UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier of the feedback item.                       |
|userAgent	|String | Information about the browser's user-agent.|
|comment	|String	|Commentary left by the user in the feedback form.|
|commentTranslated|	String	|The translated version of the commentary left by the user.|
|commentTranslatedFrom|	String	| The language from which the translation was made.|
|location|	String	|String containing geographical information about the location of the user based on the user's ip address.|
|date	| Date	|The creation date of the feedback item, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)|
|custom	|Map	|Custom variables that have been assigned in the button setup page.|
|email	|String	|Optional field that the user fills in in case he wishes to be contacted.|
|image|	String	|The screenshot of the item that has been selected to give feedback for.|
|labels	|Map	|An array of labels that have been assigned to the feedback item.|
|nps	|Integer	|The Net Promoter Score given by the user (0-10).|
|publicUrl	|String	|When the owner of the button has publicized the feedback item it becomes publicly accesible through this URL.|
|rating	|Integer	|Rating given by the user (0-5).|
|buttonId	|String	|The source button of the feedback item.|
|tags	|Array	|An array of tags assigned to the feedback item.|
|url	|Integer|The link through which the source button can be accessed in the backend.|

## Campaigns

This section covers how to retrieve data for Usabilla Campaigns. These surveys are aimed towards web users with specific targeting options set up. The survey will trigger without the users intent, actively asking for feedback.

##  Get Campaigns

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all campaigns
usabilla.websites.campaigns
  .get()
  .then(campaigns => {
    const campaign = campaigns[0];

    console.log('# campaigns', campaigns.length);

    if (!campaign) {
      return;
    }
```

```python
import usabilla as ub

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Get all campaigns
    campaigns = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_WEBSITES, api.RESOURCE_CAMPAIGN)
    first_campaign = campaigns['items'][0]
    print first_campaign['name']
```

```php
//Under construction
```

```go
//Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "5499612ec4698839368b4573",
  "date": "2014-01-15T19:48:06.003Z",
  "buttonId": "8d73568ac2be",
  "analyticsId": "901f1d832a55",
  "status": "active",
  "name": "My campaign",
  "type": "boost",
  "url": ""
}
```
### HTTPS Request

`https://data.usabilla.com/live/websites/campaign`

This request returns a list of your Usabilla for Websites Campaigns.

###Request example

`https://data.usabilla.com/live/websites/campaign?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| limit        | Integer  | Optional. URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional. UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier for a campaign.                     |
|date	|Date | The creation date of the campaign, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)|
|buttonId	|String	|The source button that triggers the campaign.|
|analyticsId|	String	|Analytics ID.|
|status|	String	| The status of the campaign. Possible values are <code>created, active, paused and finished</code>.|
|name|	String	|The name of the campaign given when created. Can be updated in the campaign setup page.|
|type	| String	|The type of the campaign.|
|url	| String	|The url of the campaign when the campaign is of type Survey, empty otherwise.|

##  Get Campaign Results

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Specify a campaign id
let campaignQuery = {
    id: campaign.id
};

// Get the responses from a campaign with the specific id
usabilla.websites.campaigns.results
    .get(campaignQuery)
    .then(responses => {
    console.log('# web campaign responses', responses.length);
    })
    .catch(reason => {
    console.error(reason);
    });
```

```python
import usabilla as ub

if __name__ == '__main__':

# Get the responses of a specific campaign
    responses = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_WEBSITES, api.RESOURCE_CAMPAIGN_RESULT, <CAMPAIGN_ID>, iterate=True)
    print len([item for item in responses])
```

```go
//Under construction
```

```php
//Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "549972d5c469885e548b4570",
  "campaignId": "5499612ec4698839368b4573",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36",
  "location": "Amsterdam, Netherlands",
  "date": "2014-01-15T19:48:06.003Z",
  "customData": {
    "form_name": "form1"
  },
  "data": {
    "text": "test"
  },
  "time": 5000,
  "url": "https://usabilla.com"
}
```
### HTTPS Request

`https://data.usabilla.com/live/websites/campaign/<id>/results`

This request returns the responses of a campaign with a specific id.

###Request example

`https://data.usabilla.com/live/websites/campaign/5499612ec4698839368b4573/results?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
|id | String | The campaign id. Use <code>*</code> (asterisk) to request feedback of all campaigns.|
| limit        | Integer  | Optional. URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional. UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier for a campaign result.                     |
|date	|Date | The creation date of the campaign result item, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)|
|campaignId	|String	|Unique identifier for the campaign it belongs to.|
|userAgent  |String	|Information about the browser user agent.|
|url  |String|Origin URL where the campaign result was registered.|
|customData  |Map|Custom variables that have been assigned in the campaign setup page.|
|data|	Array	| An array containing the values of the campaign form fields.|
|time|	Integer	| The amount of time the user has taken to complete the survey. Expressed in miliseconds.|
|location|	String	|String containing geographical information about the location of the user based on the user's IP address.|

##  Get Campaign Statistics

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get the stats of the first campaign
usabilla.websites.campaigns.stats
    .get(campaignQuery)
    .then(stats => {
    console.log('# web campaign stats', stats);
    })
    .catch(reason => {
    console.error(reason);
    });
```

```python
import usabilla as ub

if __name__ == '__main__':
# Get the stats of the first campaign
    stats = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_WEBSITES, api.RESOURCE_CAMPAIGN_STATS, first_campaign['id'])
    print stats
```

```go
//Under construction
```

```php
//Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "549972d5c469885e548b4570",
  "completed": 5,
  "conversion": 83,
  "views": 6
}
```
### HTTPS Request

`https://data.usabilla.com/live/websites/campaign/<id>/stats`

This request returns the statistics of a campaign with a specific id.

###Request example

`https://data.usabilla.com/live/websites/campaign/5499612ec4698839368b4573/results?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
|id | String | The campaign id. Use <code>*</code> (asterisk) to request feedback of all campaigns.|
| limit        | Integer  | Optional. URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional. UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier for a campaign result.                     |
| completed        | Integer  | The amount of completed campaign forms that were submitted.                    |
| views       | Integer  | The amount of views the campaign form got by counting how many times it was opened.  |
| conversion       | Integer  | The percentage of conversion (views to completion).  |

## In-Page

This section is regarding the Usabilla In-Page product and how to retrieve the data. The In-Page widget can be placed anywhere on your webpage for users to give valuable feedback on the web content.

##  Get In-Page Widgets

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all inpage widgets for this account.
usabilla.websites.inpage.get()
```

```python
# Under construction
```

```go
// Under construction
```

```php
// Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "5499612ec4698839368b4573",
  "date": "2014-01-15T19:48:06.003Z",
  "name": "My In-Page widget"
}
```
### HTTPS Request

`https://data.usabilla.com/live/websites/inpage`

This request returns a list of your Usabilla for Websites in-page widgets.

###Request example

`https://data.usabilla.com/live/websites/inpage?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| limit        | Integer  | Optional. URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional. UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier for an in-page widget.                   |
| date        | Date  | The creation date of the in-page widget, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)        |
| name       | String | The name of the In-Page widget given when created. Can be updated in the In-Page widget setup page.  |

##  Get In-Page Feedback

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all inpage widgets for this account.
usabilla.websites.inpage
  .get()
  .then(inPageWidgets => {
    // Get the feedback for an inpage widget with id.
    let inPageQuery = {
      id: inPageWidgets[0].id
    };

    // Get the feedback of the first inpage widget
    usabilla.websites.inpage.feedback
      .get(inPageQuery)
      .then(feedback => {
        console.log('# inpage feedback', feedback.length);
      })
      .catch(reason => {
        console.error(reason);
      });
  })
  .catch(reason => {
    // If the Usabilla call fails, we want to see the error message
    console.error(reason);
  });
```

```python
# Under construction
```

```go
// Under construction
```

```php
// Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "549972d5c469885e548b4570",
  "date": "2014-01-15T19:48:06.003Z",
  "widgetId": "5499612ec4698839368b4573",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36",
  "location": "Amsterdam, Netherlands",
  "geo": {
    "city": "Schoten",
    "country": "BE",
    "region": "01"
  },
  "comment": "Very nice indeed",
  "data": {
    "Did_you_enjoy_th": 5
  },
  "customData": {
    "form_name": "form1"
  },
  "mood": 3,
  "rating": 0.5,
  "nps": 5,
  "url": "https://usabilla.com"
}
```
### HTTPS Request

`https://data.usabilla.com/live/websites/inpage/<id>/feedback`

This request returns a list of your Usabilla for Websites in-page widgets.

###Request example

`https://data.usabilla.com/live/websites/inpage/5499612ec4698839368b4573/feedback?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String | The in-page widget id. Use <code>*</code> (asterisk) to request feedback of all in-page widgets.                                |
| limit        | Integer  | Optional. URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional. UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier for the In-Page widget feedback item.                  |
| date        | Date  | The creation date of the in-page widget, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)        |
| data       | Array | An array containing the values of the In-Page widget form fields.  |
| customData      | Array | An array containing the values of the widget's custom fields.  |
| widgetId     | String | Unique identifier for the in-page widget it belongs. |
| mood    | Integer | Mood given by the user (0-5). |
| rating   | Float32 | Rating from 0.0 to 1.0. |
| nps   | Integer | The Net Promoter Score given by the user (0-10).|
| comment   | String | Commentary left by the user in the feedback form.|
| userAgent  | String | Information about the browser user agent.|
| url  | String | Origin URL where the in-page widget result was registered.|
| geo  | Map | Geographic location of user, consisting of <code>city</code>, <code>country</code> and <code>region</code>.|

# Usabilla for Email

Usabilla for Email is a real-time customer feedback solution for integration with your email system. This section covers the resources that can be requested regarding the Usabilla for Email product.

##  Get Email Widgets

The email feedback widget is a widget usually placed in the footer of an email. Your recipients can click on the feedback widget after reading your email in order to give feedback.

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all email widgets for this account.
usabilla.email.widgets.get()
```

```python
import usabilla as ub

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Get all email widgets for this account
    widgets = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_EMAIL, api.RESOURCE_BUTTON)
    first_widget = widgets['items'][0]
    print first_widget['name']
```

```go
// Under construction
```

```php
// Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "date": "2015-03-11T14:35:04.536Z",
  "groups": [
    {
      "active": true,
      "date": "2015-03-11T14:35:04.539Z",
      "id": "af8c54de7067",
      "name": "Joshua's email footer"
    }
  ],
  "id": "e4b5a0661176",
  "introText": "We'd love your input",
  "locale": "en_US",
  "name": "Mail campaign"
}
```

### HTTPS Request

`https://data.usabilla.com/live/email/widget`

This request returns a list of your email widgets

###Request example

`https://data.usabilla.com/live/email/widget?limit=1`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| limit        | Integer  | Optional URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier of the widget. 12 characters long.            |
| date        | Date  | The creation date of the email widget, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)        |
| name        | String  | The name of the widget.           |
| introText       | String  | The name of the widget given when created. Can be updated in the widget setup page.         |
| locale       | String  | Regional language of the widget and form text items.    |
| groups       | Array  | List of groups this campaign item is member of.   |

##  Get Email Feedback

After the reader leaves feedback on a widget it becomes a feedback item that you can view in your Usabilla dashboard.

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all email widgets for this account.
usabilla.email.widgets
  .get()
  .then(emailWidgets => {
    // Get the feedback for a email widget with id.
    let emailQuery = {
      id: emailWidgets[0].id
    };

    // Get the feedback of the first email widget
    usabilla.email.widgets.feedback
      .get(emailQuery)
      .then(feedback => {
        console.log('# email feedback', feedback.length);
      })
      .catch(reason => {
        console.error(reason);
      });
  })
  .catch(reason => {
    // If the usabilla call fails, we want to see the error message
    console.error(reason);
  });
```

```python
import usabilla as ub

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Get all email widgets for this account
    widgets = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_EMAIL, api.RESOURCE_BUTTON)
    first_widget = widgets['items'][0]
    print first_widget['name']

    # Get the feedback of the first email widget
    feedback = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_EMAIL, api.RESOURCE_FEEDBACK, first_widget['id'], iterate=True)
    print len([item for item in feedback])
```

```go
// Under construction
```

```php
// Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "id": "5499612ec4698839368b4573",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36",
  "comment": "Thx for your email.",
  "location": "Amsterdam, Netherlands",
  "date": "2014-01-15T19:48:06.003Z",
  "customData": {
    "form_name": "form1"
  },
  "email": "dev@usabilla",
    "labels": [
    "label 1",
    "label 2"
  ],
  "nps": 10,
  "rating": 5,
  "buttonId": "8d73568ac2be",
  "tags": [
    "interesting",
    "unattractive"
  ],
  "urlParams": {
    "utm_campaign": "1234576_20160101_my_campaign",
    "utm_medium": "email",
    "utm_source": "email_feedback_form"
  }
}
```

### HTTPS Request

`https://data.usabilla.com/live/email/widget/<id>/feedback`

This request returns a list of your email widget feedback.

###Request example

`https://data.usabilla.com/live/email/widget/8d73568ac2be/feedback?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id       | 	String  | The widget id. If you wish to get all available feedback items use * (asterisk) as value.         |
| limit        | Integer  | Optional URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier of the feedback item.          |
| userAgent       | String  | Information about the browser user agent.     |
| comment        | String  | Commentary left by the user in the feedback form.          |
| location      | String  | String containing geographical information about the location of the user based on the user's ip address.         |
| date      | Date  | The creation date of the email widget, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)    |
| customData      | Map  | Custom variables that have been assigned in the widget setup page.  |
| email      | String  | Optional field that the user fills in in case he wishes to be contacted.  |
| labels      | Map | An array of labels that have been assigned to the feedback item.  |
| nps     | 	Integer | The Net Promoter Score given by the user (0-10). |
| rating     | 	Integer | Rating given by the user (0-5). |
| buttonId     | 	String | The source widget of the feedback item. |
| tags     | 	Array | An array of tags assigned to the feedback item.|
| urlParams    | 	Map | The URL parameters passed via the feedback URL.|

# Usabilla for Apps

Usabilla for Apps is a real-time user feedback system for analysis of customer experience on your mobile applications. This section covers the resources that can be requested regarding the Usabilla for Apps product.

##  Get App Forms

```javascript
**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all apps forms for this account.
usabilla.apps.forms.get().then(appsForms => {
    const appsForm = appsForms[0];

    if (!appsForm) {
      return;
    }
}
```

```python
import usabilla as ub

from datetime import datetime, timedelta

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Set the limit of buttons to retrieve to 1
    if False:
        api.set_query_parameters({'limit': 1})

    # Set a limit for last 7 days
    # NB: You might need to convert the since_unix calculation from scientific notation or set the since parameter manually
    if False:
        epoch = datetime(1970, 1, 1)
        since = timedelta(days=7)
        since_unix = (datetime.utcnow() - since - epoch).total_seconds() * 1000
        api.set_query_parameters({'since': since_unix})

    # Get all apps forms for this account
    forms = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_APPS, api.RESOURCE_APP)
    first_form = forms['items'][0]
    print first_form['name']
```

```go
// Under construction
```

```php
// Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "date": "2014-08-22T13:34:07.043Z",
  "id": "53f746cfc469889e628b4568",
  "name": "New untitled App Form",
  "status": "active"
}
```

Each App has a feedback button placed somewhere in the mobile application. App users can click on the feedback button at any time in order to give feedback. The feedback form (App Form) is initialized in the mobile application by calling the Usabilla SDK with the Form ID. 

### HTTPS Request

`https://data.usabilla.com/live/apps`

This request returns a list of your App Forms that are created in your Usabilla account. 

###Request example

`https://data.usabilla.com/live/apps?limit=1`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| limit        | Integer  | Optional URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier of the Form. 24 characters long.                |
| date        | Date  | The creation date of the Form, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)        |
| name       | String | The name of the Form.  |
| status      | String | The status the Form.  |

##  Get Form Feedback

```javascript
/**
 * argv[0] Access Key
 * argv[1] Secret Key
 */
const args = require('yargs').argv._;
const Usabilla = require('../dist/api-js-node');

const usabilla = new Usabilla(args[0], args[1]);

// Get all apps forms for this account.
usabilla.apps.forms
  .get()
  .then(appsForms => {
    const appsForm = appsForms[0];

    if (!appsForm) {
      return;
    }

    // Get the feedback for a apps form with id.
    let appsQuery = {
      id: appsForm.id
    };

    // Get the feedback of the second app form
    usabilla.apps.forms.feedback
      .get(appsQuery)
      .then(feedback => {
        console.log('# apps feedback', feedback.length);
      })
      .catch(reason => {
        console.error(reason);
      });
  })
  .catch(reason => {
    // If the usabilla call fails, we want to see the error message
    console.error(reason);
  });
```

```python
import usabilla as ub

from datetime import datetime, timedelta

if __name__ == '__main__':
    # Create an API client with access key and secret key
    api = ub.APIClient('YOUR-ACCESS-KEY', 'YOUR-SECRET-KEY')

    # Set the limit of buttons to retrieve to 1
    if False:
        api.set_query_parameters({'limit': 1})

    # Set a limit for last 7 days
    # NB: You might need to convert the since_unix calculation from scientific notation or set the since parameter manually
    if False:
        epoch = datetime(1970, 1, 1)
        since = timedelta(days=7)
        since_unix = (datetime.utcnow() - since - epoch).total_seconds() * 1000
        api.set_query_parameters({'since': since_unix})

    # Get all apps forms for this account
    forms = api.get_resource(api.SCOPE_LIVE, api.PRODUCT_APPS, api.RESOURCE_APP)
    first_form = forms['items'][0]
    print first_form['name']

    # Get the feedback of the first app form
    feedback = api.get_resource(
        api.SCOPE_LIVE,
        api.PRODUCT_APPS,
        api.RESOURCE_FEEDBACK,
        first_form['id'],
        iterate=True)
    print len([item for item in feedback])
```

```go
// Under construction
```

```php
// Under construction
```

> The data will be received in JSON format like this:

```json
HTTP/1.1 200 OK

{
  "appId": "53f746cfc469889e628b4568",
  "appName": "Usabilla Test Frame",
  "appVersion": "48",
  "batteryLevel": -1,
  "connection": "WiFi",
  "customData": {
    "custom_variable": "one",
    "shaken": 1,
    "user_id": "786690"
  },
  "data": {
    "name": "pablo",
    "nps": 8,
    "rating": 4,
    "speed": 5,
    "subject": "Compliment"
  },
  "date": "2015-04-20T08:12:13.382Z",
  "deviceName": "iPhone Simulator",
  "freeMemory": 224776,
  "freeStorage": 131880204,
  "geoLocation": {
    "country": "NL",
    "region": "07",
    "city": "Amsterdam",
    "lat": 52.35,
    "lon": 4.9167
  },
  "id": "5534b4dcea5b09b33557bb20",
  "ipAddress": "80.101.117.33",
  "language": "en",
  "location": "Amsterdam, Netherlands",
  "orientation": "Portrait",
  "osName": "ios",
  "osVersion": "8.3",
  "rooted": false,
  "screenshot": "https://usabilla-feedback-app.s3.amazonaws.com/staging/media/55/34/5534b4dcea5b09b33557bb20/screenshot",
  "screensize": "320x568",
  "timestamp": "1429517532",
  "totalMemory": 14032256,
  "totalStorage": 243924992
}
```
After the user sends feedback using the feedback button it becomes a feedback item that you can view in your Usabilla dashboard.

### HTTPS Request

`https://data.usabilla.com/live/apps/<id>/feedback`

This request returns a list of your feedback items that are connected to a specific App Form. The App Form is loaded in from the Usabilla SDK once a user clicks on the feedback button in the application.

###Request example

`https://data.usabilla.com/live/apps/8d73568ac2be8d73568ac2be/feedback?limit=1&since=1419340238645`

### Parameter

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id       | String | The Form ID. For feedback items of all Forms use <code>*</code> (asterisk) as value.                                |
| limit        | Integer  | Optional URL query parameter that limits the number of retrieved results between 1 and 100. Defaults to 100 when omitted.                                |
| since      | Timestamp  | Optional UTC Timestamp (in milliseconds) URL query parameter used to retrieve items of which creation date is after the supplied value. |

### Success 200

| Field | Type | Description                                                                         |
| --------- | ------- | ----------------------------------------------------------------------------------- |
| id        | String  | Unique identifier of the feedback item.               |
| date        | Date  | The creation date of the Form, in UTC. The format being used is <code>%Y-%m-%dT%H:%M:%S.%fZ</code> as defined in [ISO 8601.](https://www.w3.org/TR/NOTE-datetime)        |
| timestamp    | String | Date and time (Unix timestamp) when feedback was sent by the App.  |
| deviceName    | String | 	Device name on which the app is running.  |
| data   | Map | 	Map containing the Feedback data ("mood" and "nps" scores).  |
| customData   | Map | 	Map containing the optionally configured custom values.  |
| appId   | String | 	ID of the feedback form the feedback item belongs to. |
| appName   | String | 	Name of the feedback form |
| appVersion   | String | 	Version of the app. |
| osName  | String | 	Operating System name on which the app is running. |
| osVersion  | String | 	Operating System version on which the app is running. |
| location  | String | 	Readable location (City, Country). |
| geoLocation  | Map | 	Geographic location details ("country", "region", "city", "lat", "lon"). |
| freeMemory | Integer | 	Bytes of RAM left unused. |
| totalMemory | Integer | 	Bytes of RAM in total. |
| freeStorage | Integer | 	Bytes of storage memory left unused. |
| totalStorage | Integer | 	Bytes of storage memory in total. |
| screenshot | String | 	Screenshot URL. |
| screensize | String | 	Screen property size. |
| connection | String | 	Current connections. |
| ipAddress | String | 	Current IP address. |
| language | String | 	Language used. |
| orientation | String | 	The orientation of the screen at time of feedback ['Portrait','Landscape']. |
| batteryLevel| Float32 | 	Battery level from 0.0 to 1.0 (-1 means unknown). |