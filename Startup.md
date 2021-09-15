# Information

Name: Startup

Difficulty: Easy

Description: Abuse traditional vulnerabilities via untraditional means.

## Recon

**Search for open ports using nmap.**

![image](https://user-images.githubusercontent.com/43668197/133429611-fcab3718-5245-4705-900a-3abdf6f658a0.png)

Firstly we have an FTP server on port 21 open with Anonymous FTP login allowed with a text and jpg file to check out. We also have a webserver over on port 80 afterwards.

![image](https://user-images.githubusercontent.com/43668197/133431786-c8ad8e4e-522e-4e21-9587-5cd440033447.png)

![image](https://user-images.githubusercontent.com/43668197/133431904-621d4d13-0236-4733-a654-f9390be6c9c9.png)

We get a possible login name Maya.

![image](https://user-images.githubusercontent.com/43668197/133433928-6eb00f2e-d115-4da6-9c41-0b67c9177633.png)

We also have stego with data hidden in this jpg file. I wasn't able to find anything with this so it may just be a rabbit hole.Taking a look at the website page 
and source code next.

![image](https://user-images.githubusercontent.com/43668197/133444855-2777bbb1-514f-451f-877c-a6f77b928245.png)

Nothing obvious here but ill run a gobuster scan in case.

![image](https://user-images.githubusercontent.com/43668197/133451487-687a08a3-6d95-46ea-ac49-76ab841b9e89.png)

Navigating to the /files directory to see whats there.

![image](https://user-images.githubusercontent.com/43668197/133451581-8284f203-6919-4120-b464-c7a38a0581d6.png)

It seems to house the same files as the ftp server. So we should be able to upload a file onto the ftp and get an easy reverse shell.

## Initial Access

![image](https://user-images.githubusercontent.com/43668197/133456703-afd916c5-bbd5-465f-a114-07c0282972b5.png)

Start a nc listener and navigate to the reverse shell php file on the website.

![image](https://user-images.githubusercontent.com/43668197/133457124-6d7dc34e-7117-4b39-8349-bb4cee45e4e7.png)

![image](https://user-images.githubusercontent.com/43668197/133457185-2f48a5d5-e4e6-48ab-adf0-f4c4c611202b.png)

We have access to www-data. Navigating to /home reveals just one user named lennie.

![image](https://user-images.githubusercontent.com/43668197/133457954-d12a3c03-c6fc-409f-8b62-937f6c14e162.png)

I have a look around and find the recipe.txt file at the root of the system along with a strange /incidents folder.
Within the folder called /incidents which contains a pcap file. I sent this file back to my system using nc and analyze it with wireshark.
LinEnum reveals two more users: vagrant and ftpsecure and that vagrant seems to have logged in remotely.

**What is the secret spicy soup recipe?**

love

![image](https://user-images.githubusercontent.com/43668197/133466193-935cf54b-78fb-4e17-9ffc-c0ff90fcfd7d.png)

Setting the filter to http quickly reveals that someone has been here before and uploaded their own shell.php, likely vagrant.

![image](https://user-images.githubusercontent.com/43668197/133460085-f295f103-9196-48d7-8904-46207ad5ae1b.png)

Analyzing this with ettercap revels a password likely belonging to lennie.

```
ettercap -T -r suspicious.pcapng
```

![image](https://user-images.githubusercontent.com/43668197/133462274-c778cc95-6517-4f28-a19b-b77f6caf646b.png)

## Priv Esc

Now we can ssh in as lennie and get the user flag.

**What are the contents of user.txt?**

THM{03ce3d619b80ccbfb3b7fc81e46c0e79}

There is a script inside the directory which is not writable by the current user but it executes another script named /etc/print.sh which is writable by lennie.

![image](https://user-images.githubusercontent.com/43668197/133464013-c947d2e5-2de5-457a-83ea-0355f8a271f1.png)

Adding a bash shell to the /etc/print.sh from http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet.

![image](https://user-images.githubusercontent.com/43668197/133464300-7ee8decc-d720-4c17-a5d0-41d60590ac67.png)

And we are in!

![image](https://user-images.githubusercontent.com/43668197/133464670-d8e31189-133c-4ba4-888b-e98827f4b5da.png)

**What are the contents of root.txt?**

THM{f963aaa6a430f210222158ae15c3d76d}



