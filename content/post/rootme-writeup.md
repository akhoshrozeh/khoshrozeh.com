+++
title = 'CTF Writeup: RootMe'
date = 2023-09-21T14:04:57+08:00
draft = false
tags = ["CTF", "TryHackMe", "Nmap", "gobuster", "Privilege Escalation", "SUID"]
+++

### Introduction
Here, I'm going to show how I solved the [RootMe](https://tryhackme.com/room/rrootme) CTF on TryHackMe. It's supposed to be for beginners so let's jump in and find out!


### Reconnaissance
We’ll start with nmap to run a TCP SYN scan with some parallelism and collect service version information as well:

`sudo nmap -sS -sV --min-parallelism 20 10.10.48.220`

Note: we need to run SYN scans with escalated privileges because it is a more stealthy scan than say a TCP Connect scan.

We see two ports are open, 22 and 80, which are for SSH and HTTP. (OpenSSH and Apache, in this case).
![](../post/rootme-pics/nmap%20scan.png)


With this we can answer the first 3 questions.

Question 1:
**_Scan the machine, how many ports are open?_**\
    2

Question 2:
**_What version of Apache is running?_**\
    2.4.29

Question 3:
**_What service is running on port 22?_**\
    ssh

Question 4:
**_Find directories on the web server._**

Let’s break out gobuster and see if there’s any hidden directories we might find interesting. And why not use some multithreading to speed it up. We’ll use a directory list from the Kali wordlists.\
`gobuster dir -u http://10.10.48.220:80  -t 50 -w ~/kali-wordlists/dirbuster/directory-list-1.0.txt`


Question 5:
**_What is the hidden directory?_**\
After we run gobuster for a few minutes, it finds a few directories: `/panel`, `/uploads`, `/js` and `/css`.



### Getting a shell

**_Find a form to upload and get a reverse shell, and find the flag._** 

First I checkout `/panels` see it’s a file upload form. Perfect! We can probably exploit this if there’s no advanced filter on what types of files can be uploaded. I then look at `/uploads`, which appears to list the files that we upload. Great, I have the idea. To get a reverse shell, first upload a php script to the target, then make that script run by clicking on the file in the `/uploads` directory. So now let’s try and get a reverse shell.

First I'll create the netcat listener.\
`nc -vnl 4242`

Let’s create a php file to create a reverse shell. I download a php reverse shell script [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), edit IP address so it connects to my machine and upload it via the `/panels` page. 

When we try to upload the php file, we get an error in Portugese saying .php files are not permitted.

I remember encountering a similar error on the Vulnversity room on TryHackMe. We were able to find which files were accepted using Burp Intruder. The one that worked was .phtml, so let’s just try that first and see if it works before we break out Burp. 

After changing the file extension to .phtml, success! Great, now let’s go to /uploads and click on our .phtml file.

Sweet, we’ve opened a shell into our target machine. 
![](../post/rootme-pics/reverse%20shell.png)


Before we go further, let's stablize our shell first since netcat isn't inherently stable. Here's how we can do it:
1. `python -c 'import pty;pty.spawn("/bin/bash")'`, which uses Python to spawn a better featured bash shell.
2. `export TERM=xterm` -- this will give us access to term commands such as `clear`.
3. Finally (and most importantly) we will background the shell using Ctrl + Z. Back in our own terminal we use `stty raw -echo; fg`. This does two things: first, it turns off our own terminal echo (which gives us access to tab autocompletes, the arrow keys, and Ctrl + C to kill processes). It then foregrounds the shell, thus completing the process.

Okay, resuming with the CTF... We’re looking for a flag which probably has a .txt file ending. Before running `find` on the entire system which might take a while, let’s start with specific directories. First `/home`. Nothing there. 
Let’s try `/var`. Boom. We found `/var/www/user.txt`, which contains the flag: **THM{y0u_g0t_a_sh3ll}**


### Privilege Escalation

Let’s start with a bit of enumeration to get some basic information on this system.

`/proc/version`:\
Linux version 4.15.0-112-generic (buildd@lcy01-amd64-027) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020

`Uname -a`:\
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

`env`:\
APACHE_RUN_DIR=/var/run/apache2
APACHE_PID_FILE=/var/run/apache2/apache2.pid
JOURNAL_STREAM=9:19643
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
INVOCATION_ID=35d9a12342194225a79b40345767f29d
APACHE_LOCK_DIR=/var/lock/apache2
LANG=C
APACHE_RUN_USER=www-data
APACHE_RUN_GROUP=www-data
APACHE_LOG_DIR=/var/log/apache2
PWD=/

I see there's an exploit on [exploit-db](https://www.exploit-db.com/exploits/47167) that escalates privilege via the [polkit technique](https://github.blog/2021-06-10-privilege-escalation-polkit-root-on-linux-with-bug/).

However looking at the prompts for the CTF, it wants us to exploit files with the SUID bit set. So let’s see what programs have the SUID bit set. This is often an easy way to escalate privilege.

I run `find / -perm  -u=s -type f 2>/dev/null` which returns many files but one in particular stands out: `/usr/bin/python`. Its owner is root, so any python commands we run will run as if root is running it. Easy!

Let’s go to [GTFObins](https://gtfobins.github.io/) click on the python binary and get the command for the SUID section. 
`python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

Paste that into our shell and boom. We have root now and can confirm it by running `whoami` in the shell.

Now we need to find the root flag. I saw the /root folder earlier, so that’s probably where the flag is. No need for `find`, yet.

And there it is: root.txt. The flag is **THM{pr1v1l3g3_3sc4l4t10n}**. And the room is complete!

### Conclusion
Takeaways: 
- Ensure proper access controls on web pages! Hidden directories can be easy to find; don't rely on [security by obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity).
- Ensure thorough filtering for file uploads! You can refer to the [OWASP cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) for different techniques to prevent attacks and fix vulnerabilities.
- Be careful when setting the SUID bit. Always think about how a malicious user could exploit this. Make sure you know what a binary/file's capabilities are before giving users privileged access when running it. GTFObins is a great resource for this. 


This was a very straightforward CTF and took me a little less than an hour to solve. The biggest issue I had was my slow and unreliable network was caused a lot of delay for me. Although the room was pretty simple it was still enjoyable and also I think was a good warmup for getting back into my pentesting tools. I’m definitely looking forward to more challenging CTFs!















