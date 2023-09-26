+++
title = 'CTF Writeup: Break Out the Cage'
date = 2023-09-25T21:52:54+08:00
draft = false
tags = ["CTF", "TryHackMe", "Linux", "Steganography", "Privilege Escalation", "Ciphers"]
+++

### Introduction

This [CTF](https://tryhackme.com/room/breakoutthecage1) on TryHackMe has no instructions and only three questions:

1. What is Weston's password?
2. What's the user flag?
3. What's the root flag?

Let's spin up the virtual machine and get hacking.

### Break Out the Cage

The only clue we start with is there's a person named Weston. 

We'll start with a TCP SYN port scan to see services are running, as well as gathering the service and OS information.

`sudo nmap -sS -sV -O --min-parallelism 20 10.10.157.193`

We have 3 services running: ftp, ssh and http (all on their standard ports 21, 22 and 80).
The versions are (respectively):
- vsftpd 3.0.3
- OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
- Apache httpd 2.4.29 ((Ubuntu))  


The website has HTML list items that you would expect to redirect to another page, such as "About Me", "Contact", etc., but they all actually just reference the root directory. I found nothing really interesting by walking the site.

Let's start `gobuster` to start bruteforcing sub-directories on the http server:\
`gobuster dir -u http://10.10.224.92:80  -t 50 -w ~/wordlists/kali-wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt | anew dirs`

_Note: anew is similar to `tee -a`_

I found the following directories:

    /images
    /html
    /scripts
    /contracts
    /auditions

/images just shows all the images used in the homepage. 
/html is empty.
/scripts contains partial scripts for movies. I investigated the page's source for hidden comments but found nothing.
/contracts contains a folder /FolderNotInUse which is empty.
/auditions contains an file, "must_practice_corrupt_file.mp3", which is the only interesting thing I found. 

I downloaded the file with `wget` and upon listening, there's clearly a portion of the audio that's been tampered with. I grab any human readable characters out the file with the `strings` command. However, I didn't find anything interesting. I thought perhaps some information may be stored in there visually so I put the file into a spectogram using "Sonic Visualizer". There's a visible word written in what looks like MS Word Art, "namelesstwo". Ah, steganography.

Interesting. Maybe it's a password? It try to ssh into the server with Weston (I tried lowercase too) but that's not the password. Let's investigate the other services.

I tried to login via ssh with an anonymous account but that didn't work.
I also tried the same with ftp and it did work!

There's a file called `dad_tasks` so I download it to my local machine with `get dad_tasks` and open it:

    UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtbCBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlkaXp3bGtic2lkaXVzY3ds

Looks like it's encoded in base64 so let's decode it with the base64 command: 
`echo {THE_STRING} | base64 --decode`


and it outputs this:

    Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
    Sfw. Kajnmb xsi owuowge
    Faz. Tml fkfr qgseik ag oqeibx
    Eljwx. Xil bqi aiklbywqe
    Rsfv. Zwel vvm imel sumebt lqwdsfk
    Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.
    Iz glww A ykftef.... Qjhsvbouuoexcmvwkwwatfllxughhbbcmydizwlkbsidiuscwl

It looks like it's using a Caeser cipher so I try to bruteforce it with quipqiup but nothing intelligeble gets returned, so maybe it's not a Caesar cipher. We can use [boxentriq](boxentriq.com) to determine what kind of cipher we're looking at. It reports that it's actually a Vigenere Cipher. This cipher requires a key so I try the phrase from the audio file as the key. 

It worked! It outputs:

    Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
    One. Revamp the website
    Two. Put more quotes in script
    Three. Buy bee pesticide
    Four. Help him with acting lessons
    Five. Teach Dad what "information security" is.
    In case I forget.... Mydadisghostrideraintthatcoolnocausehesonfirejokes

From the note, it appears as if Weston wrote it. And it says "In case _I_ forget" which makes me think the following string is _his_ password. I try it as the password with ssh. 

It works! Great, we've now gained access to the server as `weston` and found the first flag. Now let's look around. 

I get a broadcast while ssh'ed:

    Broadcast message from cage@national-treasure (somewhere) (Tue Sep 26 06:00:01 
    What's in the bag? A shark or something? — The Wicker Man

Interesting, I'll come back to this. 

I want to gain root access so I first check if there's any programs with the setuid bit set and there's none. 
I then check if there's any programs weston can run as root using `sudo -l`. The program `/usr/bin/bees` can be run as root by me so let's see what that program is. 

It's shell script that has one line:
    wall "AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!"

wall is a program that writes a message to all others and is probably what's being used for those broadcast messages I'm getting. 'bees' is also a read-only file, otherwise we could change the script spawn a shell with root access. 

I think the broadcast message I got is probably set as a cronjob, which is a surface that can be vulnerable for privilege escalation. Let's see the cron jobs in `/etc/crontab`.

Nothing interesting there. I still don't know where the program that is that's sending these messages is, so I'll grep the system looking for a file that contains the 'wall' program:
`grep -r "wall " 2>> /dev/null | tee -a /var/tmp/grep_wall`

_Note: I'm using 'tee -a' now since 'anew' isn't installed on this system._

While grep is running, some more messages appear:

    Broadcast message from cage@national-treasure (somewhere) (Tue Sep 26 06:03:01 
    Well, I'm one of those fortunate people who like my job, sir. Got my first chemistry set when I was seven, blew my eyebrows off, we never saw the cat again, been into it ever since. — The Rock
                                                                            
    Broadcast message from cage@national-treasure (somewhere) (Tue Sep 26 06:06:01                                                                                
    That's funny, my name's Roger. Two Rogers don't make a right! — Gone in Sixty Seconds

Ok, so these messages come in set intervals of 3 minutes (note the time). Again, I want to find out from which script it's coming from.

The grep finishes so start looking through it. After searching through `grep_wall` , I find something interesting!

There's folder in /opt called `.dads_scripts` which contains `spread_the_quotes.py`. This is the python script:
    #!/usr/bin/env python

    #Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
    import os
    import random

    lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
    quote = random.choice(lines)
    os.system("wall " + quote)

The owner and group name is 'cage', but if we want to edit this we have to be logged in as cage; the group doesn't have write permissions. However, there's `.quotes` file inside which it reads from. And that file has write permissions set for the 'cage' group. By looking at `/etc/group`, we see that `weston` is part of the group! Great. We can edit the .quotes file to have code that spawns a reverse shell as the user 'cage'. While we're not gaining root privilege, we can notice that 'cage' has higher privileges than 'weston' and perhaps 'cage' has access to programs that will grant us root access. 

So we wait until the script is ran again and then the reverse shell spawns in another terminal we were listening on with netcat. We're now logged in as cage. After [stabilizing the shell](../stabilizing-shell), I see inside cage's home directory there's a file called `Super_Duper_Checklist` which contains the second flag!

Now we finally need to find the root flag. 

There's a directory in his home folder called `email_backup`. One of the emails hints that the string '`haiinspsyanileph`' has to do with the password for `root`, which is who is the person 'Sean'. The emails mention that Sean is obsessed with his **_face_**.

I try to login as root with that string but it's not the password. It looks like an encrypted string that uses a simple cipher. Back to boxentriq to analyze the text, which suggests it's a Columnar Transposition Cipher. So we need to look for the key. I try `face` but that doesn't work, as well as `namelesstwo` which also doesn't work. Maybe it's not using that cipher? Since we already used a Vigenere Cipher, I try that instead and use `face`. And it works! The password for root is `cageisnotalegend`.

After gaining root access, can find another `email_backups` folder inside `/root`, which contains the final flag. 

Room complete!



### Conclusion
I feel like this CTF was more about finding clues than hacking into a system. Nonetheless, I still enjoyed it as it required knowledge of various areas of security: steganography, cryptography, OS concepts, reverse shells and forensics.
