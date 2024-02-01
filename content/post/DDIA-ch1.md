+++
title = 'Reliable, Scalable, and Maintainable Applications'
date = 2023-12-29T17:05:43-05:00
draft = false
+++

_Note: The following information comes Martin Kleppmann's "Designing Data-Intensive Applications". The following post is a recap/discussion of ideas from Chapter 1._

### Introduction
In this post, we're going to look at what we mean by the terms reliable, scalable and maintainable. We're also going to examine how we can try to achieve these goals for our system. 

- Apps are data intensive, not computationally expensive
- problems: amount, complexity, and rate of change of data
- all the different data building blocks have many different implementations and  characteristics for different cases
    - DBs, caches, search indexes, stream processing, batch processing, message queues, etc.
  - choosing the right one can have a huge affect on performance
  - *** give example

- why the term data systems? 
  - the boundaries between these data systems are becoming increasingly blurred
  - many systems require many of these tools for data processing/storage


### Reliability
- continuing to work correctly even when things go wrong, aka a fault occurs
- we want fault tolerant or resilient systems
A fault is usually defined as one com‐ ponent of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures.

Although we generally prefer tolerating faults over preventing faults, there are cases where prevention is better than cure
    - preventing a hacker from compromising the system

#### Hardware faults
Redundancy:
    - for HD, RAM, CPUs, Power, etc. 
    -  This approach cannot completely prevent hardware problems from causing failures, but it is well understood and can often keep a machine running uninterrupted for years.
- This was fairly sufficient in the past because you could quickly repair the faulty hardware with a short downtime
- however, as data volumes and applications computing demands increased, more apps started needing more machines which means more HW faults. 
- There is now a move towards systems that can tolerate the loss of an entire machine by using SW fault-tolerance techniques in addition to redundancy. 
  - this also has an operational advantage: can upgrade nodes one at a time with the system still running rather than scheduling a planned downtime (i.e. rolling upgrades)


#### Software Faults
These are more difficult to predict and often can cause other systems connected/related systems to fail, while HW faults are generally somewhat independent events. 
Examples:
    - OS Kernel bug (Linux leap second in 2012)
    - runaway process (memory leaks)
    - a critical surface that other services relies on fails
    - cascading failures

#### Human Errors
How to prevent?
- Design systems in a way that minimizes opportunities for error. For example, well-designed abstractions, APIs, and admin interfaces make it easy to do “the right thing” and discourage “the wrong thing.”
- Decouple the places where people make the most mistakes from the places where they can cause failures. 
- Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests [3
- Allow quick and easy recovery from human errors, to minimize the impact in the case of a failure. 
- Set up detailed and clear monitoring, such as performance metrics and error rates i.e. telemetry.
- Implement good management practices and training


### Scalability
Scalability is the term we use to describe a system’s ability to cope with increased load. 
it is meaningless to say “X is scalable” or “Y doesn’t scale.”
good examples of discussing scalabilty:
    - “If the system grows in a particular way, what are our options for coping with the growth?” \
    - “How can we add computing resources to handle the additional load?”


#### Describing Load
Load can be described with a few numbers which we call "load parameters". They could be:
- req/s to a webserver
-  the ratio of reads to writes in a database -
-  the number of simultaneously active users in a chat room -
-   the hit rate on a cache
or something else, depending on your application and architecture. 

Essentially, what components/inputs/pipelines/variables/etc. are going to affect the performance of your system?

#### Describing Performance
After describing the load, you can analyze how the performance changes when the load increases. 
2 ways to look at it:
- When you increase a load parameter and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?
- When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?

--> Throughput vs. Response Time
Response time is actually a distribution, not a single number. It's also measured in avg. response time but that doesn't tell you how many users actually experienced that delay. \
Use percentiles: Sort responses and use median to tell you what the typical user experiences. \

_Note: Latency (duration req. is waiting to be handled) and Response Time (client UX) are not exactly the same._

High percentiles of response times, also known as _tail latencies_, are important because they directly affect users’ experience of the service.
    - EX: Amazon has also observed that a 100 ms increase in response time reduces sales by 1%  
Optimizing the very high percentiles can be deemed not cost effective. But the high percentiles in general are still very important to optimize. They may be the users with the most data, i.e. the most valuable customers (like for Amazon).

High percentiles become especially important in backend services that are called multiple times as part of serving a single end-user request.
- response_time = max(req1, req2, ..., reqN), therefore the more requests made the higher the chance it will be slower due one slow request
- Known as tail latency amplification
- Measure with histograms

#### Approaches for Coping with Load
It is likely you will need to rethink your architecture for every order of magnitude increase in load - possibly more often than that. 

Horizontal vs. Vertical Scaling

- Scaling up is simpler but more expensive. 
- In reality, most good archictures leverage both powerful machines and cheaper virtual machines. 
- An elastic system (automatically increases resources in response to increase in load) can be useful if load is highly unpredictable, but manually scaled sys‐ tems are simpler and may have fewer operational surprises

- Making a stateful data system distributed from a single node can introduce a lot of complexity, compared to a stateless system. 


- *There is no such thing as a generic, one-size-fits-all scalable architecture*
  - this is due to load parameters and characterics of the system (reads, writes, complexity of data, access patterns, etc.)
  
- That being said, a lot of archictectures use the same building blocks and patterns. 


### Maintainability 
The majority of the cost of software isn't the initial development but the maintainence (bugs, upgrades, new features, migration, etc). 

We want to design software that minimizes the maintainence overhead in the future. There's 3 design principles to follow:

- Operability
    - Make it easy for operations teams to keep the system running smoothly.
- Simplicity
    - Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (Note this is not the same as simplicity of the user interface.)
- Evolvability
    - Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change. Also known as extensibil‐ ity, modifiability, or plasticity.
  
#### Operability: Making Life Easy for Operations
Operations teams are vital to keeping a software system running smoothly.
Data systems can do various things to make routine operations tasks easy, including:
- Providing visibility into the runtime behavior and internals of the system, with good monitoring
- Providing good support for automation and integration with standard tools
- Avoiding dependency on individual machines (allowing machines to be taken down for maintenance while the system as a whole continues running uninter‐ rupted)
- Providing good documentation and an easy-to-understand operational model (“If I do X, Y will happen”)
- Providing good default behavior, but also giving administrators the freedom to override defaults when needed
- Self-healing where appropriate, but also giving administrators manual control over the system state when needed
- Exhibiting predictable behavior, minimizing surprises


#### Simplicity: Managing Complexity
Possible symptoms of complexity:
- explosion of the state space
- tight coupling of modules 
- tangled dependencies 
- inconsistent naming and terminology 
- hacks aimed at solving performance problems 
- special-casing to work around issues elsewhere 
- and many more.

When complexity makes maintenance hard, budgets and schedules are often over‐ run. In complex software, there is also a greater risk of introducing bugs when mak‐ ing a change: when the system is harder for developers to understand and reason about, hidden assumptions, unintended consequences, and unexpected interactions are more easily overlooked.

- Reducing complexity greatly improves the maintainability of software, and thus simplicity should be a key goal for the systems we build.
- We want to reduce accidental complexity, meaning complexity of the implementation and not inhenrently in the problem it's solving. 
  - Use abstration! Think about using Python to print something vs. using syscall vs. interacting with CPU registers. 


#### Evolvability: Making Change Easy
There's almost a constant flux in that requirements and needs of your system. 
- In terms of organizational processes, Agile working patterns provide a framework for adapting to change. 
- The ease with which you can modify a data system, and adapt it to changing require‐ ments, is closely linked to its simplicity and its abstractions: simple and easy-to- understand systems are usually easier to modify than complex ones


### Summary 

- *Reliability* means making systems work correctly, even when faults occur. Faults can be in hardware (typically random and uncorrelated), software (bugs are typically sys‐ tematic and hard to deal with), and humans (who inevitably make mistakes from time to time). Fault-tolerance techniques can hide certain types of faults from the end user.
- *Scalability* means having strategies for keeping performance good, even when load increases. In order to discuss scalability, we first need ways of describing load and performance quantitatively. We briefly looked at Twitter’s home timelines as an example of describing load, and response time percentiles as a way of measuring performance. In a scalable system, you can add processing capacity in order to remain reliable under high load.
- *Maintainability* has many facets, but in essence it’s about making life better for the engineering and operations teams who need to work with the system. Good abstrac‐ tions can help reduce complexity and make the system easier to modify and adapt for new use cases. Good operability means having good visibility into the system’s health, and having effective ways of managing it.