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

Now, let's run a few tests...

![running a few ping tests](/images/Proj%3A%20pfSense/test.png)

*Pinging the gateway from DC works; however, it looks like our DC cannot reach the Internet (pinging `8.8.8.8` fails)*

What I tried to fix this was to add a routing gateway within the Web Configurator.

![adding a gateway](/images/Proj%3A%20pfSense/routing.png)

*Systems -> Routing*

![adding a gateway](/images/Proj%3A%20pfSense/routing2.png)

*Add new gateway*

![adding a gateway](/images/Proj%3A%20pfSense/routing3.png)

*Configuring gateway settings*

![adding a gateway](/images/Proj%3A%20pfSense/routing4.png)

*Applying the changes*

However, this did not fix the problem. Instead, we get a different error, which hints at an infinite loop within the network. By adding the LAN gateway, we are basically telling all traffic leaving the LAN to route to `10.10.10.2`, which is the pfSense LAN interface, basically a self-referential loop. I decided to delete our newly created gateway since that did not help.

![adding a gateway](/images/Proj%3A%20pfSense/routing5.png)

After looking browsing around, I went back to the pfSense console, and that is when I realized:

![console misconfig](/images/Proj%3A%20pfSense/solution1.png)

The `em0` or `WAN` interface is not assigned to anything, which explains why we are not connected to the internet since the `WAN` interface is the path to the `Internet`.

![WAN backup](/images/Proj%3A%20pfSense/solution2.png)

*After resetting and configuring the WAN interface, it has an IP address now*

![WAN backup](/images/Proj%3A%20pfSense/solution3.png)

*Back in the Web Configurator, we see the Internet symbol appear next to our WAN gateway*

![WAN backup](/images/Proj%3A%20pfSense/solution4.png)

*Pinging 8.8.8.8 works now - we are connected to the Internet*

## Migrating Windows 11 Domain Users

After migrating the DC, we also need to migrate the domain users `John Doe` and `Jane Doe` (two separate Windows 11 VMs). The first thing we need to do is configure their network adapters to point to LAN_NET (the LAN segment we created). After that, to ease the transition, I temporarily disabled (unlinked) the `GPOs` that restricted their access to the Windows Settings.

![Unlinking GPOs](/images/Proj%3A%20pfSense/unlink.png)

*Unlinking GPOs that hid and restricted domain users from accessing settings*

![Unlinking GPOs](/images/Proj%3A%20pfSense/unlink2.png)

*Running gpupdate /force to force GPO update on client VMs*

However, we run into this problem of policies failing to update... We moved our DC, which means Active Directory breaks because client VMs are still using the previous static IP address of the DC to resolve `DNS` lookups. To fix this, we need to assign the preferred DNS server to `10.10.10.25` (where the DC currently is).

We need to go into Settings -> Network & Internet -> Ethernet.

![dns reassignment](/images/Proj%3A%20pfSense/reassign.png)

*Notice how the DNS server points to our old AD+DNS server's IP address*

![dns reassignment](/images/Proj%3A%20pfSense/reassign2.png)

*Changing requires administrator privileges, log in with the user `administrator` and the password you used to sign into the Windows Server 2022 administrator account*

![dns reassignment](/images/Proj%3A%20pfSense/reassign3.png)

![dns reassignment](/images/Proj%3A%20pfSense/reassign4.png)

Notice how DNS points to our new DC IP address, `10.10.10.25`, the IPv4 address of the current host (assigned by pfSense's DHCP upon joining the LAN, ranging from 10.10.10.100 to 10.10.10.254), and the default gateway being the pfSense LAN interface, `10.10.10.2`.

![dns reassignment](/images/Proj%3A%20pfSense/reassign5.png)

*Testing ping on the client Windows VM -> it is connected to the internet while being behind the firewall*

Now, let's move Jane behind the firewall.

![moving jane](/images/Proj%3A%20pfSense/jane.png)

*Before migration, notice that the DNS still points to the old DC IP address. Also, pfSense's DHCP gave Jane 10.10.10.101*

Now, we just do the exact same steps that we did for John... however, we encounter a problem.

![moving jane](/images/Proj%3A%20pfSense/jane2.png)

Somehow, Jane is still under the `GPO` restrictions even though I have unlinked them. Looks like we cannot access the network settings this way. 

One way we can circumvent this is to log into Jane's computer using the local administrator account.

![moving jane](/images/Proj%3A%20pfSense/jane3.png)

*Using `.\[local account username]` and the original password that was used to set up the workstation*

![moving jane](/images/Proj%3A%20pfSense/jane4.png)

*We can then change the DNS server preference on the local administrator account*

![moving jane](/images/Proj%3A%20pfSense/jane5.png)

*Logging back into the domain account, we now run ipconfig /all and see that the changes were applied*

## After Migration

After migrating both the DC and Domain Users behind the firewall, I relinked the GPOs within Group Policy Management inside the AD+DNS Server VM.

![relinking GPO to enforce rules](/images/Proj%3A%20pfSense/relink.png)

*Was temporarily disabled/unlinked, now relinked to enforce GPOs once again*

![update GPO](/images/Proj%3A%20pfSense/relink2.png)

*Running gpupdate /force on John -> seems like John can see the DC now, implying our domain is back up*

![update GPO](/images/Proj%3A%20pfSense/relink3.png)

*Running gpupdate /force on Jane, same result*

We have successfully migrated our AD domain behind the pfSense firewall.

## Summary / Current Network Topography

<table>
  <thead>
    <tr>
      <th>Component</th>
      <th>Interface</th>
      <th>IP Address</th>
      <th>Subnet</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>pfSense</td>
      <td>LAN</td>
      <td>10.10.10.2</td>
      <td>10.10.10.0/24</td>
      <td>Default gateway for LAN</td>
    </tr>
    <tr>
      <td>pfSense</td>
      <td>WAN</td>
      <td>DHCP</td>
      <td>Upstream ISP</td>
      <td>Internet access</td>
    </tr>
    <tr>
      <td>Domain Controller</td>
      <td>Ethernet</td>
      <td>10.10.10.25</td>
      <td>10.10.10.0/24</td>
      <td>AD DS + DNS</td>
    </tr>
    <tr>
      <td>Windows 11 (John)</td>
      <td>Ethernet</td>
      <td>10.10.10.100</td>
      <td>10.10.10.0/24</td>
      <td>Domain Client</td>
    </tr>
    <tr>
      <td>Windows 11 (Jane)</td>
      <td>Ethernet</td>
      <td>10.10.10.101</td>
      <td>10.10.10.0/24</td>
      <td>Domain Client</td>
    </tr>
  </tbody>
</table>



<a href="/projects/pfSense/index.html">Back to pfSense Project's Page</a>
