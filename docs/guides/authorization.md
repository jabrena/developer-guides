# Authorization

This guide will explain the authorization processes required to access Amadeus Self-Service APIs.

## Overview

The basic authorization process is:

1. [Create an account](https://developers.amadeus.com/register) on the Amadeus for Developers portal
2. Create an application
3. Get an `API Key` and `API Secret`
4. Call the authorization server to generate an access token
5. Use the access token to authenticate requests to Amadeus Self-Service APIs

 Remember that your `API Key` and  `API Secret` should be kept private. Read more about best practices for [secure API key storage](https://developers.amadeus.com/blog/best-practices-api-key-storage).


## Introduction to OAuth

Amadeus for Developers uses `OAuth` to authenticate access requests. `OAuth` generates an `access token` which grants the client permission to access a protected resource. 

The method to acquire a token is called **grant**. There are different types of OAuth grants. Amadeus for Developers uses the `Client Credentials Grant`.

## Generating an access token
Once you have created an app and received your `API Key` and  `API Secret`, you can generate an access token by sending a `POST` request to the authorization server.

### URL
[https://test.api.amadeus.com/v1/security/oauth2/token/](https://test.api.amadeus.com/v1/security/oauth2/token/)

### Request 
The following paramenters are required: 

| Parameter      | Description |
| ----------- | ----------- |
| `grant_type`      | The value `client_credentials`        |
| `client_id`       | The `API Key` for the application        |
| `client_secret`   | The `API Secret` for the application  |  


The authorization request is encoded as `x-www-form-urlencoded`, where the keys and values are encoded in key-value tuples separated by '&', with a '=' between
the key and the value.

 

Specify the type of the request using the `content-type` HTTP header set to `application/x-www-form-urlencoded`:

```bash
curl "https://test.api.amadeus.com/v1/security/oauth2/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials&client_id={client_id}&client_secret={client_secret}"
```
Note that the `-X POST` parameter is not specified in the `cURL`  command, as we are sending a body.

### Response

The authorization server will respond with a JSON object containing the following properties:

| Parameter      | Description |
| ----------- | ----------- |
| `type`      | set to `amadeusOAuth2Token` string        |
| `username`       | Your username \(email address\)        |
| `application_name`   | The name of your application.  |
| `client_id`      |  The `API Key` for the application         |
| `token_type`       | The type of token issued by the authentication server. The value will be `Bearer`.        |
| `access_token`   | The token to authenticate your requests.  |
| `expires_in`   | The number of seconds until the token expires.  |
| `state`   | With the value `approved`  |

Example response:

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

## Using the token

Once the token has been retrieved, you can authenticate your requests to Amadeus Self-Service APIs.

Include the value `Bearer {access_token}` in the `authorization` header of your request, where `acess_token` is the token you have just retrieved.

Example call to the `Flight Check-in Links` API to retrieve thecheck-in URL for Iberia \(`IB`\):

```bash
curl "https://test.api.amadeus.com/v2/reference-data/urls/checkin-links?airline=IB" \
     -H "Authorization: Bearer CpjU0sEenniHCgPDrndzOSWFk5mN"
```

## Using our SDKs to manage tokens
To help simplify the authentication process, we offer [Amadeus for Developers
SDKs](https://github.com/amadeus4dev). The `SDKs` automatically fetch and store the `access_token` and set the headers in all API calls.

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

