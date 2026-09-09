## Common flaws in CSRF token validation
### Validation depends on request method
Some applications will correctly validate the token, but only when the request uses the `POST` method, meaning attackers can 