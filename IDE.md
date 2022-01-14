# Information

Name: IDE

Difficulty: Easy

Description: An easy box to polish your enumeration skills!

## Network enumeration

**Search for open ports using rustscan.**

![image](https://user-images.githubusercontent.com/43668197/149426061-818ac1a0-b994-4d38-9f51-a39dc64a76b5.png)

We find four open ports here. The first thing to check would be the open ftp on port 21. Especially since anonymous login is allowed.

![image](https://user-images.githubusercontent.com/43668197/149426213-c3647bd5-9933-4d87-b32f-3c4d15709452.png)

This part attempts to be a little sneaky. We have to enter the directory `...` and get the file named `-`.

![image](https://user-images.githubusercontent.com/43668197/149426337-0068f593-945f-4972-87f7-748a60b44c0c.png)

Rename the file after you transfer it to you system and cat out the contents. Theres a note about the user `john` having a default password to login. Also take note of
the username `drac` incase its useful later on.

![image](https://user-images.githubusercontent.com/43668197/149426448-dc16b9ca-d77b-4338-9327-f9ee6b48dd0c.png)

## Web enumeration

Now we need to find somewhere to use this login information so we look for the website. Attempting to load the page at port 80 gives us a default apache page. But the 
rustscan found another http port earlier at 62337 and this brings up to Codiad 2.8.4 with a login page.

Trying out the very simple login guess of `john` with the password `password` logs us right in.

![image](https://user-images.githubusercontent.com/43668197/149427064-bc40fb0b-dfd4-49f4-b906-4f0f4573ea2a.png)

## Exploitation

Since the page nicely gives us the name and version `codiad 2.8.4` it's time to use `searchsploit` to see if there is any avaliable exploits for us. And there is! And it 
even requires authentication credentials which we happen to have! I will use `searchsploit -m 49907.py` to copy this file so I can read and understand what it is doing for
us and also to edit if necessary.

![image](https://user-images.githubusercontent.com/43668197/149427319-2bfacc84-f2bc-4311-924d-608c4f468901.png)

Create a new project on the site from the menu on the right and make sure its path is set like so.

![image](https://user-images.githubusercontent.com/43668197/149427838-a7574e39-3b57-4347-a328-a3a5e4618fe3.png)

Modify 49907.py to change data to codiad where underlined in the image below.

![image](https://user-images.githubusercontent.com/43668197/149428100-1763d6d1-88e7-4bfb-bbd6-5e47cffbd233.png)

And now run the exploit.

![image](https://user-images.githubusercontent.com/43668197/149428014-4f3e19ec-449a-43de-a396-54f0518b2045.png)

The printed path is a bit off here just ignore the `/data` and it should be like so:

![image](https://user-images.githubusercontent.com/43668197/149428311-bb587374-f05b-44b8-b75c-1be4a3d09788.png)

And we have access to `www-data` with shell.

![image](https://user-images.githubusercontent.com/43668197/149428391-77e15ec1-163d-44e0-8d71-219b4b25cfe7.png)

Heading over to the `/home` directory we see there is only one user and it's good ol `drac`. We don't have permissions to cat out the user.txt file yet but we can view 
drac's bash history.

![image](https://user-images.githubusercontent.com/43668197/149428889-924b20ac-3526-4801-a682-7aaeac1c798c.png)

We get mysql login so lets check if drac is a password reuser.

We cannot switch users like this so we must upgrade to a TTY shell first. I went to `revshells.com` and picked the `php` shell.

![image](https://user-images.githubusercontent.com/43668197/149431553-62c0c236-fbf6-4542-8957-7d4603014d46.png)

Then on your attack box you should have connection on netcat listener so upgrade it to TTY shell with the line below.

`python3 -c 'import pty;pty.spawn("/bin/bash")'`

Now switch users by using `su drac` and entering the password from bash history.

We can now `cat user.txt` for the first flag.

## Privilege escalation

The first thing to always check is `sudo -l`

![image](https://user-images.githubusercontent.com/43668197/149431880-3dbbdf72-ede7-420d-ae0a-47126e1f3fae.png)

![image](https://user-images.githubusercontent.com/43668197/149432115-9cb7a110-3771-49a5-baa9-0d30db6df5fa.png)

The location of the service is `/lib/systemd/system/vsftpd.service`. Make sure it has root privs.

![image](https://user-images.githubusercontent.com/43668197/149432246-c5345192-f4a0-461a-91eb-1e3b82e8ec62.png)

Good it does! Edit the file in your favorite editor and change ExecStart to a shell.

![image](https://user-images.githubusercontent.com/43668197/149537179-b909d277-dc75-4d12-9a95-0c56cb501cc1.png)

Run `systemctl daemon-restart` then `sudo /usr/sbin/service vsftpd restart` and you should have your root shell.

![image](https://user-images.githubusercontent.com/43668197/149537596-34e2d9a1-10c6-4cfb-a8a7-253913944e3f.png)

Go cat the `root.txt` file from the `root` directory and room cleared!
