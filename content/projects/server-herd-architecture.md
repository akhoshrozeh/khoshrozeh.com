+++
title = 'Server Herd Architecture'
date = 2023-12-24T17:23:57-05:00
draft = false
tags = ["Python", "Google API"]
+++

### Context
This is a project I did from UCLA where I implemented a server herd archiecture in Python. The idea for architecture comes from the application server being a bottleneck. In the server herd architecture, multiple application servers communicate directly with each other and the core database, handling rapidly evolving data while the database manages more stable, less frequently accessed information. This project also explores using the asyncio library for communication.

### Description

I developed a distributed application server herd using Python 3.6.3 with asyncio as the primary framework. I was tasked with exploring the effectiveness of asyncio in enabling inter-server communication and managing mobile client requests for a hypothetical Wikimedia-style news service. I then implemented a system where five servers communicated among themselves and with mobile device emulators using TCP connections. I utilized asyncio to handle message passing between servers and manage client location updates, querying, and response formatting. HTTP requests were used for querying the Google Places API due to asyncio's limitations in supporting TCP and SSL protocols directly.

The objective was to assess the viability of asyncio for real-time communication and data propagation among distributed servers. I then evaluated the ease of writing asyncio-based applications, their scalability, and performance implications. Additionally, conducted comparative analysis addressing concerns over Python's type checking, memory management, and multithreading in contrast to Java. Generated a detailed report outlining findings, encountered challenges, and a recommendation regarding the suitability of asyncio for this application type.

The outcome provided insights into asyncio's capabilities for facilitating rapid communication among distributed servers, managing client requests, and handling asynchronous tasks efficiently within a Python-based architecture.

You can find the code and report below. 

[Report](../assets/server-herd-report.pdf) \
[Code](https://github.com/akhoshrozeh/server-herd-architecture/blob/main/server.py)