# Introduction

  Since early 2019 Frontier Developments has required oAuth2 to access their CAPI (Companion API) service.
  Before a developer can make use of this API they must first apply for access.

1. Navigate a web browser to <https://user.frontierstore.net/> and **apply** for access (I can't more fully document this because I long since went through the process and only have one applicable account).
2. Once Frontier have **approved** your access (it will _not_ be instant), navigate back to <https://user.frontierstore.net/> to review the available information.  You should default to the 'User Information' view which shows:

	1. Frontier ID - your unique ID for this system

	2. Name - The real name specified in your account

	3. Email - The email address associated with your account

	4. Platform - Whether this is a Frontier account, XBox,
	Playstation or Steam.  The 'Authorized Applications' lists all
	the Clients (applications) you've authorised to request data on
	your behalf.  This might include sites like Inara.Cz.

3. The 'Developer Zone' is where you'll see your authorised Clients
once access is granted.  You will probably have 'AUTH' and 'CAPI' scopes.
Clicking 'View' on this will reveal your Client ID and Shared Key.
You can also 'Regenerate key' here if you ever need to.

 There is a link to <https://user.frontierstore.net/developer/docs>,
the developer documentation.  If you are wanting to use CAPI with a
standalone application, rather than a web-based application you will
need to follow the PKCE documentation.  You may find <https://www.oauth.com/oauth2-servers/pkce/> more useful.

[0] - It's called CAPI / Companion API because it was originally created
solely for the use of an authorised, third-party developed, Companion
application for iOS (an Android version was never produced).  Although
that application is long since dead (not working on many subsequent iOS
revisions), enough developers figured out the API and started making use
of it that Frontier Developments decided to continue supporting it.

## Required Reading

  You should first familiarise yourself with at least the [oAuth2
terminology](https://www.oauth.com/oauth2-servers/definitions/).  The
other documentation on that site is very useful for a full
understanding of how oAuth2 works.  RFCs for things like JSON Web Tokens
may also prove informative:

1. [The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)
1. [The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://tools.ietf.org/html/rfc6750)
1. [JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
1. [JSON Web Encryption (JWE)](https://tools.ietf.org/html/rfc7516)
1. [JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)

## PKCE

  If your Client does not run in a web browser than you'll want to
use PKCE for the authentication flow.  Example code here ends up using a
web browser for the Redirect URI anyway, but the principle is the same.

  If you have a good library available in your chosen development
environment then please feel free to use that.  If you're doing this all
"by hand", then read on.

### Authorization Request

  You'll want to craft an Authorization Request to pass to the User.
Either they'll have to copy and paste it into a browser, or perhaps
you can invoke a browser with it.

  This URL should be of the form:

	https://auth.frontierstore.net/auth?audience=frontier&scope=auth%20capi&response_type=code&client_id=YOUR_APPROVED_CLIENTID&code_challenge=CODE_CHALLENGE&code_challenge_method=S256&state=STATESTRING&redirect_uri=REDIRECT_URI

  An 'audience' of 'frontier' means a Frontier Developments account,
rather than a Steam, XBox, or Playstation account.  See the 'AUDIENCE'
part of <https://user.frontierstore.net/developer/docs> for the valid
values.

  A 'scope' of 'auth capi' means we're requesting access to the CAPI, instead
of just 'auth' which would just let us know some information associated
with the account (real name, email address).

  Obviously you need to replace the requisite parts of this:

1. YOUR_APPROVED_CLIENTID is your 'Client ID' from the Frontier
    'Developer Zone' for your application.
2. You need to generate a CODE_VERIFIER.

	1. Generate 32 bytes (octets) of random data, as securely as you can.
	2. Base64 encode this, in a URL safe version; replace '+' with
      '-', and '/' with '_', but the '=' on the end needs to remain in
      place.

3. From this generate a CODE_CHALLENGE:

	1. Create a binary, not hex, representation of a sha256 hash of
      the raw, not string form, CODE_VERIFIER.
	2. Base64 encode this in a URL safe manner.  You **must**
     strip off the trailing '='.  If not you'll get:

			{"message":"An error occured.","logref":"<hex id>"}
     when you try to use the CODE_VERIFIER in the subsequent Token Request.

	3. Make sure you have a string representation of this (not, e.g.
      python bytes).

4. Generate a STATESTRING

	1. Generate 32 bytes (octets) of random data, as securely as you
      can.
	2. Base64 encode this, in a URL safe version; replace '+' with
      '-', and '/' with '_', **and** strip the '=' from the end.
	3. Make sure you have a string representation of this (not, e.g.
      python bytes).

5. REDIRECT_URI is how your Client receives back the CODE from
    Frontier's Authorization Server.  If you do have a web server
    available then set up a receiving script there.  If operating on
    a mobile device you probably want to register a custom URL scheme
    handler and point to that, e.g. myapp://fd-auth-redirect.  Just so
    long as:

	1. The web browser on the User's device understands and can reach this
       URL.
	2. You can then get the received CODE back into your Client.

Now that you have crafted the Authorization Request URL, give it to the User.
They'll be asked to login on Frontier's server (if needs be) and then
approve your Client's requested access.  The key thing is that with
PKCE you do **NOT** want to send a query to this URL yourself.

  Once the User has logged in and approved your Client you'll
receive a code back as a GET parameter at the REDIRECT_URI you
specified.

6. In the REDIRECT_URI handler you *should* check that the received
*state* matches the STATESTRING you originally sent in order to verify
you're in sync with the Authorization Server.

### Token Request

7. Craft a Token Request, which this time you *are* going to send to
the Authorization Server yourself.  Note you need to use a POST request,
not GET.

	1. The URL is:

			https://auth.frontierstore.net/token

 	2. You need to set a header:

			Content-Type: application/x-www-form-urlencoded

	3. And the data in the body will be a string (NB: *not* a JSON
	   object!):

			redirect_uri=REDIRECT_URI&code=CODE&grant_type=authorization_code&code_verifier=CODE_VERIFIER&client_id=CLIENTID

		1. REDIRECT_URI - this needs to be the same as you
		used in the Authorization Request, and	does need to be
		URL-Encoded (%XX versions of ':' and '/' at least).  It
		won't be *used* at this stage, as you're making the
		Token Request yourself and will receive the answer back
		directly.

		2. CODE - The value you received back as a 'code=XXX'
		GET parameter in the REDIRECT_URI script.

		3. CODE_VERIFIER - The URL-Safe Base64 version of your
		VERIFIER.  This **MUST** still have the trailing '=',
		else you'll get

			{"message":"An error occured.","logref":"<hex id>"}

		4. CLIENTID - Your Client ID.

8. Send this Token Request and if it's all worked you'll get a 200 response,
with the body being a JSON object containing the tokens.  If you
did something wrong, or took too long, you'll likely get a 401
response.  
The JSON will contain a few keys and their values:
	1. access_token - the token to be used on CAPI endpoint requests

	2. refresh_token - the token that can be used to get a new
       access_token if the latter has expired, assuming this token hasn't
       also expired.

	3. token_type - "Bearer"

	4. expires_in - Seconds until the Access Token expires. 

For possible future compatibility you might want to store the token_type
to literally repeat it back when using the Access Token in CAPI
requests.

### Summary

1. Generate a VERIFIER.  You'll then generate a CHALLENGE from this,
sent in the Authorization Request.  You will later send the
VERIFIER in the subsequent Token Request to prove it was you who made
the initial request.

2. Generate the initial Authorization Request, which will ask the User
to authorize your Client's access to their account.

3. Success will cause the browser to go to the specified Redirect URI,
with a CODE, and the STATE you specified, passed as GET parameters.

4. You then send a Token Request, including your VERIFIER, Client ID,
and the CODE you just got to ask for the Access and Refresh Tokens.

## Utilising the Refresh Token

The Access Token will have an expiry time, and when this has been
reached you will receive HTTP '422 Unprocessable Entity' response from
the CAPI endpoints.  To get a new Access Token you are not required to
initiate a new Authorization Request, which would require the User to
approve your Client again.  Instead you can utilise the Refresh
Token that you got as part of that process.

**Note, however, that you will only be able to use *any* Refresh token
for an account for at most 25 days after the initial Authorization.
After that you will need the user to re-Authorize your Application.**
When they do they should only need to login on `auth.frontierstore.net`
with this succeeding at that point, rather than also asking them to
Approve your Application again.

1. Craft a POST request in order to get a new Access Token.
	1. This being POST you'll need to include the header

			Content-Type: application/x-www-form-urlencoded
	1. Then the body of the request will be data comprising of:
		1. grant_type=refresh_token
		1. client_id=CLIENTID - Your Application's "Client ID"
		1. client_secret=SHAREDKEY - Your Application's "Shared
		   Key"
		1. refresh_token=REFRESH_TOKEN
	
	NB: You only need the client_secret if using non-PKCE Authorization.
	All together that makes:

			grant_type=refresh_token&client_id=CLIENTID&client_secret=SHAREDKEY&refresh_token=REFRESH_TOKEN

1. Send this to the token endpoint on the Authorization Server,
<https://auth.frontierstore.net/token>.  If the Refresh Token hasn't
also expired (what's the lifetime?) then you'll receive a HTTP 200
response, *Content-Type: application/json*, with the body being a JSON
object containing:

	1. A new Access Token
	2. A new expires_in time delta (14400 seconds = 4 hours) for
	   that Access Token
	3. A new Refresh Token.

1. If the User has Revoked your Client's access the Authorization Server
   will respond with a '401' HTTP Status when you try to use the Refresh
   Token.

## A note on the nature of Tokens

The tokens you get back from the Authorization Server are JSON Web Tokens
(JWT).  They are not simply a key to use but also contain information
themselves.  The `/decode` and `/me` endpoints on Frontier's Authorization
Server decrypt this information and pass it back to you.

The JWTs Frontier creates for Clients' use contain an encrypted
Ciphertext.  Within that Ciphertext is some Personal Identifying
Information (PII) of the User, along with information about allowed
'downloads' (ship skins and the like), and some metadata like when the
Access Token expires.  The contents can only be decrypted with a
key that only Frontier has access to.  Both the CAPI and Authorization
Servers do check the `exp` field in an Access Token's encrypted Ciphertext
and will refuse to operate on it if that time has passed.

Frontier does not have to store the Access Token their end, and it seems
they do not, they can simply verify it cryptographically.

This has two consequences for Clients.  Firstly it is unsurprising that
if a User revokes a Client's access this does not actually revoke any
currently *unexpired* Access Token.  It will continue to work with the
CAPI servers until it expires.

Secondly a Client can always use the `/decode` and `/me` endpoints of
the Authorization Server with an *unexpired* Access Token.  All the
Authorization Server is doing is verifying the `exp` time hasn't passed
and then decrypting the content you passed it.
