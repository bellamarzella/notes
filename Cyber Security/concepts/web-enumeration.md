When performing [[service-scanning|service scanning]], we often come across web servers running on ports 80 and 443. These host web application(s) which often provide a considerable attack surface and are a very valuable target during a penetration test. 

**Web enumeration is the process of discovering hidden files, directories, subdomains, user accounts and software versions;** think of it as mapping out a web server.

## Fuzzing
Once we've discovered a web app, we want to see if we can uncover any hidden files or directories not intended for public access. We use a tool such as ffuf or [[gobuster]] to run thousands of inputs on a target and analyse how it responds.

## Banner Grabbing/