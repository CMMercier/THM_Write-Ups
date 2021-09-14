# Information

Name: Brute It

Difficulty: Easy

Description:
In this box you will learn about:

- Brute-force

- Hash cracking

- Privilege escalation

## Recon

**Search for open ports using nmap.**

Port and service scan with nmap:

![image](https://user-images.githubusercontent.com/43668197/133253252-55a0bf28-4b1b-4067-b72b-32df4b55868d.png)

We find two ports open here. The way in is likely through port 80.

**How many ports are open?**

2

**What version of SSH is running?**

OpenSSH 7.6p1

**What version of Apache is running?**

2.4.29

**Which Linux distribution is running?**

ubuntu

**Search for hidden directories on web server.**

![image](https://user-images.githubusercontent.com/43668197/133255251-f2ee3039-82cf-418b-9097-80373db9c2a8.png)

**What is the hidden directory?**

/admin

##  Getting a shell

Head over to http://10.10.176.190/admin/. We get a login page and checking the source code reveals the username is admin.

![image](https://user-images.githubusercontent.com/43668197/133255535-c8b05009-1e17-4f3a-82f2-db1e3d1eae2b.png)

Now we use Hydra to brute force the password.

![image](https://user-images.githubusercontent.com/43668197/133258039-4915103c-1ce2-46c7-b679-523307420aaf.png)

**What is the user:password of the admin panel?*

admin:xavier

Logging in brings us to the admin panel which we discover is still under development but gives us an rsa key to download and the web flag.

![image](https://user-images.githubusercontent.com/43668197/133258328-34785a3e-5466-469c-8941-caa2196f82ab.png)

If we attempt to ssh with this key we discover it requires a passphrase.

![image](https://user-images.githubusercontent.com/43668197/133258637-be5ee76e-31cd-418d-b767-633972d736d4.png)

So we must use John the Ripper to crack it.

![image](https://user-images.githubusercontent.com/43668197/133259547-898c3b5b-a9de-4a32-ab20-83e4a6cab826.png)

**What is John's RSA Private Key passphrase?**

rockinroll

**user.txt**

THM{a_password_is_not_a_barrier}

**Web flag**

THM{brut3_f0rce_is_e4sy}

# Privilege Escalation

The first thing we want to do is check what sudo commands the user has access to.

![image](https://user-images.githubusercontent.com/43668197/133260159-4663d26f-5158-451a-ab6d-6696c3826834.png)

The go to site to look up how to exploit this is GTFOBins. https://gtfobins.github.io/gtfobins/cat/#sudo

![image](https://user-images.githubusercontent.com/43668197/133260874-f1a84226-a68b-4bdd-8d67-3768c4bd8435.png)

And this is as simple as using cat to read shadow and get the root password hash.

![image](https://user-images.githubusercontent.com/43668197/133261964-fe9b30fd-92f3-4019-a8a5-14db540dec78.png)

For this I will use hashcat.

![image](https://user-images.githubusercontent.com/43668197/133262636-dc07caa3-ded1-4cbc-9e7a-e39dc0fc1fa7.png)

**What is the root's password?**

football

With that I just switch user with su and grab the root flag.

![image](https://user-images.githubusercontent.com/43668197/133263126-243ee4bc-de42-44bc-b220-0344ef3bf479.png)

**root.txt**

THM{pr1v1l3g3_3sc4l4t10n}
