---
title: "Windows Active Directory (AD) Lab"
author: Jason Chan
---

# Configuring Server to be a Domain Controller

## Setting a Static IP

To convert our Windows Server 2022 to a Domain Controller, we must first make the server's IP address static. This ensures the server maintains a consistent network identity and remains reliably reachable by clients/services on the network.

Before we actually change our IP address, it is best to figure out the current network topography. This can be done by running `ipconfig` in `Command Prompt`. This gives us the current subnet, default gateway, etc, ensuring boundaries are respected and we do not run into routing issues.


![ipconfig results](/images/Proj%3A%20AD/ipconfig.png) <br>
*Subnet mask is 255.255.255.0, that means 8 bits for host -> with .0 and .255 reserved, so 192.168.19.1-254 can be assigned*

Now, to change the IP address of the server, we must go into settings -> network & internet -> change adapter settings -> right click on ethernet0 -> properties -> select Internet Protocol Version 4 (TCP/IPv4)

![static config](/images/Proj%3A%20AD/static2.png) <br>
*Fill out the configurations based on YOUR ipconfig results. Also fill out Preferred DNS Server and Alternative DNS Server*

Modify the alternative DNS Server to 8.8.8.8 (Google's), else we will not have an internet connection since DNS queries are made through our newly set up server (which doesn't know anything about the internet), we will have trouble resolving domains to IP addresses. However, if we use Google's -> DNS queries are resolved, and we are "connected" to the Internet.

Now that we have a static IP address for our server, we also treat our server as a DNS server, this is because Active Directory heavily relies on DNS. AD uses DNS to locate domain services, it also allows clients to find the server (Domain Controller).

## Configuring Server with Server Manager

After configuring a static IP, we need to install Active Directory on our server, which can be done directly through the Server Manager.

<ol>
  <li>Server Manager</li>
  <li>Add Roles and Features (in Dashboard)</li>
  <li>Role/feature based installation</li>
  <li>Select AD DS (Active Directory Domain Services Role) and DNS</li>
  <li>Go through with installation process with recommended features</li>
</ol>

![finish downloading AD DS](/images/Proj%3A%20AD/download.png) <br>

## Promoting our Server to be a Domain Controller

Now, the final step is to promote the server to be a Domain Controller (DC) - which is basically a server in a network that hosts Active Directory and is responsible for authenticating and authorizing all users/computers within a domain.

![DC Promoting](/images/Proj%3A%20AD/promoteDC.png) <br>
*To promote our server to a DC, click on the little flag icon in Server Manager*

We will be prompted to 1. Add DC to an existing domain, 2. Add a new domain to an existing forest, or 3. Add a new forest.

A forest is basically a top-level container that can contain multiple domains, since we do not have a forest -> we need to select option 3 to add a new forest. This will prompt you to enter a name for your forest. I named mine `jasonCorp.local`.

You will also be prompted to set a directory services restore mode (DSRM) password, pretty important as DSRM is used to repair/restore the AD if something goes wrong.

We can skip DNS delegation since our server/DC is authoritative -> this means all domain queries will be answered by the DNS server, which is the DC, we do not need to delegate DNS lookups to another server.

Restart the server, and you have a functioning domain controller.







