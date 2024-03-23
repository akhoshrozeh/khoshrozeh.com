+++
title = 'TechTrove'
date = 2024-03-23T01:07:43-07:00
draft = false
tags = []
+++

https://akhoshrozeh.github.io/tech-blog-aggregator/

### Introduction
[TechTrove](https://akhoshrozeh.github.io/tech-blog-aggregator/) (placeholder name) is an aggregation site that provides references to useful engineering articles written by top tech companies. It allows for users to by topics and/or keywords (not implemented).

### Inspiration
The big O'Reilly books are great for general principles and developing your theoretical understanding but it always feel just a little detached. And it's hard to find real production level examples of solutions on Youtube. Which is my I like the engineering blogs of the top tech companies. So why not build a site which aggregates those articles?  

<!-- *Note:* Under construction.  -->
### Design
Two important characteristics of this project:
    - our data input isn't frequently updated. These companies usually publish an article or two a month. So we won't need to query our endpoints often. 
    - the data we're storing is relatively small, since we're collecting small data from a small number of sources.

With that in mind, it didn't really make since to setup a server, as I could just host a static site (for the time being).
I can just store the backend aggregation scripts and store the data on my local computer. I can then setup cron to run these every couple weeks or so. After the scripts run and data files are updated, I can just re-build and re-deploy using Github's CI/CD.

Keep it simple when you can!

### Getting the Data
Some of the engineering blogs, I'll have scrape the data using BeautifulSoup in Python. Others that may use another blogging platform, such as Medium, will have an API available so I won't have to scrape it myself.

Now, with the data I have (article titles, links, tags, etc.), I need to store it persistently. Honestly, I could probably get away with just storing it as a JSON file. But I'll probably setup a DB just to make the data more manageable in case I add more features in the future.

Check it [out](https://akhoshrozeh.github.io/tech-blog-aggregator/)! 

Note: I just started this project so there'll be a lot more improvements coming soon!


