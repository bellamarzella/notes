# What is clickjacking?
Clickjacking is a type of **UI redressing** attack. An attacker sets up a webpage that consists of an invisible `iframe` linked to the legitimate page with malicious interactive elements (like a *Click here for your free gift!* button) overlaid over the real website's buttons to trick users into executing unintended actions while authenticated.

In this way, it is similar to [CSRF](obsidian://open?vault=cyber-security&file=concepts%2Fcsrf%2Fbasics) attacks, in that the goal is to get a user to execute an unintended action, but where CSRF forces the victim to do it without their knowledge, clickjacking tricks them into doing it themselves. Because these actions occur on the actual domain, CSRF tokens are placed into requests and passed to the server as they would in any normal session.

# Constructing a Basic Clickjacking Attack
The attacker incorporates the target website, like your bank or social media, as an `iframe` layer on top of the decoy website. The decoy doesn't necessarily have anything to do with the target, but the point is that its structured in such lead the victim into performing some action on the underlaid legitimate website.

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
</head> ... <body> <div id="decoy_website"> ...decoy web content here... </div> <iframe id="target_website" src="https://vulnerable-website.com"> </iframe> </body>
```

