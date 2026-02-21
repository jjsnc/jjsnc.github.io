---
title: "Firewall and Network Segmentation Lab"
author: Jason Chan
---

# Collecting Logs With Splunk Pt. 1

In this section, I will go over log collection and analysis using `Splunk` as my `SIEM` (Security Information and Event Management) platform. A `SIEM` is critical in network security because it aggregates logs across the network, enabling centralized monitoring of activity for threat detection, incident response, and overall security management.

I will be targeting two main sources of logs: Windows Logs from my Domain Controller (DC) and pfSense Firewall Logs.

## Installing Ubuntu Server

I will be creating a new virtual machine (VM) that will host an Ubuntu Server, which will act as our Splunk Server. Following the installation, I changed the network adapter from `NAT` to `Custom: LAN_NET`. This means my server should now exist within the same network as my DC, and should be automatically behind the pfSense firewall.

![Ubuntu Server Installation](/images/Proj%3A%20pfSense/home.png)

*Notice the IP Address - 10.10.10.102. This means pfSense DHCP successfully assigned our new Ubuntu Server an IP Address (pfSense DHCP assigns IP addresses from 10.10.10.100 to 10.10.10.254*

Next, we actually want the Ubuntu Server to have a static IP Address.



<a href="/projects/pfSense/index.html">Back to pfSense Project's Page</a>
