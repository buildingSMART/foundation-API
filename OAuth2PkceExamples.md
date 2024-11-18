# Open CDE APIs OAuth2 with PKCE Example

This example describes an example OAuth2 workflow using the _Authorization Code Grant_ flow as per [section 4.1 of the OAuth2 specification](https://tools.ietf.org/html/rfc6749#section-4.1). URIs of required endpoints are assumed to have been obtained from the authentication resource as described in [section 2.2 of the Foundation API specification](README.md#221-obtaining-authentication-information).

For this example, it is assumed that the client has been registered with the server and have a valid `client_id`.

PKCE (Proof Key for Code Exchange) is usually required for public clients where the client is susceptible to the authorization code interception attacks.

## Client Creates a Code Verifier
The client needs to create a `code_verifier` for each OAuth 2.0 authentication request. The `code_verifier` should be a secure random string per [RFC7636 Section 4.1](https://www.rfc-editor.org/rfc/rfc7636#section-4.1). 

## Client Creates a Code Challenge
Using the Code Verifier, generate a `code_challenge` using the method defined in `code_challenge_method` per [RFC7636 Section 4.2](https://www.rfc-editor.org/rfc/rfc7636#section-4.2).

Important note: Per RFC, `S256` _MUST_ be used if the client can. Not all servers support `plain`.

## Authorization Request

To initiate the workflow, the client sends the user to the **"oauth2\_auth_url"** with the following parameters added:

|parameter| value                                                                                                                                 |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------|
|response_type| `code` as string literal                                                                                                              | 
|client_id| your client id                                                                                                                        |
|state| unique user defined value                                                                                                             |
|redirect_url| The `redirect_url` registered for your client. This parameter is optional for OAUTH servers that don't support multiple redirect URLs |
|code_challenge| `code_challenge`                                                                                                                      |
|code_challenge_method| `code_challenge_method`, i.e. `S256` or `plain`                        |

Example URL:

    GET https://example.com/foundation/oauth2/auth?response_type=code&client_id=<your_client_id>&state=<user_defined_string>&redirect_url=https://YourWebsite.com&code_challenge=<your_code_challenge>&code_challenge_method=S256

Example redirected URL:

    https://YourWebsite.com/retrieveCode?code=<server_generated_code>&state=<user_defined_string>

The Open CDE API server will ask the user to confirm that the client may access resources on his behalf. On authorization, the server redirects to an url that has been defined by the client author in advance. The generated `code` parameter will be appended as query parameter. Additionally, the `state` parameter is included in the redirection, it may be used to match server responses to client requests issued by your application.

If the user denies client access, there will be an `error` query parameter in the redirection indicating an error reason as described in [section 4.1.2.1 of the OAuth2 specification](https://tools.ietf.org/html/rfc6749#section-4.1.2.1).

## Token Request

With the obtained _authorization code_, the client is able to request an access token from the server. The  **"oauth2\_token_url"** from the authentication resource is used to send token requests to, for example:

    POST https://example.com/foundation/oauth2/token

The POST request should be made with parameters using the "application/x-www-form-urlencoded" in the request entity body per [RFC6749 Section 4.1.3](https://www.rfc-editor.org/rfc/rfc6749#section-4.1.3).

**Example Request**

    POST https://example.com/foundation/oauth2/token

    Headers
        Content-Type: application/x-www-form-urlencoded
    
    Body
        client_id=<your_client_id>
        grant_type=authorization_code
        code=<your_access_code>
        code_verifier=<your_code_verifier>
        redirect_uri=https://YourWebsite.com

The access token will be returned as JSON in the response body and is an arbitrary string. There is no maximum length, per [oauth2 documentation](https://tools.ietf.org/html/rfc6749#section-1.4).

**Response parameters**

|parameter|type|description|
|---------|----|-----------|
|access_token|string|The issued OAuth2 token|
|token_type|string|Always `bearer`|
|expires_in|integer|The lifetime of the access token in seconds|
|refresh_token|string|The issued OAuth2 refresh token, one-time-usable only|

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "access_token": "Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh",
        "token_type": "bearer",
        "expires_in": "3600",
        "refresh_token": "MTRiMjkzZTYtOTgwNC0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh"
    }

## Refresh Token Request

Refer to [OAuth2 Example](OAuth2Examples.md)

## Requesting Resources

Refer to [OAuth2 Example](OAuth2Examples.md)
