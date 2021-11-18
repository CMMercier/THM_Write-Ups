# Information

Name: Erit_Securus_I

Difficulty: Easy

Description: Learn to exploit the BoltCMS software by researching exploit-db.

## Reconnaissance

This part asks us to use `nmap` scanner but I have been trying out another tool that I currently prefer called `RustScan`. So far it is much faster than nmap while
having all the same capabilities and more.

![image](https://user-images.githubusercontent.com/43668197/142428816-b1c8059c-8da4-4c31-b355-c2f361de14ed.png)

**How many ports are open?**

2

**What ports are open? Comma separated, lowest first:**

22,80

## Webserver

We can answer the next question by simply looking down to the bottom of the website:

![image](https://user-images.githubusercontent.com/43668197/142429052-7f1e7657-5fd3-4667-8359-dc4929c28993.png)

Or using the handle little browser extension called `Wappalizer`:

![image](https://user-images.githubusercontent.com/43668197/142429157-e8734297-92e5-426c-ac89-b12b9eec7832.png)

**What CMS is the website built on?**

Bolt

##  Exploit

This next part actually provides us with the exploit. But checking out the exploit shows that we must first aquire login details.

![image](https://user-images.githubusercontent.com/43668197/142429354-4c265fcd-7315-4ada-a530-573dc9388312.png)

**In the exploit from 2020-04-05, what language is used to write the exploit?**

Python

Directory enumeration attempts were unsuccessful so I had to search for Bolt documentation to find where the login page was. After having to reset the machine so many
times trying to enumerate it made me laugh a little to discover how easy it would have been just to go straight to the documentation and find out the login is simply
located at `http://machineip/bolt`.

![image](https://user-images.githubusercontent.com/43668197/142442342-9b8fbeb2-3305-44ad-b016-d789f02686f9.png)

The hint on TryHackMe gives us the login details `admin:password`. I try these on the Bolt login and get in.

## Reverse shell

Follow the provided instructions. I changed the `echo '<?php system($_GET["c"]);?>' > c.php` to `echo '<?php system($_GET["c"]);?>' > cmd.php` so that the rest of the
commands worked. Else the page will tell you there is no `cmd.php`.

![image](https://user-images.githubusercontent.com/43668197/142452798-993c1c42-d5f4-4f80-b232-71845b377dfd.png)

**What is the username of the user running the web server?**

www-data

## Priv esc

![image](https://user-images.githubusercontent.com/43668197/142453436-f43755e6-8d84-4d0b-b408-747f4784f873.png)

![image](https://user-images.githubusercontent.com/43668197/142453744-c0911b78-affc-4ed5-842c-0fe741460dcd.png)

We can now switch into the user `wileec`

![image](https://user-images.githubusercontent.com/43668197/142454097-839c69a1-48f3-4521-9da8-dc66f2aa24de.png)

**What is the users password?**

snickers

And the flag is located in the users home directory.

**Flag 1**

THM{Hey!_Welcome_in}

## Pivoting

![image](https://user-images.githubusercontent.com/43668197/142454767-d4c0ff90-0489-4f51-9954-44e02a990271.png)

**User wileec can sudo! What can he sudo?**

(jsmith) NOPASSWD: /usr/bin/zip

## Privesc #2

The sudo option for zip from https://gtfobins.github.io/gtfobins/zip/#sudo works but we have to modify it to the user `jsmith`.

![image](https://user-images.githubusercontent.com/43668197/142456099-fec55663-50b6-4c1a-b1b3-1187e9e1a25a.png)

**Flag 2**

THM{Welcome_Home_Wile_E_Coyote!}

## Root

![image](https://user-images.githubusercontent.com/43668197/142456458-a7a16bb7-2a05-4620-a04e-85e969925fbb.png)

User jsmith has sudo access to all commands.

**What sudo rights does jsmith have?**

(ALL : ALL) NOPASSWD: ALL

If you know where the file is you wish to read you can just sudo cat the file like so:

![image](https://user-images.githubusercontent.com/43668197/142457123-23f0ee97-1df8-4865-a109-21d0657fab83.png)

But as we want to own the machine as root we can simply use `sudo -s` for this.

![image](https://user-images.githubusercontent.com/43668197/142457368-df8b3543-88a8-4540-bae5-7f43ebe4da66.png)

**Flag 3**

THM{Great_work!_You_pwned_Erit_Securus_1!}
