# Information

Name: HackPark

Difficulty: Medium

Description: Bruteforce a websites login with Hydra, identify and use a public exploit then escalate your privileges on this Windows machine!

## Deploy the vulnerable Windows machine

**Deploy the machine and access its web server.**

![image](https://user-images.githubusercontent.com/43668197/140530079-43ab7f11-5f6b-4f13-a6f4-48f2f8fd2ed4.png)

**Whats the name of the clown displayed on the homepage?**

Pennywise

If you are not familiar with the Stephen King's It then a reverse image search would get the answer.

##  Using Hydra to brute-force a login

We need to find a login page to attack and identify what type of request the form is making to the webserver. Typically, web servers make two types of requests, a GET request which is used to request data from a webserver and a POST request which is used to send data to a server.

You can check what request a form is making by right clicking on the login form, inspecting the element and then reading the value in the method field. You can also identify this if you are intercepting the traffic through BurpSuite (other HTTP methods can be found here).

We can find the login page under the sub menu on the top right.

![image](https://user-images.githubusercontent.com/43668197/140531194-932b41db-bcc8-4d09-86c4-320457c1fa61.png)

Sending fake information and capturing the burp request.

![image](https://user-images.githubusercontent.com/43668197/140531516-f1b0b366-af38-4f0e-9d85-743585d699a9.png)

**What request type is the Windows website login form using?**

post

Now we know the request type and have a URL for the login form, we can get started brute-forcing an account.

Run the following command but fill in the blanks:

hydra -l <username> -P /usr/share/wordlists/<wordlist> <ip> http-post-form

**Guess a username, choose a password wordlist and gain credentials to a user account!**
  
1qaz2wsx
  
![image](https://user-images.githubusercontent.com/43668197/140535050-20690484-f031-4f4d-ac8d-b30f087efbdc.png)
  
## Compromise the machine
  
![image](https://user-images.githubusercontent.com/43668197/140535190-3064ca26-fb3a-4cc7-bc33-3833ee9cfc54.png)
  
**Now you have logged into the website, are you able to identify the version of the BlogEngine?**
  
3.3.6.0
  
Use the exploit database archive to find an exploit to gain a reverse shell on this system.

![image](https://user-images.githubusercontent.com/43668197/140535469-aa2f36cf-012e-4ca3-8f50-b8dfc2dc1df0.png)

**What is the CVE?**
  
cve-2019-6714
  

Using the public exploit, gain initial access to the server.
  
Download the 46353.cs and rename it to PostView.ascx as instructed. Edit the ip to ur attacking machines ip and then upload this file on the blog as instructed. `http://10.10.10.10/admin/app/editor/editpost.cshtml`
  
![image](https://user-images.githubusercontent.com/43668197/140537518-e3bc2962-4621-49e2-a2d3-75e5143db827.png)
  
We don't actually have to create a post we just need the file to be uploaded. Then run an nc listener and navigate to `http://10.10.10.10/?theme=../../App_Data/files` as instructed in the exploit.

 ![image](https://user-images.githubusercontent.com/43668197/140537940-565cab27-6f9d-4399-b6d7-75c44c2b2155.png)


**Who is the webserver running as?**
  
iis apppool\blog

## Windows Privilege Escalation
  

Our netcat session is a little unstable, so lets generate another reverse shell using msfvenom.

If you don't know how to do this, I suggest completing the Metasploit room first!



Tip: You can generate the reverse-shell payload using msfvenom, upload it using your current netcat session and execute it manually!

No answer needed
You can run metasploit commands such as sysinfo to get detailed information about the Windows system. Then feed this information into the windows-exploit-suggester script and quickly identify any obvious vulnerabilities.

  ![image](https://user-images.githubusercontent.com/43668197/140545367-1c073549-fa8a-4ad9-b7e4-86d4abd9595c.png)

  ![image](https://user-images.githubusercontent.com/43668197/140545411-0836b510-d4b6-410d-be54-ed1fe0a7398f.png)

  ![image](https://user-images.githubusercontent.com/43668197/140545446-4d631edf-e8fa-4e6e-b728-11af93c877bb.png)

**What is the OS version of this windows machine?**

Windows 2012 R2 (6.3 Build 9600)


Further enumerate the machine.
  
![image](https://user-images.githubusercontent.com/43668197/140547599-6b68ebd6-d9ca-415c-95e5-380774a9b5fa.png)

  ![image](https://user-images.githubusercontent.com/43668197/140548635-d1af7c5f-7865-4d5d-993b-8f41859901b4.png)

**What is the name of the abnormal service running?**

WindowsScheduler  
 
![image](https://user-images.githubusercontent.com/43668197/140549503-a89d0153-4ee0-43d7-a3fd-ce927719abd6.png)

Hmm, wservice isnt the answer. Had to dig further to find the correct binary.
  
![image](https://user-images.githubusercontent.com/43668197/140549888-6991a38e-cba1-40a8-8719-9c11147256bb.png)
  
**What is the name of the binary you're supposed to exploit?**
  
 Message.exe
  
![image](https://user-images.githubusercontent.com/43668197/140550695-4e4cbbc7-feb5-4b6f-9e38-106785def999.png)

![image](https://user-images.githubusercontent.com/43668197/140550781-991049dd-5972-4f9c-84a8-77318204ec6c.png)

![image](https://user-images.githubusercontent.com/43668197/140550760-6f20d0ac-9dbe-4590-b4bf-45b1f0b4e2c0.png)

![image](https://user-images.githubusercontent.com/43668197/140551075-fbe5dc5f-1882-4b20-b391-05f82b0cd7ef.png)

**What is the user flag (on Jeffs Desktop)?**
  
759bd8af507517bcfaede78a21a73e39
  
**What is the root flag?**
  
7e13d97f05f7ceb9881a3eb3d78d3e72
  
  ## Privilege Escalation Without Metasploit
  
  Just use `nc -lnvp port` instead of msfconsole.
  
  Use winpeas or `systeminfo | findstr /i date` for the answer.
  
  **Using winPeas, what was the Original Install time? (This is date and time)**
  
  8/3/2019, 10:43:23 AM
  
