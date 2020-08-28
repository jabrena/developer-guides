# Authorization

This guide will explain the authorization processes required to access Amadeus Self-Service APIs.

## Overview

The basic authorization process is:

1. [Create an account](https://developers.amadeus.com/register) on the Amadeus for Developers portal
2. Create an application
3. Get an `API Key` and `API Secret`
4. Call the authorization server to generate an access token
5. Use the access token to authenticate requests to Amadeus Self-Service APIs

!!!information
    Remember that your `API Key` and  `API Secret` should be kept private. Read more about best practices for [secure API key storage](https://developers.amadeus.com/blog/best-practices-api-key-storage).

The current guide covers steps 4 and 5 of the authorization process.

## What is OAuth?

Amadeus for Developers uses `OAuth` to authenticate access requests. `OAuth` generates an `access token` which grants the client permission to access a protected resource. 

The method to acquire a token is called **grant**. There are different types of `OAuth grants`. Amadeus for Developers uses the `Client Credentials Grant`.

## Requesting an acces token

Once you have created an app and received your `API Key` and  `API Secret`, you can generate an access token by sending a `POST` request to the authorization server:

[https://test.api.amadeus.com/v1/security/oauth2/token/](https://test.api.amadeus.com/v1/security/oauth2/token/)

The body of the request is encoded as `x-www-form-urlencoded`, where the keys and values are encoded in key-value tuples separated by '&', with a '=' between
the key and the value:

| Key | Value |
| ----------- | ----------- |
| `grant_type`      | The value `client_credentials`        |
| `client_id`       | The `API Key` for the application     |
| `client_secret`   | The `API Secret` for the application  |

We also need to specify the type of the request using the `content-type` HTTP header with the value `application/x-www-form-urlencoded`.

The following example uses `cURL` to request a new token, sending the body and the headers as described:

```bash
curl "https://test.api.amadeus.com/v1/security/oauth2/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials&client_id={client_id}&client_secret={client_secret}"
```
Note that the `-X POST` parameter is not needed in the `cURL` command. As we are sending a body, `cURL` sends the request as `POST` automatically.

### Understanding the response

The authorization server responds with a JSON object like this one:

```javascript
{
    "type": "amadeusOAuth2Token",
    "username": "foo@bar.com",
    "application_name": "BetaTest_foobar",
    "client_id": "3sY9VNvXIjyJYd5mmOtOzJLuL1BzJBBp",
    "token_type": "Bearer",
    "access_token": "CpjU0sEenniHCgPDrndzOSWFk5mN",
    "expires_in": 1799,
    "state": "approved",
    "scope": ""
}
```
Let's take a look to the parameters:

| Parameter      | Description |
| ----------- | ----------- |
| `type`      | The type of the resource. This value will be `amadeusOAuth2Token`. |
| `username`       | Your username \(email address\)        |
| `application_name`   | The name of your application.  |
| `client_id`      |  The `API Key` for the application  |
| `token_type`       | The type of token issued by the authentication server. The value will be `Bearer`.        |
| `access_token`   | The token to authenticate your requests.  |
| `expires_in`   | The number of seconds until the token expires.  |
| `state`   | The status of your request. Values can be `approved` or `expired`.  |

## Using the token

Once the token has been retrieved, you can authenticate your requests to Amadeus Self-Service APIs.

Each API call must contain the `authorization` HTTP header with the value `Bearer {access_token}`, where `acess_token` is the token you have just retrieved.

The following example calls to the `Flight Check-in Links` API to retrieve the check-in URL for Iberia \(`IB`\):

```bash
curl "https://test.api.amadeus.com/v2/reference-data/urls/checkin-links?airline=IB" \
     -H "Authorization: Bearer CpjU0sEenniHCgPDrndzOSWFk5mN"
```

## Managing tokens from your source code

The process to retrieve a token using your favourite programming language is
similar to the `cURL` examples described above: send a `POST` request and parse
the `JSON` response. The only thing you need to take into consideration is the
expiration time of the token. There are different strategies to keep your token
updated: check the remaining time before each API call or capture the
`unauthorized` error when the token expires. In both cases you will need to
request a new token.

In order to simplify the authentication process implementation, you can use any
of the [SDKs](https://github.com/amadeus4dev) available on GitHub. The `SDKs`
automatically fetch and store the `access_token` and set the headers in all API
calls.

Example of how to initialize the client and authenticate with the `Node` SDK:

```javascript
var Amadeus = require('amadeus');

var amadeus = new Amadeus({
  clientId: '[API Key]',
  clientSecret: '[API Secret]'
});
```

You can then call the API. Example call to the `Flight Check-in Links` API to retrieve thecheck-in URL for Iberia \(`IB`\):


```javascript
amadeus.referenceData.urls.checkinLinks.get({ airline: 'IB' });
```

