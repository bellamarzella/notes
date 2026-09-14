# What is the DOM?
The Document Object Model is a browser's hierarchical representation of the elements on the page. Websites can use JavaScript to manipulate the nodes and objects of the DOM, as well as their properties. The ability to manipulate the DOM isn't a problem in and of itself, in fact, it's integral to how modern websites work. However, JavaScript that handles data insecurely can enable various attacks.
# Taint-flow vulnerabilities
Many DOM-based vulnerabilities can be traced back to problems with the way client-side code manipulates attacker-controllable data.
## What is taint flow?
### Sources
A source is a JS property that can **accept data that could be attacker controlled**. For example, `location.search` reads input from the query string, which is simple for an attacker to control. Ultimately, any property that can be controlled by the attacker is a potential source. Some other examples include the referring URL (`document.referrer`), the user's cookies (`document.cookie`) and web messages.
### Sinks
A sink is a potentially dangerous JS function on DOM object that can **cause undesirable effects if attacker-controlled data is passed to it.** For example, the `eval()` function is a JS sink because it processes the argument passed to it as JS. `document.body.innerHTML` is a HTML sink because it potentially allows attackers to inject malicious HTML and execute arbitrary JS code.


**DOM-based vulnerabilities arise when the websites contains JS that takes an attacker-controllable value, the source and passes it into a dangerous function, the sink.**

The most common source is the URL, usually accessed with the `location` object. An attacker can can construct a link to send a victim to a vulnerable page with a payload in the query string. Consider the following code:

``` js
goto = location.hash.slice(1) 
if (goto.startsWith('https:')) {   
	location = goto; 
}
```

This is vulnerable because `location.hash` is a source that is handled unsafely. 