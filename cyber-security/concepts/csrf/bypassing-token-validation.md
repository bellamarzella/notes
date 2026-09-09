## Common flaws in CSRF token validation
### Validation depends on request method
Some applications will correctly validate the token, but only when the request uses the `POST` method, meaning attackers can just switch to a `GET` request to bypass the validation entirely.
### Validation depends on token being present
Some applications will correctly validate the token when it is present, but just skip it if it is omitted. In this situation, an attacker can remove the entire token parameter.
### Token is not tied to user session
Some applications do not validate the token belongs to the same session as the user making the request. Instead, it maintains a global pool of issued tokens and accepts any token so along as it exists within this pool. The attacker can login with their own account, obtain a valid token and feed it to the victim.
### Tokens & Cookies
The next flaws involve tokens being tied to cookies, so it's important we understand what this actually means. 
One way of validating a CSRF token is to issue a cookie holding it to the user when the login, which is copied to any `POST` requests they make. The server then verifies that the cookie token and the request token match. This relies on the fact that cookies are stored client-side on the browser, so an attacker should have no way to get a victim's cookie and therefore their token.
#### Token is tied to a non-session cookie
If the application doesn't tie the token to the same cookie that tracks the session, usually because it employs two different frameworks for session tracking and CSRF tokens, then there exists an avenue for a CSRF attack.
If the web application has some behaviour that would allow the attacker to set a cookie on the victim's browser, the attacker can assign whatever token they please to the victim, then just use the same one in their CSRF page.

>**Note**
>The cookie-setting behaviour doesn't even need to exist within the same web application as the CSRF vulnerability. Any other application within the same overall DNS domain can potentially be leveraged to set cookies in the target application, if the cookie has suitable scope. For example, a cookie-setting function on `staging.demo.website.com` could be leverage to place a cookie submitted to `secure.website.com`.

### Token is duplicated in a cookie
In a further variation of the above, some applications do not maintain server-side records of tokens that have been issues, but instead duplicate each token within a cookie and a request parameter.