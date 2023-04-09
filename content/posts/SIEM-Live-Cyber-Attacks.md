---
title: "SIEM Live Cyber Attacks"
date: 2023-04-09T12:24:20-05:00
draft: false
author: "Darren Fitzpatrick"
---
I setup Azure Sentinel (SIEM) and connect to a live virtual machine acting as a honey pot. The virtual machine is a windows 10 with all firewalls turned off to expose it to attackers ASAP. Once the VM was spun up I setup a custom powershell script that filters thru events and grabs failed login logs, then uses those events to get the attacker's ip and uses a free geo-locate ip api service.

![Powershell](/powershell.png)

After I setup the output logs I had to create filters to parse and grab each field. Each log's filter will be used to setup a queue that will refresh every 5 mintues to geolocate the attacks on a map.

After about two hours the first brute force attacks started to come in. As you can see majority of the attacks came from India. (4/8/2023)
![Attack Map](/ipmap.PNG)

Five hours later... (4/8/2023)
![Attack Map](/5hour.png)

The next morning... (4/9/2023)
![Attack Map](/nextmorning.png)
