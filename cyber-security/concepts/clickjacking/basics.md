# What is clickjacking?
Clickjacking is a type of **UI redressing** attack. An attacker sets up a webpage that consists of an invisible `iframe` linked to the legitimate page with malicious interactive elements (like a *Click here for your free gift!* button) overlaid over the real website's buttons to trick users into executing unintended actions while authenticated.

In this way, it is similar to [CSRF](obsidian://open?vault=cyber-security&file=concepts%2Fcsrf%2Fbasics) attacks, in that the goal is to get a user to execute an unintended action, but where CSRF forces the victim to do it without their knowledge, clickjacking tricks them into doing it themselves. Because these actions occur on the actual domain, CSRF tokens are placed into requests and passed to the server as they would in any normal session.

# Constructing a Basic Clickjacking Attack
The attacker incorporates the target website, like your bank or social media, as an `iframe` layer on top of the decoy website. The decoy doesn't necessarily have anything to do with the target, but the point is that its structured in such lead the victim into performing some action on the underlaid legitimate website.

For example:
```html
<head> 
	<style> 
		#target_website { 
			position:relative; 
			width:128px; 
			height:128px; 
			opacity:0.00001; 
			z-index:2; 
			} 
		#decoy_website { 
			position:absolute; 
			width:300px; 
			height:400px; 
			z-index:1; 
			} 
	</style> 
</head> 
... 
<body> 
	<div id="decoy_website"> 
	...decoy web content here... 
	</div> 
	<iframe id="target_website" src="https://vulnerable-website.com"> 
	</iframe> 
</body>
```

The target website iframe is positioned within the browser so that there is a precise overlap of the target action with the decoy website using appropriate width and height position values. Absolute and relative position values are used to ensure that the target website accurately overlaps the decoy regardless of screen size, browser type and platform. The z-index determines the stacking order of the iframe and website layers. The opacity value is defined as 0.0 (or close to 0.0) so that the iframe content is transparent to the user. Browser clickjacking protection might apply threshold-based iframe transparency detection (for example, Chrome version 76 includes this behavior but Firefox does not). The attacker selects opacity values so that the desired effect is achieved without triggering protection behaviours.

## Clickbandit
Creating a clickjacking POC tends to be tedious in practice, so we can use Burp's [Clickbandit](https://portswigger.net/burp/documentation/desktop/tools/clickbandit) (which is included in community edition!) instead. This lets us use our browser to perform the desired actions on a frameable page, then generates a HTML file with a suitable clickjacking overlay, allowing us to generate a POC in seconds.
# Clickjacking with prefilled form input
Some website forms allow form inputs to be pre-populated via GET parameters prior to submission (i.e., we don't need to trick the user into filling out the form, the URL we use to redirect them populates the form for us and we just get the victim to click the submit button). Others require actual text, which we'll need to trick the user into filling out.

# Frame busting
Clickjacking is possible when a website can be framed. Therefore, preventative techniques are based on restricting this. A common client-side protection is to use **frame busting/breaking** scripts. These can be implemented via add-ons or extensions such as NoScript. Scripts are usually crafted so that they:

- check and enforce the current application window is the main or top window
- make all frames visible
- prevent clicking on invisible frames
- intercept and flag potential clickjacking attacks to the user

Materially, they check whether the website is the main window, and if they aren't, they attempt some action to defend themselves:

```js
if (top !== self) { 
	// A simple framebuster. If the webpage isn't on top, redirect the top page to        itself. This is vulnerable as we'll see.
	top.location = self.location; 
}
if (top === self) { 
	// A more sophisticated technique. If the webpage isn't on top, make                  everything invisible, rendering the website unusable. An attacker would            have to disable scripts on the frame to circumvent this, which would likely        render the site, and by extension the attack, unusable.
	document.body.style.display = 'block'; 
}
```

Due to the flexibility of HTML, these can still be circumvented, and, because they are JavaScript, the browser's security settings may prevent their operation or the browser might just not support JavaScript in the first place.

An effective **workaround** for attackers is to use the **HTML5 iframe `sandbox`** attribute. When this is set with the `allow-forms` or `allow-scripts` values and `allow-top-navigation` is omitted, our first frame buster cannot react. It sees that it isn't the top frame and tries to redirect, but isn't allowed to because of `allow-top-navigation`'s absence.
# Combining clickjacking with DOM XSS 
The true potency of clickjacking is revealed when it is used as a vector for another attack, such as [[xss-types#DOM-based XSS |DOM-based XSS]]. Implementation of this is usually simple provided the attacker has identified the XSS exploit. This is combined with the `iframe` target so that the user clicks on something that then executes the XSS attack.
# Multistep Clickjacking
If desired attack necessitates multiple actions, such as placing an item into a shopping basket then checking out, the attacker may use multiple `<div>` tags or iframes. These attacks require considerable precision and care from the attacker if they are to be effective and stealthy.
