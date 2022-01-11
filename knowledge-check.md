# Hack The Box Academy - Getting Started Module (Knowledge Check Box)

This HTB box is from the Getting Started Module from HTB Academy and it is the last challenge in the module that has to be solved without a walkthrough.

Objective: 
1. Initial Foothold - `user.txt`
2. Privilege Escalation - `root.txt`

## Enumeration

We will first perform enumeration with `nmap` to identify the services running on the machine.

![image](https://user-images.githubusercontent.com/57955404/148878655-3a209b3d-5628-44c7-90cb-a01f0809ca60.png)

It is running an apache webserver, moreover nmap also detected a robots.txt and an admin directory.

Lets take a look at the website. Upon entering the main page, there is nothing interesting but we do know that it is running a CMS called "GetSimple". Inthe admin page, we are presented with a login form. 

![image](https://user-images.githubusercontent.com/57955404/148879260-4dc509c2-b3f6-4e90-8c10-9a04e9afd058.png)

We can use `gobuster` to search for hidden directories in hope to find useful information that may lead us to obtaining the login credentials.

![image](https://user-images.githubusercontent.com/57955404/148879466-1125edf2-c9c4-49c4-b032-3b30abe3f8c7.png)

`gobuster` found a few directories.

![image](https://user-images.githubusercontent.com/57955404/148879610-9387b88e-4562-4cad-9433-2b9135736a6c.png)

After going through the directories, there were two files with some interesting information. Lets take a look atthe content.

In this file, which is in the `data/cache/` directory, it says that something is still running the v3.3.15 version which is an outdated version. We can assume here that it is mentioning the GetSimple CMS.

![image](https://user-images.githubusercontent.com/57955404/148880398-b9bc4736-8c2b-4189-8854-0f50bc765f6d.png)

In the other file, which is located at `/data/users/admin.xml`, it gives a username `admin` and also a hashed password.

![image](https://user-images.githubusercontent.com/57955404/148880577-4b7e4ebc-80ed-4aac-ade0-1fa482732adb.png)


# Initial Foothold

Now that we have the version of the CMS, we can use `metasploit` to search for known exploits for that version.

![image](https://user-images.githubusercontent.com/57955404/148882199-510e1802-d60d-4cd0-9080-45841af34937.png)

There is indeed an exploit for it, which is a Remote Code Execution (RCE) exploit. To understand how the exploit work, can take a look [here](https://ssd-disclosure.com/ssd-advisory-getcms-unauthenticated-remote-code-execution/). Lets move on to `metasploit` and configure the settings.

![image](https://user-images.githubusercontent.com/57955404/148882347-2f94094f-c07d-48f7-9955-d08b49dd2522.png)

We need to set the target IP and attacker IP, which we will set to the `tun0` interface since we are connecting to the HTB VPN.

![image](https://user-images.githubusercontent.com/57955404/148882416-a0a986de-0f30-4484-b0df-062e35959f3e.png)

Then, run the `check` command to see if everything is ready to go and start the exploit.

![image](https://user-images.githubusercontent.com/57955404/148882958-914128c8-1cd3-493d-95f9-89e46c1bd405.png)

The exploit works and we initiated a reverse shell to the target machine.

We can enter the `shell` command to enter the shell of the target machine and test to see if it works.

![image](https://user-images.githubusercontent.com/57955404/148883117-bb2207bb-1b88-468c-bc29-1e309f61c00f.png)

Now that we are in, we can spawn an actual shell with the following command:
> `python3 -c 'import pty; pty.spawn("/bin/bash")'`

Now that we got an actual shell, we can navigate to the `home` directory to see if there is any users.

![image](https://user-images.githubusercontent.com/57955404/148883668-51e7f806-b36c-4a66-9f69-43f6960e73f7.png)

There is a user called `mrb3n`, enter the directory and the user flag is there.

# Privilege Escalation 

Now that we have the initial foothold, lets escalate our privilege to root access.

We can use `sudo -l` command to see what privileges that the user has.

![image](https://user-images.githubusercontent.com/57955404/148884116-98167389-0d50-4eb5-909b-e09ef77fa7ac.png)

It seems that we can run sudo on php. Based on [here](https://gtfobins.github.io) is a privilege escalation cheatsheet. We can use a php one-liner payload to try gaining root access. We know that we can run sudo with php, and the following [command](https://gtfobins.github.io/gtfobins/php/#sudo) is perfect for our use case.

> `sudo php -r "system('/bin/sh');"`

![image](https://user-images.githubusercontent.com/57955404/148884521-c4a2fcd8-7400-420d-b877-8dbe2438a690.png)

The payload works perfectly and we now have root access.

Enter the root directory to obtain the root flag is there.








