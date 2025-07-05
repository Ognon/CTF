**Web - BioCorp**

Summary
A .zip file with the website's source code was provided. After checking it, we found vulnerabilities that gave access to sensitive features and led to flag extraction.

Initial Discovery
The source code had a panel.php file that needed a specific HTTP header to be accessed:

![test](Images/20241116212236.png)



![test](Images/20241116222648.png)

Once authenticated, a "Control Panel" button showed up and redirected to this page:

![test](Images/20241116222746.png)

Source Code Analysis
By looking at the page source, we found extra elements worth digging into:

![test](Images/20241116222820.png)

XXE Exploitation
The structure pointed to a possible XML External Entity (XXE) vulnerability. We tried this payload:

![test](Images/20241116223440.png)

The attack worked. We accessed the target resource and got the flag:


![test](Images/20241116223522.png)

