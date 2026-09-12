## Common flaws in CSRF token validation
### Validation depends on request method
Some applications will correctly validate the token, but only when the request uses the `POST` method, meaning attackers can just switch to a `GET` request to bypass the validation entirely.
### Validation depends on token being present
Some applications will correctly validate the token when it is present, but just skip it if it is omitted. In this situation, an attacker can remove the entire token parameter.
### Token is not tied to user session
Some applications do not validate the token belongs to the same session as the user making the request. Instead, it maintains a global pool of issued tokens and accepts any token so along as it exists within this pool. The attacker can login with their own account, obtain a valid token and feed it to the victim.
### Naive Double-Submit Cookie Pattern
The next flaws involve tokens being tied to cookies, so it's important we understand what this actually means. 

One way to validate whether a CSRF token is valid is to issue a cookie holding the token to the user upon login. When that user makes a sensitive `POST` request, the token is copied to the request and is sent alongside the original cookie. The server then verifies that the two values match and responds accordingly. This relies on the assumption that cookies are stored client-side, so an attacker should have no way to read the cookie value and therefore cannot find the correct token to include in their payload.

However, while cookies cannot be read by an attacker, **it is possible to for an attacker to inject  cookies**, which introduces vulnerabilities depending on the scenario:
#### Token is tied to a non-session cookie
However, if the application doesn't tie the token to the same cookie that tracks the session, usually because it employs two different frameworks for session tracking and CSRF tokens, then there exists an avenue for a CSRF attack.

In practice, this looks like a session cookie and a CSRF cookie, with a new cookie parameter to identify the CSRF cookie, something like `csrfKey`. 

If the web application has some behaviour that would allow the attacker to set a cookie on the victim's browser, the attacker doesn't need to read the victim's cookie. They can simply assign whatever token they please to the victim then use the same one in their CSRF page.

>**Note**
>The cookie-setting behaviour doesn't even need to exist within the same web application as the CSRF vulnerability. Any other application within the same overall DNS domain can potentially be leveraged to set cookies in the target application, if the cookie has suitable scope. For example, a cookie-setting function on `staging.demo.website.com` could be leverage to place a cookie submitted to `secure.website.com`.
#### Token is tied to a non-session cookie but server maintains record of tokens
If the server maintains a pool of valid tokens, then, as before, we can login to generate our own token, place a cookie on the victim's browser with our token, then include that same token in the CSRF payload.

## The Gold Standard
### Signed Double-Submit Cookie Pattern
The solution to the above, and the standard protection against CSRF, is **Signed Double-Submit Cookie Pattern**, which introduces **hashing**.

When the user logs in, the server uses a secret key to hash a random CSRF token. The token and the resulting hash are then passed to the client via a cookie. The client extracts only the un-hashed token from the cookie and passes that in any requests. When the request gets back to the server, it hashes the token in the request and checks that it matches the hash in the cookie. Now, without knowing the hash, CSRF attacks are impossible.