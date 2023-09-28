+++
title = 'CTF Writeup: Lian Yu'
date = 2023-09-27T17:48:51+08:00
draft = false
tags = ["CTF", "TryHackMe", "Steganography", "Privilege Escalation"]
+++

### Introduction

This CTF looks pretty straightforward based on the questions. I think I'll try and do a speed run on it and do the writeup at the same time.

These are the questions:
1. What is the Web Directory you found?
2. what is the file name you found?
3. what is the FTP Password?
4. what is the file name with SSH password?
5. user.txt
6. root.txt
 
Let's go!

### CTF

Directory enumeration:
`gobuster dir -u http://10.10.78.66/:80  -t 50 -w ~/wordlists/kali-wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt | anew dirs `

nmap:
`sudo nmap -sS -sV -O -oN nmap_output --min-parallelism 25 10.10.78.66`

nmap output:
    PORT    STATE SERVICE VERSION
    21/tcp  open  ftp     vsftpd 3.0.2
    22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
    80/tcp  open  http    Apache httpd
    111/tcp open  rpcbind 2-4 (RPC #100000)


We find the directory "island". It's an HTML page but there's a comment in the page source, "!-- go!go!go! -->" and also a code word "vigilante" written in white font. It has "Lian_Yu" in bold.

I try to login via ftp and "vigilante" works as a username but requires a password. I'll break out `hydra` and try to bruteforce it while looking for more clues. 

I can't find anything else, so I go back to /island page where I re-read where it says "You should find a way to 'Lian_Yu'". "Finding a way to" might refer to a path, so I'll use gobuster again to enumerate directories within /island.

We've found another directory, /island/2100. There's an embedded youtube video but it's not available anymore, a comment in the page source, "you can avail your .ticket here but how?" and an h1 header, "How Oliver Queen finds his way to Lian_Yu?".

Nothing obvious here. I tried looking up how Oliver found his way to Lian Yu in the series and apparently he was shipwrecked. I tried variations of that as the ftp password but nothing worked. I'll enumerate directories again within /island/2100. 

I wasn't able to find anything so I went back the 2100 page. The ".ticket" reference makes me think maybe there's a file that ends in .ticket. So I back to gobuster and start fuzzing for files ending in .ticket:

`gobuster fuzz -u http://10.10.78.66/island/2100/FUZZ.ticket  -t 50 -w ~/wordlists/kali-wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt >> 2100_sub`


Ok, we found a file: green_arrow.ticket. This is what the page contains:

    This is just a token to get into Queen's Gambit(Ship)


    RTy8yhBQdscX

Looks like an encoding, probably base64. When I decode it from base64, I get a garbage string. Maybe it's not base64. It turns out it's actually base 58 and gets decoded to "!#th3h00d".

It works as the password over ftp but not for ssh. It looks like there's a few folders and levels in the ftp server, so I'm just going to download it all and look at it on my local machine using wget. 

`wget -r --ftp-user=vigilante --ftp-password=\!\#th3h00d ftp://10.10.78.66`

There's a few different pictures inside vigilante's directory, one of them with name "Leave_me_alone.png" which can't be opened. Turns out the magic header is wrong. So we can just edit the first 4 bytes of the .png header with `hexedit` and the file can now be opened. 

Now let's see what information is here. The image suggests there's password in here and I'm guessing it's using steganography.

First, I check with the `strings` command to see if there's anything obvious but I don't find anything. 
So, I'll use `steghide`, a common steganography CLI tool. `steghide` can't give any information on "Leave me alone" nor "Queens Gambit", but it can for aa.jpg. The password is just "password" which was hinted to into in the picture "Leave me alone". Inside aa.jpg is an encrypted file "ss.zip".

We got the info with `steghide info [file]` and extracted the .zip with `steghide extract -sf [file]`.

After unzipping, there's a "passwd.txt" file and "shado" file. 


In shado, there's the word "M3tahuman" which seems like a password. Possibly for ssh? It doesn't work for the user vigilante nor Lian_Yu. But then I remember there was another user, slade, in the /home directory of the target machine, which I saw when used ftp.

Sweet! Now I'm in the machine via ssh. There we find the first flag, user.txt:
    THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
			--Felicity Smoak


Felicity is a character from the arrowverse, so I'm not sure if that's going to be needed or it's just part of the theme of this CTF. Anyways, we need to get root access to get the root.txt flag. 


First I check what 'slade' can run as root with `sudo -l` and it shows I can 'pkexec'. Perfect. Using GTFObins, I see I can open a shell with `sudo pkexec /bin/bash`. 

We've opened a shell and there is root.txt!

Room complete. 

### Conclusion