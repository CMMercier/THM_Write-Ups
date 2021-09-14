# Information

Name: Source

Difficulty: Easy

Description: Webmin, Metasploit

## Recon

**Search for open ports using nmap.**

![image](https://user-images.githubusercontent.com/43668197/133313658-088612db-1e13-45d5-808e-df07c1bd9139.png)

There is a webmin server active at port 10000. Navigating to the site gives us a webmin login page.

![image](https://user-images.githubusercontent.com/43668197/133314127-e50d168c-b89e-458f-a95e-cbf4b9ca87fa.png)

The source code does not contain any useful information. But what we do know is that this is running webadmin so its time to load up msfconsole and see if theres
any promising looking exploits available.https://www.webmin.com/exploit.html

![image](https://user-images.githubusercontent.com/43668197/133316122-411493c4-4143-4ed8-9603-cdb160d971c0.png)

The backdoor looks promising. We want to give the command use 5 to tell metasploit to use this exploit and then check our options. There is a few options we must set
for this to work.

![image](https://user-images.githubusercontent.com/43668197/133316598-59771edf-015e-4d92-9446-2015623964d0.png)

And I wasn't able to get metasploit to actually work even after trying to follow the provided guide on TryHackMe. Luckily I found another way online using 
https://github.com/MuirlandOracle/CVE-2019-15107/blob/main/CVE-2019-15107.py and this got me in.

![image](https://user-images.githubusercontent.com/43668197/133327914-5476815d-cbfe-44e0-a4c8-dc789ecadd67.png)

We instantly have root access and can get root flag from cat /root/root.txt and user flag from cat /home/dark/user.txt.

