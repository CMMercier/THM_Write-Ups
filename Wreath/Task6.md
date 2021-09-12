# Information

Name: Wreath

Difficulty: Easy

Description: Learn how to pivot through a network by compromising a public facing web machine and tunnelling your traffic to access other machines in Wreath's network.

## Webserver Exploitation

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

