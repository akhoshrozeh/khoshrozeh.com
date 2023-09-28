+++
title = 'Network Device Hardening'
date = 2023-09-28T18:11:55+08:00
draft = false
tags = ["Network Hardening", "Securing Network Devices"]
+++

### Introduction

In this post, I'm going to go over some common threats and hardening techniques for various network devices including VPNs, routers, switches and more. Since organizations' networks are almost always a critical aspect of their security, it is important that the proper steps are taken to secure it. While no system can ever be 100% secure, we still can greatly reduce the organization's risk by preventing, detecting and responding appropriately and promptly to attacks. Network device hardening helps us achieve that. 


### What is a Network Device?
You might think that your phone is a network device, and while it is a device that does connect to a network(s), we actually categorize it as an **endpoint device** in network terminology. Let's separate the two terms by describing some of their characteristics. 

A **network device** functions to manage and control network resources. They forward, route and control network traffic, and often require complex configurations to properly setup. Some examples are routers, switches, firewalls, load balancers and gateways. They sit inside the core of the network. 

An **endpoint device** functions to access and consume network resources. They are the sources and destinations of network traffic (there are exceptions, such as configuring a router which would then be the destination). Generally, they have simpler configurations. Think about what you had to configure on your phone to connect to the Internet; just the WiFi password. Some examples are smartphones, PCs, tablets, smart watches, your smart fridge, IoT devices and printers. Website servers also considered endpoints. 


### Common Threats Against Network Devices
Here our briefly go over some of the threats network devices face and their attack vectors. It is by no means an exhaustive list.  

<u>Unauthorized access</u>: password attacks, exploiting known vulernabilities, social engineering/phising\
<u>Denial of Service (DOS)</u>: flooding with fake requests, manipulating packets\
<u>Man-in-the-Middle Attacks (MitM)</u>: ARP spoofing, DNS spoofing, rogue access point\
<u>Privilege Escalation</u>: password attacks, misconfigurations, exploiting known vulnerabilities\
<u>Bandwidth theft/hotlinking</u>: scraping large amounts of data, DoS attacks, malware attacks\


### Common Hardening Techniques
There are a few general techniques used to decrease the attack surface of a network device. We should note these techniques apply to more than just network devices. 

**_Updating and patching_** all software including firmware, operating systems, applications, etc. Updates often fix discovered vulnerabilities, so using outdated firmware/software increases your risk.

**_Disabling unnecessary ports and servces_**. If the port/service is not needed, then leaving it open just gives an adversary another possible avenue attack.

**_Principle of Least Privilege_**. The underlying idea is similar to the above technique; exposing only the necessary. Only give users the bare minimum level of access they need for their functions and nothing more.

**_Log Monitoring_**. Enabling logging on your systems means you can quickly detect suspicious behavior and helps you recover from an incident (e.g. analyzing the steps an attacker made to break into your system), thereby reducing the overall risk.

**_Enforcing Strong Passwords_**. With fast hardware being cheap and wordlists easily accessibile, weak passwords can easily be cracked. Ensuring users use long _and_ complex passwords, we can reduce the chance of a bruteforcing type of attack. We can go further by checking if a password is found in a known wordlist before setting it.

**_Multi-Factor Authentication_**. Authentication can be categorized into 3 factors: something you know (a password), something you have (your debit card when you buy something), or something you are (your fingerprint). Using two of these in the authentication process makes it more difficult for an adversary to break. 


### Secure Protocols
Using secure protocols is critical in hardening your network devices, ensuring security principles such as the confidentiality, integrity and authentication, like how SSL/TLS does. Proper usage of cryptography is at the core of these. 

_Note: I will have another post about different secure protocols._

It is also equally important to disable insecure protocols. If you leave available to use, it's possible that someone may use them and expose condifidential information. It is possible to use inherently secure protocols like LDAP but misconfigure them which actually now makes them insecure. This concept goes back to the idea of disabling all unnecessary services. 


### Hardening VPNs
VPNs are very common nowadays thanks to remote work and increased awareness of computer privacy. This also means they've become a larger target for attackers and therefore the proper security controls need to be in place to prevent attacks and keep the data of businesses and users safe.

I'm going to show an example standard hardening practices for a VPN. In this example it'll be one provided by OpenVPN. 

#### Example
Usually, all the VPN servers consist of server-side and client-side configurations through a standard config file. In this example, they are `client.conf` and `server.conf`.

I'm going to cover some the significant hardening best-practices for a VPN.

1. Use a strong encryption algorithm.
In our `server.conf`, there's a line that controls the cipher used to protect in transit data: `cipher AES-256-CBC`.
This means we're using AES with a key size of 256 bits in cipher block chaining mode. This is considered one of the strongest symmetric encryptions. If you're using a weak encryption cipher, such as DES, you'll want to change it. 

2. Ensure up-to-date VPN gateway software.
A gateway is simply the entry point all traffic must go through if a device is communicating outside it's LAN. On Linux, we can upgrade it with `sudo apt upgrade openvpn but it'll different depending on your OS and VPN. VPN's usually have a GUI where you can check for and install updates. 

3. Implement strong authentication
We can use the `auth` directive to ensure that a secure hashing algorithm was used for packet authentication. For example, add this line to the `server.conf` file: `auth SHA256`.

4. Change Default Settings
Changing default usernames and passwords is a must and greatly reduces the risk of unauthorized access. This is also not something unique to VPNs and should be done with all services and devices you use.

5. Enable Perfect Forward Secrecy (PFS)
Perfect Forward Secrecy (PFS) in OpenVPN generates unique session keys for each session to strengthen the security of the VPN connection. Because of this, even if an adversary successfully obtained a session key, they could not use it to decode more sessions. 

    We can use the tls-crypt directive in the OpenVPN configuration file to enable PFS. The `tls-crypt` directive requires a key that can be generated using the command `sudo openvpn --genkey --secret my.key` and should be placed in the same directory on the server.


6. Dedicated Users for VPN Server
Limit user access by creating a dedicated user account and group with restricted permissions specifically for running the OpenVPN server.



There are many other options for configuration of your VPN but depends on your organization's needs. 


### Hardening Routers, Switches and Firewalls
Routers and switches must be hardened for the network infrastructure to be secure and reliable. Every network needs routers and switches, often the first line of defence against potential security risks and attacks. By hardedning these devices, we can prevent unauthorised access, avoid data breaches, and ensure network service availability, as well as improve performance and increase resilience.

1. Proper Setup Details\
While setting up any network device, it is necessary to fill in all relevant details like hostname, timezone, logging, and more. These features assist in conducting incident handling in case of a compromise. For example, logging must be enabled to log all the events with the default alert level Debug. Similarly, timezone and time synchronization must be set accurately to properly correlate events with their occurrence time. 

2. Changing Default Credentials\
Like I mentioned before, changing default usernames and passwords is critical since it's very easy to do and can have great consequences, such as being locked out of your own network. 

3. Enabling Secure Protocols\
Use protocols like SSH, HTTPS, SSL/TLS and others and not telnet, ftp, etc. Even better (for ssh), use public SSH-Keys for passwordless logins.

4. Disabling Unnecessary Scripts\
Almost every network device executes some startup scripts to provide a better user experience to a user, such as `crontab`. Threat actors try to gain persistent access on a network device by adding their malicious scripts on the startup. We can add/remove startup scripts and set the priority on our devices. 

5. Securing WiFi\
If the router has WiFi capabilities, increase security by enabling strong encryption like WPA2/WPA3, disabling SSID broadcast, changing default passwords, and more. 

6. Manage Traffic Rules (Firewalls)\
Network devices allow you to create and implement traffic rules that accept/deny network traffic, preventing infiltration and exfiltration. 

7. Monitor Traffic\
Network administrators should keep track of network traffic, like uploads and downloads of data at different intervals, which is essentual. For example, you have excessive data uploaded from one of the email servers to an unknown IP address. Such alerts enable you to take remedial measures and stop data pilferage quickly. Usually, network devices provide real-time graphs to monitor the traffic. 

8. Configure Port Forwarding\
A firewall's port forwarding capability enables inbound traffic from the internet or other sources to be routed to a particular device or service on the internal network. The firewall can send incoming traffic to the appropriate device or service on the internal network by establishing port forwarding rules while blocking any other incoming traffic that does not comply with the rules.

    However, port forwarding should be used carefully because it can expose internal devices and services to potential security issues if improperly secured and configured.

9. Monitoring Scheduled Tasks\
It is important to monitor scheduled tasks to confirm that the original scheduled tasks lists have not been not modified by an attacker or unauthorized actor.

10. Update Firmware\
As mentioned before, it is essential to update the firmware and installed packages on a regular basis to avoid any know/unknown attacks. Doing so can greatly reduce your risk. 

11. Configuring Port Security\
This includes limiting the number of MAC addresses registered on a switch port and taking particular action whenever unauthorized access is detected.

12. Preventing ARP Spoofing\
ARP spoofing is one of the most common vectors for launching MitM attacks on the network. The threat can be mitigated by enabling static ARP tables and implementing MAC address filtering. 

13. Preventing Rogue DHCP Servers\
The attacker creates a spoofed DHCP server that can be later on used for assigning IPs to clients and launching MITM attacks. Mitigation measures to prevent such attacks include configuring static DHCP binding and ensuring no unknown devices are added to a network through network mapping tools.

14. Enabling IPv6\
Unlike IPv4, IPv6 has built-in support of IPsec that can be used to secure network communication and provide confidentiality, integrity, and authenticity. Moreover, this will help in protection against MITM, eavesdropping, and tampering of packets in transit.



### Network Monitoring
I mentioned it before, but network monitoring is essential for maintaining security and performance of your network. There are various sophisticated tools and protocols you can use to capture and analyze real-time network traffic. They can also enable admins to detect and troubleshoot problems such as bandwidth bottlenecks, network outages, and security threats. Some of these tools include Nagios, SolarWinds Network Performance Monitor, PRTG and Zabbix, which all have different features. 


### Conclusion
This is by no means a complete checklist of things to do when attempting to secure your network devices. Items mentioned here just touch the surface on that specific that hardening technique; you could write several pages about just one of these techniques. Even books have been written just about firewalls. It should however, give you a good starting point and an insight to some of the bare minimum steps one should take. 