We've seen common client-side mitigations in frame busting scripts, and consequently we've seen how they can be circumvented. Thus, server driven protocols have been devised that contrain iframe usage and mitigate against clickjacking.
# X-Frame-Options
X-Frames-Options was originally introduces as an unofficial response header in Internet Explorer 8 and rapidly adopted by other browser. It provides the website owner with control over the use of iframes or objects.

1. Prohibit the inclusion of a website in an iframe:
	`X-Frame-Options: deny`
2. Restrict framing to the same origin:
	`X-Frame-Options: sameorigin`
3. Or from a whitelisted website:
	``X-Frame-Options: allow-from https://normal-website.com``

X-Frame-Options is not implemented consistently across browsers (`allow-from` is not supported in Chrome version 76 or Safari 12 for example). However, when used in conjection with Content Security Policy as part of a multi-layer defence strategy it can provide an effective defence against clickjacking.
# Content Security Policy
CSP is a detection and prevention mechanism that provides mitigation against various attacks, including clickjacking. CSP is usually implemented in the web server as a return header of the form `Content-Security-Policy: [policy]`, where `[policy]` is a string of policy directives seperated by semicolons. The CSP provides the client browser with information about permitted sources of web resources that the browser can apply to the detection and interception of attacks.

The recommended clickjacking protection is to incorporate the `frame-ancestors` directive in the CSP. The `frame-ancestors 'none'` functions similarly to `X-Frame-Options: deny`, and `frame-ancestors 'self'` similarly to `X-Frame-Options: sameorigin`. 