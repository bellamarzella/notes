# What is clickjacking?
Clickjacking is a type of **UI redressing** attack. An attacker sets up a webpage that consists of an iframe linked to the legitimate page with malicious interactive elements (like a *Click here for your free gift!* button) overlaid invisibly over the real website's buttons to trick users into executing unintended actions while authenticated.

In this way, it is similar to [CSRF](obsidian://open?vault=cyber-security&file=concepts%2Fcsrf%2Fbasics) attacks, in that the goal is to get a user to execute an unintended action, but where CSRF forces the victim to do it without their knowledge, clickjacking tricks them into doing it themselves.

