# What is the DOM?
The Document Object Model is a browser's hierarchical representation of the elements on the page. Websites can use JavaScript to manipulate the nodes and objects of the DOM, as well as their properties. The ability to manipulate the DOM isn't a problem in and of itself, in fact, it's integral to how modern websites work. However, JavaScript that handles data insecurely can enable various attacks.

DOM-based vulnerabilities arise when the websites contains JS that takes an **attacker-controllable value, the source** and passes it into a **dangerous function, the sink**.

# Taint-flow vulnerabilities
Many DOM-based vulnerabilities can be traced back to problems with the way client-side code manipulates attacker-controllable data.
## What is taint flow?
