+++
title = 'Recon Tactics and Tools with TomNomNom'
date = 2023-09-25T13:51:05+08:00
draft = false
tags = ["Reconnaissance", "Subdomain Enumeration", "Pentester Tools", "Asset Discovery", "Information Gathering"]
+++

### Introduction
In this post I'm going to go over some reconnaissance tools, most of them built by well known and respected hacker [TomNomNom](https://twitter.com/TomNomNom?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor), or Tom Hudson. I'm also going to cover some of his methodologies and techinques, for example how he utilizes VIM in his recon. 

You can find and install his tools from his [github](https://github.com/tomnomnom/). 

I'm going to demonstrate some of these tactics on Admiral Labs which currently has a bug bounty on HackerOne. The domains I'll be reconning are within the scope of the bug bounty program: `*.getadmiral.com` and `*.levenlabs.com`. But this post will mainly about Tom's tools and tactics, not a writeup for a bug bounty. 

These notes were taken from this [video](https://www.youtube.com/watch?v=SYExiynPEKM) which I encourage everyone to watch. This is also a more beginner friendly [video](https://www.youtube.com/watch?v=l8iXMgk2nnY), where he goes a bit slower and explains commands a bit more. 

Before we get started, let's start by making a recon directory for the files we download and will use and `cd` into it. Let's go. 

### Subdomain Enumeration
One of the first steps in recon is subdomain enumeration. There are many tools to do this but the one we'll discuss is `assetfinder`. TomNomNom also recommends using [findomain](https://github.com/Findomain/Findomain) in conjuction with `assetfinder`, however it is a paid service. 


### assetfinder
Built by TomNomNom, the idea of this program is fetch subdomains from passive resources rather than doing any bruteforcing on the target. Let's put our targets into a file `wildcards` then pipe those into `assetfinder`. 

We'll also use his tool `anew`, which "appends lines from stdin to a file, but only if they don't already appear in the file. It also outputs new lines to stdout too, making it a bit like a `tee -a` that removes duplicates.

Let's grab the subdomains. 

`cat wildcards | assetfinder --subs-only | anew domains`

One issue about assetfinder is that it will sometimes include unwanted characters at the front of the string like wildcards and underscores. We can get rid of those in a low-tech way in VIM. 

Open `domains` in VIM and open the command line with `:`. We can take the current file with `%` and run it through a shell with `!` so now it functions as piping the content from the current file (`domains`) into the process. Now we sort it so those special charaters appear together. This is what your vim command line should be: `:%! sort`. 

Now there's a few `*.` strings at the beginning which we can quickly delete using Visual Block mode in VIM. 

Great, now we have some domains to investigate further. 

### httprobe
`httprobe` takes a list of domains and probes them for running http and https servers. The `--prefer-https` option doesn't check for HTTP if HTTPS is working. There are a few more options for this command, including `-c` which controls the concurrency level. We'll run this and store the results in a file `hosts`.

`cat domains | httprobe --prefer-https | anew hosts`

We can see we now have 65 running hosts by using `wc`. Now let's take a look around at what these hosts have. 


### fff
`fff` stands for "Fairly Fast Fetcher" and it requests a bunch of URLs from stdin. The same thing can be accomplished with `meg`, another tool from Tom. The main idea of fff is to fetch new requests without waiting for the last one to finish. While this makes it much faster than waiting for each request to return (some can be very slow), it's possible we may run out of file descriptors due to a new socket being created. You can change your file descriptor limit using `ulimit`. Also, we can set the delay between issuing request using the `-d <time in ms>` option. 

In this example though, we only have 65 hosts so it shouldn't be a problem. `-S` saves the response and `-o <file>` specifies the file to save the responses.

`cat hosts | fff -S -o roots`

Inside the `roots` folder, we've saved the body and headers from the host's repsonse. These are saved in folders with name of the respective host.


Why do we collect the headers? HTTP reponse headers have a bunch of metadata that can sometimes be boring but sometimes can be interesting, like what server software is being run. 

**_We want to look for things are different or that standout, such as uncommon headers._**

This brings us to the tool `gf`.


### gf
`gf` is a wrapper for grep to avoid typing long patterns. Some of the patterns we can grep for are AWS keys, debug pages, S3 buckets and many more. You can view the available patterns using `gf -list`.

I've created a small shell script called `allgf` which runs all the options for `gf`:

    gf -list | while read -r option; do
     printf "Running: gf %s\n" "$option"
     gf "$option"
    done

Add this script to your path so it's easily accessible. You can also redirect this output into a file to save it. 

Again, we're looking for stuff that stands out. For example, if we find places/files/service configs that aren't consistent with the rest of their environment, maybe that process was built outside their standard SDLC, which means there may be some interesting unique bugs. 

Example:

Running `gf meg-headers | vim -` inside `roots` puts all the headers inside a VIM buffer. We can sort that and start to look for interesting headers, such as headers that may appear only once. 

If we find something interesting or unexpected, say a server is running PHP version 4, we can investigate that further.


But suppose we don't find anything interesting. What else can we do?


### waybackurls

`waybackurls` fetches known urls from the Wayback Machine for `*.domain`.

It also has a useful option, `--get-versions`, which shows multiple versions of a resource. This is important because as files are being pushed to, one version might have something removed but endpoint is still accessible and may be vulnerable. This can be a goldmine. 

So let's see if there's anything interesting the Wayback Machine has for our wildcards.

`cat wildcards | waybackurls | anew waybackurls`

Looking through the `waybackurls`, there's an interesting link: http://dev.images.getadmiral.com/. This looks like a dev page so let's check it out. It returns a 404 response printing "file not found" but the service is still running. Interesting; we can use `gobuster` to enumerate directories on this server via bruteforce. 

We find there's a directory called `/upload` but requires a signature. This would something worth investigating further (but I won't do that here).

_Note: `ffuf` is also a good tool for web fuzzing._

_Another Note: TomNomNom uses raft-large-files and raft-large-directories from [seclists](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content) as his wordlist for fuzzing._




### Burp
Obviously, this is not a tool Tom created, however here's one way how he uses it. Say we've found an interesting URL/resource we want to investigate further. We use Burp Proxy to see what other requests are made behind the scenes when we request the initial page in our browser, a.k.a. the HTTP history feature, and poke around there as well. 

### urinteresting
This [tool](https://github.com/tomnomnom/hacks/tree/master/urinteresting) accepts URLs on stdin and outputs the ones that look 'interesting'.

### inscope
Not built by Tom but still very helpful in the information gathering phase. After you've gathered thousands or maybe tens of thousands of URLs, it's likely some of those won't be in scope. [inscope](https://github.com/nil0x42/inscope) is a good tool to filter those unwanted or out-of-scope endpoints out.

### threatcrowd
[Threatcrowd](http://ci-www.threatcrowd.org/) is a great resource for finding endpoints that aren't subdomains of our target but may be related to it. `assetfinder` spits out stuff like that. It's hard to bruteforce for things related to a domain!


### Miscellaneous Points and Tips

- Historic data is very valuable. Why? Companies generally get better at building stuff overtime which means they weren't as good at building stuff in the past. If there's any old stuff hanging around, it's possibly more likely to have issues with it. Also knowing how it used to work will give you more information as well. 

- Whenever you see a redirect, it's worth checking to see if you can turn it into an open redirect.

- In terms of creating your own tooling, if there's process you do often and takes a decent amount of time, think about how you can build a tool to speed that up. 

- Filtering for uri's that have a query string are intersting because by definition they have an input. And finding out what effect that input has can be a good place to look for vulnerabilities.

### Conclusion

The main point Tom makes is to find things that stick out and then poke around them. And you can't figure out what stands out until you know what normal looks like, which can often be a lot of the early stages. 

For example, what's the static assest structure? Are they passing things around in query string parameters or POST data? How do responses change as you go through each level of the directory structure? His reconnaissance is a mix of both asset discovery and information gathering. 

Again, I highly encourage everyone interested in pentesting to watch the video end to end. It is a goldmine of useful tactics for reconnaissance. I personally learned a lot from it and will be incorporating them into my strategy. 




