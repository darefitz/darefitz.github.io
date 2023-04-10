---
title: "Setting up a Home Lab with Active Directory and 1k+ users"
date: 2023-04-09T17:58:53-05:00
draft: false
---

![Network Diagram](/diagram.png)

I set up a Windows Server 2019 with Active Directory and generated 1k users with PowerShell, and use the server as a domain controller for a Windows 10 VM to connect to.

I used VMWare for the server and Win 10 Machine.
First I setup the 2019 server:
![Windows 2019 Server](/1.png)

Then I had to ensure that I had an external internet and an internal network:
![Networks](/image(1).png)

After figuring out which network was the NAT I configured the ip address for the internal network based off my diagram:
![Internal ip](/image(2).png)

After, I installed Active Directory Domain Services, (AD DS) on the server.
Once that installed I setup the powershell script to create 1k+ users to simulate a corporate setting.
The users created are in the format: flastname (f=first letter in firstname) Ex: John Smith = jsmith

![Powershell Script](/image(3).png)

Here's a look at the users in our Active Directory:

![AD users](/image(4).png)

Now that the AD was setup I went over to the Win 10 VM to make sure that our dhcp was working.
As you can see the Domain Controller was working and we were assigned the first ip in our range:
![DHCP test](/image(5).png)
And our pings were successful:
![DHCP test](/unnamed.png)

Next I setup the machine to be connected to the domain service so we can login as our created users:
![Domain Assignment](/image(6).png)

Back on the server we can see that the client machine connected successfully:
![DHCP Connection](/image(7).png)
![AD Connection](/image(8).png)

Back on the client I logged in with the Domain group as the user dfitzpatrick:
![AD Connection](/whoami.png)

Setting up a Windows Server 2019 with Active Directory and generating sample users allows you to test managing users, computers, and other resources in a centralized manner, which can greatly enhance your organization's IT infrastructure.