+++
title = 'Nginx and Pm2'
date = 2024-10-01T22:52:03-07:00
draft = false
+++

### Background

For [DevReadz](https://devreadz.com), I decided to move off Vercel and switch hosting the app a Hetzner VPS. Vercel has a nice convenience to it but it also feels too abstracted almost. I didn't want to constantly be looking at my billing usage and also I like working on a Linux machine. Anyways, the point of this is to say after I got my app all setup and online, I started to look the CPU usage on Hetzner's dashboard while I played with the site out of curiosity and noticed the CPU was hitting almost 20% usage. That seemed too high. 

### VPS
I'm using Hetzner's cheapest shared VPS which has 2 Virtual CPUS, 2 GB of RAM, and 40 GB of local storage. Even though it's a modest server, I felt that it should handle more. 

### 1st Fix: Stupid Mistake
I realized I left a conosle.log on my server code which was writing too much stdout and was meant for development purposes. Taking that out reduced usage by about 5%. I know, this is not rigourous or scientific "testing" but I was still just looking around. 


### Utilizing both cores
Next.js uses Node.js, which is single-threaded. Why don't I just spin up 2 instances using pm2 and use ngnix as a load balancer as well (I was already using it as a reverse proxy)? That was pretty easy to do and ended reducing CPU usage by about half. 


### Jmeter
I acutally did some load testing using Jmeter but I don't think I saved it. I'm to recreate it by turning off one server then turning both back on. 



