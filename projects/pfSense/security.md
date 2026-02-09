---
title: "Firewall and Network Segmentation Lab"
author: Jason Chan
---

# Network Security and Firewall Rules (under construction)

This post will revolve around hardening our network security by introducing Firewall Rules to control traffic between subnets, protect infrastructure, and limit potential attack vectors.

```markdown
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
```

## Disabling pfSense's Web Configurator on USER_NET (OPTIONAL)

Currently, pfSense has an interface on `USER_NET` with the IP Address of `10.10.20.2`, which means users can theoretically access the pfSense Web Configurator by querying the interface IP address in a browser.

Not too important since the username and password for pfSense are still needed to access the Web GUI, but I figured it would be best practice to prevent users in `USER_NET` from reaching the pfSense Web Configurator.

We can create a rule to either `BLOCK` or `REJECT` any traffic from `USER_NET` to `10.10.20.2`. 

A `BLOCK` rule causes the packet to be dropped, which essentially indicates a timeout. A `REJECT` rule instantly sends an immediate refusal message. Because we are dealing with internal traffic, I will use `REJECT` - better for users to be instantly notified that they cannot connect to a specific destination. For anything external, we will use `BLOCK` as that gives out the least amount of information to attackers.

![Blocking port 80](/images/Proj%3A%20pfSense/http.png)

![Blocking port 443](/images/Proj%3A%20pfSense/https.png)

Here, I am blocking `TCP` (Transmission Control Protocol) to block `HTTP` and `HTTPS`, fundamental application-layer protocols that run on `TCP` to serve up webpages.

Note: It is redundant to block `port 80` and `port 443` for pfSense Web Configurator, but I am doing it more as a guide since I do not know if you have the pfSense Web GUI running on `HTTP` or `HTTPS`. I personally have it running on `HTTPS`.

![blocking web GUI](/images/Proj%3A%20pfSense/refuse.png)

*Querying for `10.10.20.2` on a user machine in `10.10.20.0/24` gives me a refused connection*


## Aliasing
placeholderrrr


## Firewall Rules: Restricting USER_NET Access to LAN (INFRA_NET)

With our current setup, any users from `USER_NET` can route traffic to our `LAN` (INFRA_NET). That means that if a user device is compromised, the attacker can access critical infrastructure (like our DC) because there are no mitigations in place.

