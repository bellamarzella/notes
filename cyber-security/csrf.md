## What is CSRF?
Cross-site request forgery is a websec vulnerability that allows an attacker to induce users to perform actions they do not intend to. It allows an attacker to partly circumvetn the *same origin policy*, which is designed to prevent different websites from interfering with eachother.
## What are the potential impacts of a CSRF attack?
Causing the victim to carry out some action unintentionally could allow the attacker to gain control over a victims account, for example by changing their password, which could lead to a full system compromise if the victim has a privileged role.
## How does it work?
There are three conditions that must be in place for a CSRF attack to be possible:
### A desirable action
There is an action within the application that the attacker has a reason to induce. There's no point in forcing a user to, say, transfer money to another account they still have control over (unless, for example, this could be leveraged for a scam).
### Cookie-based session handling
Performing this action involving issuing one or more HTTP requests and the application relies solely on session cookies to identify who made the request. There is **no other mech**
