---
title: "Firewall and Network Segmentation Lab"
author: Jason Chan
---

# Migrating an Active Directory Domain Behind a pfSense Firewall

I will be assuming that you have an existing Active Directory Domain and that you would like to put a firewall between the domain and the Internet. However, if you do not have an existing Active Directory Domain, you can check out my [previous project](https://jjsnc.github.io/projects/windowsAD/) to build one from scratch, or you may follow along with just pfSense and a Windows 11 client (specific steps will differ, but general procedures will likely be similar).

## Migrating the Domain Controller

To migrate the Domain Controller, we will need to assign the Domain Controller a static IP address within the LAN subnet `10.10.10.0/24`. We go into `Ethernet Properties` and into `Internet Protocol Version 4 (TCP/IP) Properties` to assign an IP address manually.

![DC into LAN](/images/Proj%3A%20pfSense/DCLAN.png)

![DC into LAN](/images/Proj%3A%20pfSense/DCLAN2.png)

The DC is assigned `10.10.10.25` with the default gateway being the pfSense LAN interface IP address `10.10.10.2`. This essentially tells the DC to route traffic through pfSense (`10.10.10.2`) if the destination host is outside our current network (LAN).

Unfortunately, we run into a problem...

![DC into LAN PROBLEM](/images/Proj%3A%20pfSense/DCLAN3.png)

It seems like we cannot reach the gateway since our `ping` fails, which is weird.... since we are in the same subnet. However, the problem here illustrates the difference between `Layer 2 (Data Link Layer)` and `Layer 3 (Network Layer)` in the `OSI Model`. Although we are in the same subnet, there is a physical limitation.

The DC sees the destination as `10.10.10.2`, and knows that it is inside the subnet - so it should talk to it directly (through Layer 2). It uses `ARP`, the address resolution protocol, to map the destination IP address to a local MAC address. However, this fails.

`ARP` sends a broadcast on the local network... `Who has 10.10.10.2? Tell 10.10.10.25`, but because pfSense is not on the same Layer 2 Network, the broadcast does not reach it, and it never hears the `ARP` request, prompting the gateway to be unreachable.

The fix is actually quite easy! One thing I overlooked was that I never placed the DC VM on the same LAN segment as the VM running pfSense. This means, even though the IP address is inside the same subnet, it cannot physically reach the gateway because it lives on a different virtual switch.

![fix](/images/Proj%3A%20pfSense/fix1.png)

*Go into the VM settings for the DC and choose the LAN segment that we made*

![fix](/images/Proj%3A%20pfSense/fix2.png)

*After configuring the network adapter for the DC VM, we can ping the gateway*

## pfSense Web Configurator

Knowing that the DC can reach the default gateway `10.10.10.2`, we can go into the pfSense Web Configurator on the DC VM, the web-based GUI that allows you to manage the firewall without using the console. To get to the Web Configurator, all we need to do is query for the LAN interface IP address in a browser.

![web config](/images/Proj%3A%20pfSense/web.png)

![web config](/images/Proj%3A%20pfSense/web2.png)

*After going through the Web Configurator and leaving settings as default/recommended, we arrive at the homepage*

PLACEHOLDER - CONTINUE HERE

## Migrating Windows 11 Domain Users

After migrating the DC, we also need to migrate the domain users `John Doe` and `Jane Doe` (two separate Windows 11 VMs). The first thing we need to do is configure their network adapters to point to LAN_NET (the LAN segment we created). After that, to ease the transition, I temporarily disabled (unlinked) the `GPOs` that restricted their access to the Windows Settings.

![Unlinking GPOs](/images/Proj%3A%20pfSense/unlink.png)

*Unlinking GPOs that hid and restricted domain users from accessing settings*

![Unlinking GPOs](/images/Proj%3A%20pfSense/unlink2.png)

*Running gpupdate /force to force GPO update on client VMs*

However, we run into this problem of policies failing to update... We moved our DC, which means Active Directory breaks because client VMs are still using the previous static IP address of the DC to resolve `DNS` lookups. To fix this, we need to assign the preferred DNS server to `10.10.10.25` (where the DC currently is).

PLACEHOLDER - CONTINUE HERE



<a href="/projects/pfSense/index.html">Back to pfSense Project's Page</a>
