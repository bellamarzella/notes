## Reflected XSS (Non-Persistent)

### What is it?
The payload is part of the request sent to the server and is immediately reflected back in the server's HTTP response. It is non-persistent, meaning the payload has to be manually triggered per victim and is never saved.

### What might it look like?
The attacker crafts a malicious URL containing a script and sends it to a victim. When the victim clicks the link, the browser sends the request, and the server immediately echoes the payload back into the HTML response, triggering the script on the victim's machine.

For example, an attacker targets a website's search feature. They swap a normal search term in the URL with a malicious script (e.g., `?search=<script>...`). When the victim clicks this link, the server generates a search results page that blindly echoes that script back into the page text, executing it instantly.
## Stored XSS 
### What is it?
The payload is stored on the server (usually in a database, file system or log file) and is pulled and embedded onto the page, being shown to any user who visits the site.
### What might it look like?
Stored XSS is a two phased attack:
#### Injection
The attacker first needs to get the payload onto the server. They input a malicious script somewhere that is saved, such as in the form of a comment on a blog.
#### Execution
The payload sits on the server until it is pulled. For example, a different user goes to that same blog and loads the comments, at which point the attackers script is pulled and triggered on the victims machine.

## DOM-based XSS
### What is it?
Unlike the other two, the server is completely oblivious and uninvolved to DOM-XSS. The server sends a normal, safe response to the browser. The vulnerability exists entirely within the victim's browser after the page loads due to some client-side vulnerability.
### How it works
DOM-XSS relies on client-side JS executing something it shouldn't. It is defined by two components:
#### Source
A JS property an attacker can control from the outside world, such as a URL hash, URL query or `document.refferer`.
#### Sink
A dangerous JS function or DOM object can execute code if given raw text, such as `element.innerHTML`, `document.write`, `eval()` or legacy jQuery selectors like `$()`.

### What might it look like?
#### Setup
The attacker crafts a link where the URL contains a malicious script after a 
