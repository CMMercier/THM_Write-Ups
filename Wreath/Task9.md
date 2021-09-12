# Information

Name: Wreath

Difficulty: Easy

Description: Learn how to pivot through a network by compromising a public facing web machine and tunnelling your traffic to access other machines in Wreath's network.


## Pivoting Enumeration

**What is the absolute path to the file containing DNS entries on Linux?**

/etc/resolv.conf

**What is the absolute path to the hosts file on Windows?**

C:\Windows\System32\drivers\etc\hosts

**How could you see which IP addresses are active and allow ICMP echo requests on the 172.16.0.x/24 network using Bash?**

for i in {1..255}; do (ping -c 1 172.16.0.${i} | grep "bytes from" &); done
