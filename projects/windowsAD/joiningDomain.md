---
title: "Windows Active Directory (AD) Lab"
author: Jason Chan
---

# Joining the Domain For Users/Clients

## Windows 11 Evaluation Copy

After you have installed the Windows 11 Eval Copy ISO, you can open a 2nd virtual machine (a 3rd one even) quite easily. Edit the settings for the VM like you have done for the Domain Controller/Windows Server 2022, locate the ISO file, and follow through with the installation steps.

After you have finished setting up the Eval copy, during login, you will be prompted to sign into a Microsoft Account (you shouldn't), instead, opt to select the option to join a domain. This allows you to create a `local account`, way easier than setting up an entire Microsoft Account (especially if you have to do this for several users).

At this point, you can open your third VM (2nd client machine for another Windows 11 computer to simulate multiple users/computers). Optional, but highly recommended.

## DNS Reassignment

For new users/computers that join the domain, they must use the AD DNS server rather than the default assigned one. You can check if the user is currently connected to the AD DNS Server by doing `ipconfig /all` in the command prompt.

![ipconfig results](/images/Proj%3A%20AD/ipconfig.png) <br>
*Running ipconfig /all on a client Windows 11 computer*

As you can see, the current DNS Server does not point to the static IP address 192.168.19.100 that we set before. We must set the DNS Server to be 192.168.19.100 for this client, or else it will not be able to find the domain controller within the network, which means it will not be able to connect to the domain and access related AD services. This can be done in settings via DNS server assignment.

![reassigning DNS preference](/images/Proj%3A%20AD/ipconfig2.png) <br>
*Click on edit, and then input the static IP address that you have configured for the AD+DNS server*

![ipconfig again](/images/Proj%3A%20AD/ipconfig3.png) <br>
*Now, run ipconfig /all again, and you should see that the preferred DNS server is the static IP that you have for your AD+DNS server*

## Creating Users in Active Directory Users and Computers (ADUC)

Now, we have to actually create user accounts for our clients who will be joining our domain. This can be done via ADUC or Active Directory Users and Computers.

![how to find ADUC](/images/Proj%3A%20AD/ADUC.png) <br>
*Select AD DS, and then right-click on the server -> and select Active Directory Users and Computers*

![picture of ADUC](/images/Proj%3A%20AD/check.png) <br>
*This is an example of how ADUC would look*

Navigate into the `Users` Organizational Unit (OU). An OU is basically the folders you see within ADUC. It is a folder/container that is used to organize users/computers/resources within the AD environment. The `Users` OU should be automatically created and prefilled.

Now, inside the `Users` OU, right-click -> new -> User.

![user setup](/images/Proj%3A%20AD/newuser.png) <br>
*Fill out accordingly*

![user setup2](/images/Proj%3A%20AD/newuser2.png) <br>
*It will prompt you to create a password and other password settings*

![user setup3](/images/Proj%3A%20AD/newuser3.png) <br>
*After you have finished setting up the user, you should see that user populate inside the `Users` OU*

![user setup4](/images/Proj%3A%20AD/newuser4.png) <br>
*Not required, but definitely helpful to have multiple users*

## Joining The Domain

Now, on the client computers (Windows 11 VMs), we need to join the actual domain. This can be done via the settings. Go into settings -> system -> about -> Domain/Workgroup. Another way is settings -> accounts -> Access work or school. 

![joining the domain](/images/Proj%3A%20AD/join.png) <br>
*Input your forest/domain name*

![joining the domain2](/images/Proj%3A%20AD/join2.png) <br>
*USE ADMINISTRATOR TO LOG IN, basically administrator and the same password used to log in to your Windows Server 2022 (AD+DNS)*

![joining the domain3](/images/Proj%3A%20AD/join3.png) <br>
*A successful join will prompt you to restart*

If at any point you cannot join your domain, `DNS` will probably be your best option to troubleshoot. Otherwise, you may have installed Windows 11 HOME edition rather than a Windows 11 Enterprise Evaluation copy.

![joining domain greyed out](/images/Proj%3A%20AD/lockedout.png) <br>
*If you cannot select to join a domain, it means you have downloaded the Windows 11 Home edition, in which case you need to restart the client process*

You may wonder why I used the administrator account to join the domain. If we use the newly created user accounts (as we did in ADUC), we will encounter the following problem.

![cannot join](/images/Proj%3A%20AD/join4.png) <br>
*Trying to log in with the newly created username and password within ADUC*

![cannot join](/images/Proj%3A%20AD/join5.png) <br>
*Error prompt, to circumvent, need to log in as administrator FIRST*

## Logging into the Domain

After you have successfully joined the domain, to log into the domain, we need to select `Other User`, and log in with the username/password combination we created in `ADUC`.

![logging into the domain](/images/Proj%3A%20AD/login.png) <br>

![prompt new password](/images/Proj%3A%20AD/login2.png) <br>
*This will pop up if you have selected the option that the user must change their password for their next login*

![logging into the domain](/images/Proj%3A%20AD/login3.png) <br>
*Enter the old password alongside the new password you want*

![password complexity check](/images/Proj%3A%20AD/login4.png) <br>
*You may run into the following problem due to the default account password policy -> make the password more complex, include symbols, numbers, capital letters, make it decently long, and make sure to not include the username in the password*

![confirmation](/images/Proj%3A%20AD/login5.png) <br>
*To confirm you have successfully signed in, just press the Windows key and click on the account*

Now do the same for your other client VM (if you have one). And you are basically done. Next step is to organize everything (OUs) and to create Group Policy Objects (GPOs).





















