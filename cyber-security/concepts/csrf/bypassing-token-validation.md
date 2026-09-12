## Generic flaws in CSRF token validation
### Validation depends on request method
Some applications will correctly validate the token, but only when the request uses the `POST` method, meaning attackers can just switch to a `GET` request to bypass the validation entirely.
### Validation depends on token being present
Some applications will correctly validate the token when it is present, but just skip it if it is omitted. In this situation, an attacker can remove the entire token parameter.
## Flaws in Stateful CSRF token validation
*In this case, stateful means that the database holds onto something to do with CSRF token validation*.
### Token is not tied to user session
Some applications do not validate the token belongs to the same session as the user making the request. Instead, it maintains a global pool of issued tokens and accepts any token so along as it exists within this pool. The attacker can login with their own account, obtain a valid token and feed it to the victim.

>**Note**
>*The following flaws occur when cookies have been implemented to improve CSRF resilience. The server maintains a list of ID/token pairs. When a user logs in, they are sent a cookie containing an ID alongside it's corresponding token. The token is copied to client requests and is sent alongside the original cookie containing the ID to the server. The server checks whether the cookie ID and request token correspond, and replies accordingly*.
>*Also note that, ideally, there isn't a dedicated CSRF ID, and instead each session has a paired token, we'll see why subsequently.*
### Token is tied to a non-session cookie
If the application doesn't tie the token to the same cookie that tracks the session, usually because it employs two different frameworks for session tracking and CSRF tokens, then there exists an avenue for a CSRF attack. In practice, this looks like a session cookie and a CSRF cookie, with a new cookie parameter to identify the CSRF cookie, something like `csrfKey`. 

If the web application has some behaviour that would allow the attacker to set a cookie on the victim's browser, the attacker doesn't need to read the victim's cookie.  The attacker logs in, generating a `csrfKey` and corresponding `token`. The CSRF page then injects the `csrfKey` cookie parameter belonging to the attacker onto the victims browser, and then does a standard CSRF attack with the corresponding token.

*The cookie-setting behaviour doesn't even need to exist within the same web application as the CSRF vulnerability. Any other application within the same overall DNS domain can potentially be leveraged to set cookies in the target application, if the cookie has suitable scope. For example, a cookie-setting function on `staging.demo.website.com` could be leverage to place a cookie submitted to `secure.website.com`.*
## Flaws in Stateless CSRF token validation
*Stateless meaning nothing is stored server-side.*
### Naive Double-Submit Cookie Pattern
If we don't want to store anything server-side, then we can use **Double-Submit Cookie Pattern**. Again, the server submits a cookie containing the token which is passed to any requests, but instead of maintaining a key-value pair list on the database, the server just checks whether the cookie token and request token match.

Once more, if we can inject cookies, we can just inject an arbitrary token and include a matching one in our CSRF payload.
## The Gold Standard
### Signed Double-Submit Cookie Pattern
The solution to the above, and the standard protection against CSRF, is **Signed Double-Submit Cookie Pattern**, which introduces **hashing**.

When the user logs in, the server uses a secret key to hash a random CSRF token. The token and the resulting hash are then passed to the client via a cookie. The client extracts only the un-hashed token from the cookie and passes that in any requests. When the request gets back to the server, it hashes the token in the request and checks that it matches the hash in the cookie. Now, without knowing the hash, CSRF attacks are impossible.