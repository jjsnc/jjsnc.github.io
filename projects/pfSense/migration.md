---
title: "Firewall and Network Segmentation Lab"
author: Jason Chan
---

# Migrating an Active Directory Domain Behind a pfSense Firewall

I will be assuming that you have an existing Active Directory Domain and that you would like to put a firewall between the domain and the Internet. However, if you do not have an existing Active Directory Domain, you can check out my [previous project](https://jjsnc.github.io/projects/windowsAD/) to build one from scratch, or you may follow along with just pfSense and a Windows 11 client (specific steps will differ, but general procedures will likely be similar).

## Installing pfSense/Netgate Installer ISO File

Before we even do anything, we must install the necessary software. Search up pfSense on your browser and locate the `Netgate Installer`. Previous versions of pfSense did not require an installer; however, for the most up-to-date pfSense CE (Community Edition), you must create an account on Netgate to download it. If you do not wish to create an account, you can get outdated versions of pfSense from this [mirror](https://atxfiles.netgate.com/mirror/downloads/).

I personally got the ISO version because I will be installing pfSense onto a virtual machine with VMware.

## Setting up VM

After you have installed the ISO file for pfSense, you will need a virtual machine to run it. I am using VMware. Create a new VM and specify to use the pfSense ISO file.

The specs for the virtual machine:
<ul>
  <li>1 Processor with 2 Cores</li>
  <li>2 GB RAM</li>
  <li>20 GB Disk</li>
</ul>

Processor/cores should not really matter too much - the virtual machine will run only the console, which is not performance-heavy. If you want to collect logs, I recommend 20 GB for the disk or even more. I am personally planning on collecting logs through Splunk (I will touch upon this in a later subpart).

For the specific network adapter, I selected `NAT` (network address translation). This allows the VM to have internet access while also isolating it from the physical network. VMware will perform network address translation for outbound traffic while forwarding return traffic to the correct VM.

## Installing pfSense Inside VM

After creating the new VM and everything works, you should see the Netgate Installer in the VM.

![netgate installer](/images/Proj%3A%20pfSense/dl1.png)

The WAN (Wide Area Network) is already active due to `NAT` from our hypervisor, providing an upstream connection to the pfSense VM. 

Our task is to install pfSense onto the LAN (Local Area Network) interface, which is the currently inactive one (since we have yet to set it up). We will be essentially configuring the LAN interface since this represents the network that pfSense will protect.

Follow through with the installation:

![netgate installer](/images/Proj%3A%20pfSense/dl2.png)

*We will use the CE (Community Edition)*

![netgate installer](/images/Proj%3A%20pfSense/dl3.png)

*It is okay to select Stripe for a lab environment*

![netgate installer](/images/Proj%3A%20pfSense/dl4.png)

*If you chose to download pfSense from the mirror, the latest you would have is 2.72*

![netgate installer](/images/Proj%3A%20pfSense/dl5.png)

After the installation is complete, you should see the pfSense console, which looks something like:

![pfSense console](/images/Proj%3A%20pfSense/pfconsole.png)

to be continued









<a href="/projects/pfSense/index.html">Back to pfSense Project's Page</a>
