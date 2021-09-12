# Information

Name: Wreath

Difficulty: Easy

Description: Learn how to pivot through a network by compromising a public facing web machine and tunnelling your traffic to access other machines in Wreath's network.

Note: Not all tasks have questions that require answers and so those tasks are not listed here.

## Task 5 - Webserver Enumeration
We always start with an nmap scan since it may take awhile to complete.

```
┌──(root💀kali)-[~/ctfs/thm]
└─# export IP=10.200.188.200     
──(root💀kali)-[~/ctfs/thm]
└─# nmap -p-15000 -vv $IP -oG ~/ctfs/thm/wreath
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-11 11:49 EDT
Initiating Ping Scan at 11:49
Scanning 10.200.188.200 [4 ports]
Completed Ping Scan at 11:49, 0.19s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:49
Completed Parallel DNS resolution of 1 host. at 11:49, 0.02s elapsed
Initiating SYN Stealth Scan at 11:49
Scanning 10.200.188.200 [15000 ports]
Discovered open port 22/tcp on 10.200.188.200
Discovered open port 80/tcp on 10.200.188.200
Discovered open port 443/tcp on 10.200.188.200
Discovered open port 10000/tcp on 10.200.188.200
SYN Stealth Scan Timing: About 33.37% done; ETC: 11:51 (0:01:02 remaining)
SYN Stealth Scan Timing: About 58.62% done; ETC: 11:51 (0:00:43 remaining)
Completed SYN Stealth Scan at 11:51, 126.97s elapsed (15000 total ports)
Nmap scan report for 10.200.188.200
Host is up, received echo-reply ttl 63 (0.11s latency).
Scanned at 2021-09-11 11:49:46 EDT for 127s
Not shown: 14995 filtered ports
Reason: 14863 no-responses and 132 admin-prohibiteds
PORT      STATE  SERVICE          REASON
22/tcp    open   ssh              syn-ack ttl 63
80/tcp    open   http             syn-ack ttl 63
443/tcp   open   https            syn-ack ttl 63
9090/tcp  closed zeus-admin       reset ttl 63
10000/tcp open   snet-sensor-mgmt syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 127.36 seconds
           Raw packets sent: 44879 (1.975MB) | Rcvd: 140 (9.832KB)
```

**How many of the first 15000 ports are open on the target?**

4

We can see that there are 4 open ports and 1 closed port.

**What OS does Nmap think is running?**

centos

```
┌──(root💀kali)-[~/ctfs/thm]
└─# nmap -p 22,80,443,10000 -sV -oA ~/ctfs/thm/wreath $IP 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-11 12:28 EDT
Nmap scan report for 10.200.188.200
Host is up (0.11s latency).

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.0 (protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1c)
443/tcp   open  ssl/http Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1c)
10000/tcp open  http     MiniServ 1.890 (Webmin httpd)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.11 seconds
```

Running the nmap scan again to check services with the ports we found reveals that nmap believes this to be a centos OS.

**Open the IP in your browser -- what site does the server try to redirect you to?**

https://thomaswreath.thm/

```
──(root💀kali)-[~/ctfs/thm]
└─# curl http://10.200.188.200                                                     
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="https://thomaswreath.thm">here</a>.</p>
</body></html>
```

We must add this line to the /etc/hosts file to access the page:

10.200.188.200  thomaswreath.thm

**Read through the text on the page. What is Thomas' mobile phone number?**

447821548812

Found simply by scrolling to the bottom of the page.

![image](https://user-images.githubusercontent.com/43668197/132990849-e753d568-84d2-48a6-91a8-d869d5c3d06c.png)

**Look back at your service scan results: what server version does Nmap detect as running here?**

MiniServ 1.890 (Webmin httpd)

```
10000/tcp open  http     MiniServ 1.890 (Webmin httpd)
```

**What is the CVE number for this exploit?**

CVE-2019-15107

https://www.exploit-db.com/exploits/47230

## Task 6 - Webserver Exploitation

**Which user was the server running as?**

root

![image](https://user-images.githubusercontent.com/43668197/132992604-0f44d7e3-36d5-4a22-8297-c69abca65c1f.png)

**What is the root user's password hash?**

$6$i9vT8tk3SoXXxK2P$HDIAwho9FOdd4QCecIJKwAwwh8Hwl.BdsbMOUAd3X/chSCvrmpfy.5lrLgnRVNq6/6g0PxK9VqSdy47/qKXad1

We cat out the /etc/shadow file

![image](https://user-images.githubusercontent.com/43668197/132992914-c5e7c8e1-eec9-4529-a994-dc93e9d965aa.png)

**You won't be able to crack the root password hash, but you might be able to find a certain file that will give you consistent access to the root user account through one of the other services on the box.**

**What is the full path to this file?**

/root/.ssh/id_rsa

## Task 8 - Pivoting High-level Overview

**Which type of pivoting creates a channel through which information can be sent hidden inside another protocol?**

tunnelling

**Research: Not covered in this Network, but good to know about. Which Metasploit Framework Meterpreter command can be used to create a port forward?**

portfwd

https://www.offensive-security.com/metasploit-unleashed/portfwd/

## Task 9 - Pivoting Enumeration

**What is the absolute path to the file containing DNS entries on Linux?**

/etc/resolv.conf

**What is the absolute path to the hosts file on Windows?**

C:\Windows\System32\drivers\etc\hosts

**How could you see which IP addresses are active and allow ICMP echo requests on the 172.16.0.x/24 network using Bash?**

for i in {1..255}; do (ping -c 1 172.16.0.${i} | grep "bytes from" &); done

## Task 10 - Pivoting Proxychains & Foxyproxy

**What line would you put in your proxychains config file to redirect through a socks4 proxy on 127.0.0.1:4242?**

SOCKS4 127.0.0.1:4242

**What command would you use to telnet through a proxy to 172.16.0.100:23?**

proxychains telnet 172.16.0.100:23

**You have discovered a webapp running on a target inside an isolated network. Which tool is more apt for proxying to a webapp: Proxychains (PC) or FoxyProxy (FP)?**

FP

## Task 11 - Pivoting SSH Tunnelling / Port Forwarding

**If you're connecting to an SSH server from your attacking machine to create a port forward, would this be a local (L) port forward or a remote (R) port forward?**

L

**Which switch combination can be used to background an SSH port forward or tunnel?**

-fN

**It's a good idea to enter our own password on the remote machine to set up a reverse proxy, Aye or Nay?**

nay

**What command would you use to create a pair of throwaway SSH keys for a reverse connection?**

ssh-keygen

**If you wanted to set up a reverse portforward from port 22 of a remote machine (172.16.0.100) to port 2222 of your local machine (172.16.0.200), using a keyfile called id_rsa and backgrounding the shell, what command would you use? (Assume your username is "kali")**

ssh -L 2222:172.16.0.100:22 kali@172.16.0.200 id_rsa -fN

**What command would you use to set up a forward proxy on port 8000 to user@target.thm, backgrounding the shell?**

ssh -D 8000 user@target.thm -fN

**If you had SSH access to a server (172.16.0.50) with a webserver running internally on port 80 (i.e. only accessible to the server itself on 127.0.0.1:80), how would you forward it to port 8000 on your attacking machine? Assume the username is "user", and background the shell.**

ssh -L 8000:172.16.0.50:80 user@127.0.0.1 -fN

## Task 12 - Pivoting plink.exe

**What tool can be used to convert OpenSSH keys into PuTTY style keys?**

puttygen

