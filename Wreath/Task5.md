# Information

Name: Wreath

Difficulty: Easy

Description: Learn how to pivot through a network by compromising a public facing web machine and tunnelling your traffic to access other machines in Wreath's network.


## Webserver Enumeration
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
