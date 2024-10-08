## OAuth 2.0

All endpoints for retrieving a FasCat user's data requires successful authorization via the [OAuth 2.0](https://oauth.net/2/) protocol. [IETF RFC-6749](https://tools.ietf.org/html/rfc6749) documents the full standard.

### OAuth Configuration

#### Client ID / Secret

Available on request.

#### Authorization URL

This URL prompts a user to authorize your app to access the user's data. After the user grants authorization, FasCat will respond with an authorization code. On mobile platforms where the CoachCat app is installed, this URL will launch the app and prompt the user to authorize your app.

```
https://api.fascatapi.com/auth/v1/oauth/authorize
```

### Token URL

This URL provides your app with the authorization code from the authorization URL after the FasCat user authorizes your app to access their data. This endpoint provides an access token and refresh token in exchange for the authorization code.

```
https://api.fascatapi.com/auth/v1/oauth/token
```

### Redirect URIs

FasCat will redirect the user to the [Redirect URI](https://www.oauth.com/oauth2-servers/redirect-uris/) after they authorize your app to access the request scopes. A valid URI will take the form of `https://mydomain.com/example/redirect` or `myapp://example/redirect`.

### State

The [state parameter](https://auth0.com/docs/protocols/state-parameters) is a generated random string that you provide to FasCat as a query parameter appended to the [Authorization URL](#authorization-url). Upon successful authorization by the user, FasCat will respond with this same state appended to the [Redirect URI](#redirect-uris).

### Scope

The [scope](https://oauth.net/2/scope/) parameter is how your application declares which data and what level of access is being requested. There are currently three scopes available for FasCat API access:

- `profile:read` - Read user profile data.
- `workout:read` - Read a user's planned workouts.
- `activity:write` - Upload completed activities.

When the OAuth flow prompts a FasCat user to authorize access to your app, the user will see the list of scopes your app requested. The user must authorize access to them before FasCat grants your app a token.

## Authentication

OAuth2 Uses the following authentication workflow. The goal is to obtain an `access_token` for a user that can be used for accessing FasCat APIs on the user's behalf.

### OAuth Workflow

<b>Step 1.</b> Send the user to FasCat [Authorization URL](#authorization-url) in order to login and grant access to your app using:

```
https://api.fascatapi.com/auth/v1/oauth/authorize?response_type=code&client_id=<clientId>&redirect_uri=<redirectUri>&scope=activity:write%20workout:read%20profile:read&state=<appState>
```

Parameters:

- `response_type=code` - The response type is always `code` for the authorization code grant type.
- `client_id` - Your app's client ID.
- `redirect_uri` - The final URI that FasCat will redirect to after the user authorizes your app. It must match one of the [Redirect URIs](#redirect-uris) configured for your app.
- `scope` - The scopes your app is requesting access to. Can be left off if you want to use the default scopes for your app.
- `state` - A random string generated by your app that FasCat will return to you as part of the final redirect URI parameters.

<b>Step 2.</b> After successful authorization the user will be redirected back to your `redirect_uri` with `code` and `state` query parameters:

```
<redirect_uri>?code=<code>&state=<state>
```

<b>Step 3.</b> Exchange authorization code for access token. Send an `HTTP POST` to FasCat with the code and your app's OAuth2 credentials and receive the `access_token` and `refresh_token`:

```http
POST https://api.fascatapi.com/auth/v1/oauth/token
Content-Type: application/x-www-form-urlencoded
Accept: application/json

grant_type=authorization_code&code=<authorizationCode>&redirect_uri=<redirectUri>&client_id=<clientId>&client_secret=<clientSecret>
```

Response:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9",
  "refresh_token": "SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "profile:read workout:read activity:write"
}
```

<b>Step 4.</b> When the access_token expires after 1 hours you can use the `refresh_token` to get a new `access_token` and a new `refresh_token`:

```http
POST https://api.fascatapi.com/auth/v1/oauth/token
Content-Type: application/x-www-form-urlencoded
Accept: application/json

grant_type=refresh_token&refresh_token=<refreshToken>&client_id=<clientId>&client_secret=<clientSecret>
```

Response:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9",
  "refresh_token": "SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "profile:read workout:read activity:write"
}
```

<b>Step 5.</b> You are now ready to send requests to our API by putting the new `access_token` within the following HTTP Header:

```
Authorization: Bearer <access_token>
```

## Deauthorization

Application access can be revoked by sending a DELETE request to the following endpoint:

```http
DELETE https://api.fascatapi.com/auth/v1/oauth/permissions
Authorization: Bearer <access_token>
```
