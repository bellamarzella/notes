## What is CSRF?
Cross-site request forgery is a websec vulnerability that allows an attacker to induce users to perform actions they do not intend to. It allows an attacker to partly circumvetn the *same origin policy*, which is designed to prevent different websites from interfering with eachother.
## What are the potential impacts of a CSRF attack?
Causing the victim to carry out some action unintentionally could allow the attacker to gain control over a victims account, for example by changing their password, which could lead to a full system compromise if the victim has a privileged role.
## How does it work?
There are three conditions that must be in place for a CSRF attack to be possible:
### A desirable action
There is an action within the application that the attacker has a reason to induce. There's no point in forcing a user to, say, transfer money to another account they still have control over (unless, for example, this could be leveraged for a scam).
### Cookie-based session handling
Performing this action involving issuing one or more HTTP requests and the application relies solely on session cookies to identify who made the request. There is **no other mechanism** in place to verify who is making a request.
### No unpredictable request parameters
The requests that perform the action do not contain parameters whose value the attacker cannot determine or guess. For example, a password changing function is vulnerable if the user must input their existing password to change it.
### Example
Suppose an application contains a function that lets the user change the email associated with their account. When a user performs this action, they make a HTTP request that looks like the following:

```HTML
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 30 
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE 

email=wiener@normal-user.com
```

1. **A desirable action**: Changing the email address on a user's account is of interest to an attacker. Following this, the attacker would typically be able to trigger a password reset and take full control of the account.
2. **Cookie-based session handling**: The application uses a session cookie (`session=yvth...`) to track which user issued the request with no other mechanisms in place to do this.
3. **No unpredictable request parameters**: The attacker can easily determine the values of the other parameters needed for the request.

With these conditions all being true, the attacker can construct a webpage with the following HTML:

```HTML
<html> 
	<body> 
		<form action="https://vulnerable-website.com/email/change"                          method="POST"> 
			<input type="hidden" name="email" value="pwned@evil-user.net" /> 
		</form> 
		<script> 
			document.forms[0].submit(); 
		</script> 
	</body> 
</html>
```

If the victim visits this webpage:
1. The attacker's page triggers a HTTP request to the vulnerable website.
2. If the user is currently logged in, their browser will automatically include their session cookie in the request.
3. The vulnerable website processes the request in the normal way, treating it as having been made by the victim and changing their email.

> **Note**
> XSRF is usually described in relation to cookie-based session handling, it can also arise in other contexts where the application automatically adds some user credentials to requests, such as HTTP Basic authentication and certificate-based authentication.

### Constructing a CSRF attack
Manually writing the HTML for a CSRF attack can be annoying, especially with many parameter,  but we can use tools like [CSRFShark](https://csrfshark.github.io/) to make it easier for us. Burp Suite professional also has a PoC generator, keyword being *professional*!  

### Delivering a CSRF attack
The delivery mechanisms for a CSRF attack are essentially the same as for reflected XSS. Typically 