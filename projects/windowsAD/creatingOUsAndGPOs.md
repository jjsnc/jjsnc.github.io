---
title: "Windows Active Directory (AD) Lab"
author: Jason Chan
---

# Creating Organizational Units (OUs) and Group Policy Objects (GPOs)

## How to create an Organizational Unit

Within the AD+DNS Windows Server 2022 VM, you can open up `Active Directory Users and Computers (ADUC)` by searching for it in the start menu, or by accessing it through Server Manager.

Within `ADUC`, you should see something along the lines of:

![ADUC](/images/Proj%3A%20AD/ADUC.png) <br>

Now, to create an OU, right-click on the domain -> New -> Organizational Unit

![new OU](/images/Proj%3A%20AD/ADUC2.png) <br>

You will then be prompted to create a name for your OU.

After creating an OU, you can create additional OUs within that newly created OU. I personally structured mine to be like this:

![OU categories](/images/Proj%3A%20AD/ADUC3.png) <br>
*One Employees OU, which contains an Employee Users OU, and an Employee Workstations OU*

The purpose of creating an OU for users and another for workstations will be seen later. It is very important, organizationally, and can impact the effectiveness of GPOs.

Now you can move the users you have created in the default `Users` OU into their respective OUs. In this case, I have placed John Doe and Jane Doe into Employee Users and their respective workstations from the default `Computers` OU into the Employee Workstation OU.

![demo](/images/Proj%3A%20AD/ADUC4.png) <br>
*You can ignore the Employee Security Group - that is for a feature that I will cover later*

![demo](/images/Proj%3A%20AD/ADUC5.png) <br>

Now, you should have a good organizational structure to implement Group Policy Objects (GPOs)

## How to create Group Policy Objects

Within the same VM - search for `Group Policy Management`.

![GP Management](/images/Proj%3A%20AD/GP.png) <br>
*Within Group Policy Management (GPM), you should see a relatively similar structure as ADUC*

Right-click the OU that you want to create a GPO for. A GPO is basically a set of rules and restrictions defined within an OU that applies to users/computers found within that OU. It is used to centrally manage/configure user/computer settings, which could include rules for security, applications, desktop environments, etc.

To create a GPO, all you would have to do is right-click on the OU that you would want the GPO to apply to, and then select `Create a GPO in this domain, and Link it here...`, which will prompt you for a name for that GPO.

![GP Management](/images/Proj%3A%20AD/GP2.png) <br>
*Your layout should be something like this*

## Group Policy Objects



