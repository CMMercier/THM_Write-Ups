# Information

Name: Anthem

Difficulty: Easy

Description: Exploit a Windows machine in this beginner level challenge.

## Website Analysis

**Let's run nmap and check what ports are open.**

![image](https://user-images.githubusercontent.com/43668197/133477105-7099dddb-62da-4c9c-893b-487550204056.png)

**What port is for the web server?**

80

**What port is for remote desktop service?**

3389

**What is a possible password in one of the pages web crawlers check for?**

UmbracoIsTheBest!

First thing we check is to see if the site contains the robots.txt file.

![image](https://user-images.githubusercontent.com/43668197/133477701-138652fd-62de-4799-b1de-2056acd9090c.png)

It does and it even gives us a possible passward and some directories to check out.

**What CMS is the website using?**

Umbraco

**What is the domain of the website?**

anthem.com

![image](https://user-images.githubusercontent.com/43668197/133479844-21b40bb1-5e83-4f37-b8df-1a996cf70503.png)

**What's the name of the Administrator**

Solomon Grundy

![image](https://user-images.githubusercontent.com/43668197/133487560-fa37f0f8-1ac6-4c06-b2b6-632cdc58969c.png)


**Can we find find the email address of the administrator?**

sg@anthem.com

The blog post titled "we are hiring" gives us the email JD@anthem.com which we use to guess the admins email.

## Spot the flags

**What is flag 1?**

THM{L0L_WH0_US3S_M3T4}

The source code of the "We are hiring" blog post.

![image](https://user-images.githubusercontent.com/43668197/133489335-f670e60f-9c23-4cd8-901f-71566f3a6474.png)

**What is flag 2?**

THM{G!T_G00D}

The source code of both the blog posts.

![image](https://user-images.githubusercontent.com/43668197/133489155-79fca85a-849b-4bfc-956a-acbe8bfb9994.png)

**What is flag 3?**

THM{L0L_WH0_D15}

Located /authors from a gobuster search. 

![image](https://user-images.githubusercontent.com/43668197/133488295-b30ca753-2a84-47a5-b497-b25ce69f46b8.png)

**What is flag 4?**

THM{AN0TH3R_M3TA}

The source code of the "A cheers to our IT department" blog post.

![image](https://user-images.githubusercontent.com/43668197/133488964-995bd235-c32a-458f-bc99-f1f87064dbd4.png)

##  Final stage

Using remmina to access RDP I try the credentials we know of sg:UmbracoIsTheBest! and successfully log in.

**Gain initial access to the machine, what is the contents of user.txt?**

THM{N00T_NO0T}

Located right on the Desktop when we log in.

**Can we spot the admin password?**

ChangeMeBaby1MoreTime

First I changed the settings to be able to see any hidden files or folders.

![image](https://user-images.githubusercontent.com/43668197/133491905-14eb5b81-9df9-411e-848f-b243877823a4.png)

I found a hidden file named backup on C: with a single file named restore.

![image](https://user-images.githubusercontent.com/43668197/133492076-b72e467a-cb25-4b9c-b36f-79afcc903f90.png)

![image](https://user-images.githubusercontent.com/43668197/133492175-d8b08a46-e2fa-4166-835d-120e048c0b98.png)

I was able to simply give the sg user the required permissions.

![image](https://user-images.githubusercontent.com/43668197/133492476-8149b5d2-ebfc-4ec0-8c51-eeb8a51ecbda.png)

and we have the admin password.

![image](https://user-images.githubusercontent.com/43668197/133492506-c0eee922-8a24-444f-969f-a251a0e879fd.png)

**Escalate your privileges to root, what is the contents of root.txt?**

THM{Y0U_4R3_1337}

Login with remmina with the credentials administrator:ChangeMeBaby1MoreTime and the root.txt is waiting on the Desktop.
