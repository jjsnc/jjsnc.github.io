---
title: "Firewall and Network Segmentation Lab"
author: Jason Chan
---

# Introduction to Firewalls

To start this project, we must understand exactly what firewalls are and why they are a fundamental component of modern networks.

## So... what exactly is a firewall? What do they do? Why?

A firewall, at its core, is a network security control that acts like a barrier between a trusted network (like your home network) and untrusted/external networks (like the internet). They can exist in different forms, hardware or software, personal or on top of a network, or even exist in the cloud. However, they all serve one purpose: to prevent unauthorized or unnecessary traffic from reaching the internal network.

Firewalls `monitor`, `allow`, or `block` network traffic based on a set of rules, allowing administrators to filter traffic coming in and out of a network. By placing a firewall between the internal network and the external network, access is no longer implicit. All communications will need to be explicitly allowed, reducing exposure to malicious actors and allowing corporations to meet compliance requirements.

## What is pfSense?
[pfSense](https://www.pfsense.org) is an open-source firewall based on FreeBSD, a Unix-like operating system. Rather than the automatically configured firewalls that you would find on computers, pfSense is a network firewall, blocking traffic on a network level - crucial for big corporations to scale their operations and to centralize security. They offer a console configurator alongside a web configurator that allows us to set up, define, segment, and secure networks.

## What exactly will we be doing?
We will be experimenting with pfSense, likely setting up inbound/outbound traffic rules, and then testing our firewall against a simulated attack from another virtual machine (Kali). But first, we must migrate our AD domain to be behind the pfSense firewall, which will be the topic of the next post.



<a href="/projects/pfSense/index.html">Back to pfSense Project's Page</a>
