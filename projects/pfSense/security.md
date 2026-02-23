---
title: "Firewall and Network Segmentation Lab"
author: Jason Chan
---

# Network Security and Firewall Rules

This post will revolve around hardening our network security by introducing Firewall Rules to control traffic between subnets, protect infrastructure, and limit potential attack vectors.

<pre>
Our Current Network Topography

Internet (WAN)
      |
   pfSense
   /      \
LAN        USER_NET
(Infra)    (Users)
10.10.10.0/24   10.10.20.0/24

LAN:
  └─ Domain Controller (AD / DNS)

USER_NET:
  └─ Domain Users
</pre>

## Threat Model

Why are we implementing security measures? Well, we will assume that endpoints within `USER_NET` may get compromised through phishing, malware, etc.

Thus, I will set up security mitigations to address these specific points.

<ul>
      <li>Preventing lateral movement from USER_NET into infrastructure systems</li>
      <li>Protecting management interfaces (pfSense Web Configurator)</li>
      <li>Reducing exposed services to only those required for Active Directory domain operations</li>
</ul>

This allows us to enforce segmentation while also following the `Principle of Least Privilege`.

## Disabling pfSense's Web Configurator on USER_NET (OPTIONAL)

Currently, pfSense has an interface on `USER_NET` with the IP Address of `10.10.20.2`, which means users can theoretically access the pfSense Web Configurator by querying the interface IP address in a browser.

Not too important since the username and password for pfSense are still needed to access the Web GUI, but I figured it would be best practice to prevent users in `USER_NET` from reaching the pfSense Web Configurator, reducing the exposed attack surface.

We can create a rule to either `BLOCK` or `REJECT` any traffic from `USER_NET` to `10.10.20.2`. 

A `BLOCK` rule causes the packet to be dropped, which indicates a timeout. A `REJECT` rule instantly sends an immediate refusal message. Because we are dealing with internal traffic, I will use `REJECT` - better for users to be instantly notified that they cannot connect to a specific destination. For anything external, we will use `BLOCK` as that gives out the least information to attackers.

![Blocking port 80](/images/Proj%3A%20pfSense/http.png)

![Blocking port 443](/images/Proj%3A%20pfSense/https.png)

Here, I am blocking `TCP` (Transmission Control Protocol) traffic to ports 80 (HTTP) and 443 (HTTPS), preventing access to the Web Configurator from `USER_NET`.

Note: It is redundant to block `port 80` and `port 443`, but I am doing it more as a guide since I do not know if you have the pfSense Web GUI running on `HTTP` or `HTTPS`. I personally have it running on `HTTPS`.

![blocking web GUI](/images/Proj%3A%20pfSense/refuse.png)

*Querying for `10.10.20.2` on a user machine in `10.10.20.0/24` gives me a refused connection, web GUI can still be accessed through `LAN (INFRA_NET)`*

## Aliasing and Hardening USER_NET -> LAN (INFRA_NET)

Aliasing in pfSense provides `abstraction` by allowing firewall rules to reference logical objects instead of hardcoded IP addresses, networks, or ports. This improves rule readability and maintainability.

To harden traffic between `USER_NET` and `LAN (INFRA_NET)`, I created two aliases, `DC_SERVER` and `AD_REQUIRED_PORTS`. The objective is to enforce the `Principle of Least Privilege` by restricting `USER_NET` access to only specific `Active Directory` services required for normal domain operations.

![alias](/images/Proj%3A%20pfSense/alias.png)

<ul>
      <li><code>DC_SERVER</code> references the IP address of the DC (Domain Controller)</li>
      <li><code>AD_REQUIRED_PORTS</code> contains the set of ports required for essential AD services</li>
</ul>

| Port Range    | Protocol | Service              | Function                           |
|---------------|----------|----------------------|------------------------------------|
| 53            | TCP/UDP  | DNS                  | Domain Name Resolution             |
| 88            | TCP/UDP  | Kerberos             | Authentication (TGT/TGS)           |
| 123           | UDP      | NTP                  | Time Synchronization               |
| 135           | TCP      | RPC Endpoint Mapper  | RPC Service Coordination           |
| 389           | TCP/UDP  | LDAP                 | Directory Queries                  |
| 445           | TCP      | SMB                  | SYSVOL & Group Policy Access       |
| 49152–65535   | TCP      | Dynamic RPC          | Ephemeral RPC Communications       |

With Aliasing done, we can make a set of rules to enforce the `Principle of Least Privilege` between `USER_NET` and `LAN (INFRA_NET)`.

![rules for least priv](/images/Proj%3A%20pfSense/ruleset.png)

Rule order is critical - the allow rule for DC traffic must precede the deny rule to LAN, as pfSense evaluates rules top-down.

1. Allow `USER_NET -> LAN (DC on AD Ports)` to ensure AD functionality
2. Block `USER_NET -> LAN (except DC)` to prevent users from interacting with infrastructure other than DC
3. Allow `USER_NET -> WAN` for Internet Access
4. Restrict `USER_NET` for everything else for Implicit Denial

## Securing USER_NET Internet Access

pfSense has a default `WAN` rule that blocks all incoming traffic. However, we do not have any rules in place for outbound traffic from `USER_NET -> WAN (Internet)`. This means users can initiate connections to any external service on any port. This unrestricted access allows compromised devices to exfiltrate sensitive data, connect to command-and-control servers, or bypass security measures (tunneling traffic over open ports).

I have created another alias called `ALLOWED_OUTBOUND_PORTS` consisting of ports necessary to browse the Internet.

![alias for outbound](/images/Proj%3A%20pfSense/outbound.png)

*Allowing HTTP/HTTPS, DNS, and NTP*

Now, I have restricted WAN (Internet) access to only allow the ports defined in `ALLOWED_OUTBOUND_PORTS`, permitting just the necessary services for browsing, DNS resolution, and time synchronization.

![most recent rule](/images/Proj%3A%20pfSense/newrule.png)

We can test our newly configured firewall rule.

Logging in as a client machine on `10.10.20.0/24`, I will run PowerShell on `John-PC` and use the `Test-NetConnection` command.

![testing 443 and 80 port](/images/Proj%3A%20pfSense/porttest.png)

*Successfully connected to Google on ports 80 (HTTP) and 443 (HTTPS), demonstrating that users can access these ports*

![testing 22 port](/images/Proj%3A%20pfSense/porttest2.png)

*Failed to establish a connection to GitHub's port 22 (SSH), demonstrating that outbound traffic to non-approved ports is blocked*

![normal](/images/Proj%3A%20pfSense/porttest3.png)

*Successful connection to GitHub from my host computer (outside the VM) to demonstrate that GitHub does have an open port on 22 and is listening for SSH; this means that the previous failure was due to the firewall rather than the destination being unavailable*

## Summary

In this section, I have hardened our network security by implementing firewall controls in pfSense to enforce segmentation, reduce the attack surface, and limit lateral movement.

We started by assuming that endpoints in `USER_NET` (10.10.20.0/24) may be compromised (through phishing, malware, etc). This led me to introduce firewall rules to limit lateral movement and reduce attack vectors while maintaining domain functionality.

Key Actions:

<ul>
      <li>Restricting pfSense Web GUI on <code>USER_NET</code> to reduce management-plane exposure</li>
      <li>Creating pfSense Aliases to improve firewall rule clarity and maintainability</li>
      <li>Enforcing <code>Principle of Least Privilege</code> by limiting traffic from <code>USER_NET</code> to <code>LAN</code> to only required Active Directory ports for the Domain Controller (DC)</li>
      <li>Limiting outbound Internet Access from <code>USER_NET</code> to only essential services (DNS, HTTPS, HTTP) needed for Internet browsing to reduce exfiltration and command-and-control risk</li>
</ul>

<a href="/projects/pfSense/index.html">Back to pfSense Project's Page</a>









