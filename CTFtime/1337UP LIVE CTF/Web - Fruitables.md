**Web - Fruitables**

Summary
This challenge didn’t include source code, so everything had to be done manually. We found key endpoints, exploited an SQL injection, bypassed file upload restrictions, got RCE, and retrieved the flag.


Initial Reconnaissance
Without source code, we used gobuster for directory enumeration and found useful endpoints:

![test](Images/20241116220024.png)

Two important ones:

    /upload: Likely for file uploads.

    /account.php: A login page. Registration was disabled, and no credentials were available.
    
SQL Injection
Testing the username field with a single quote (') showed a possible SQL injection:

![test](Images/20241116220206.png)

Manual exploitation didn’t work, so we used sqlmap to automate it:

![test](Images/20241116220335.png)


![test](Images/20241116220349.png)

We cracked the admin password and logged in. The dashboard had a file upload option:

![test](Images/20241116220430.png) 

File Upload Bypass
First attempts to upload a web shell were blocked:

![test](Images/20241116220720.png)

To bypass the filter, we:

    Renamed the file with a .jpg extension

    Added a valid JPEG header

    Set Content-Type to image/jpeg

The file upload worked:

![test](Images/20241116221201.png)


![test](Images/20241116221318.png) 

We accessed the uploaded shell, ran commands on the server, and got the flag:

![test](Images/20241116221410.png)
