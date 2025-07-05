**Web - Cat Club**

Summary

A .zip file with the website's source code was provided. Looking through it, we found vulnerabilities that allowed SSTI exploitation and flag retrieval.


Initial Discovery

The site has a login page and a gallery showing cat images. The username is reflected on the gallery page, which pointed to a possible SSTI:

![test](Images/20241116214032.png)

But during registration, only alphanumeric usernames are allowed. The site uses JWTs, and with the source code, we were able to forge a custom token.

Endpoint Analysis
We found an interesting endpoint in the code:

![test](Images/20241116214439.png) 

and

![test](Images/20241116214456.png)

Using this, we created a JWT with the HS256 algorithm and injected #{9*9} in the username field to test for SSTI:

![test](Images/20241116214654.png)

The test worked:

![test](Images/20241116214859.png)

Exploitation
The code showed that PugJS was used as the template engine. Using a payload from HackTricks, we went further. The page didn’t show a response, so we tried running id to confirm execution:

![test](Images/20241116215217.png)

The response was empty:

![test](Images/20241116215239.png)

We then used curl to trigger an external request:

![test](Images/20241116215537.png)

It worked. The command ran, and we got a response:

![test](Images/20241116215517.png)

Once code execution was confirmed, getting the flag was easy:

![test](Images/20241116215724.png)
