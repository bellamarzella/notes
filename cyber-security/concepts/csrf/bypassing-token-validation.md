## Common flaws in CSRF token validation
### Validation depends on request method
Some applications will correctly validate the token, but only when the request uses the `POST` method, meaning attackers can just switch to a `GET` request to bupass the validation entirely (or vice-versa).
### Validation depends on token being present
Some applications will correctly validate the token when it is present, but just skip it if it is omitted. In this situation, an attacker can just 
