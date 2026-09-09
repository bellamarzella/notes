## Common flaws in CSRF token validation
### Validation depends on request method
Some applications will correctly validate the token, but only when the request uses the `POST` method, meaning attackers can just switch to a `GET` request to bupass the validation entirely (or vice-versa).
### Validation depends on token being present
Some applications will correctly validate the token when it is present, but just skip it if it is omitted. In this situation, an attacker can remove the entire token parameter.
### Token is not tied to user session
Some applications do not validate the token belongs to the same session as the user making the request. Instead, it maintains a global pool of issued tokens and accepts any token so along as it exists within this pool. The attacker can login with their own account, obtain a valid token and feed it to the victim.
### Token is tied to a non-session cookie
In a variation of the above, some applications do tie the token to a cookie, but not to the one that tracks the session. This can easily occur if an application employs two different frameworks, one for session handling and one for CSRF protection.
This is harder to exploit, but still possible. If the website has any behaviour that would allow an attacker to set a cookie in a victim's browser, then the attack is possible. The attacker can login with their own account, obtain a valid tok