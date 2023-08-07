---
title: "GSoC 2023: Week 9"
date: 2023-07-30T21:47:19+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **week 9** of GSoC.  

Project is hosted on [GitHub](https://github.com/Lab-Brat/gentoo_update), 
blog post can be also found on 
[Gentoo Blogs](https://blogs.gentoo.org/gsoc/2023/08/07/week-910-report-automated-gentoo-system-updater/)


### Progress on Week 9
This week, much of my time was devoted to improving Dart and Flutter skills, 
preparing to develop the mobile app and researching app architecture.  

I have made some `gentoo_update` code improvements as well:
* Parse update log into dataclass instead of a dictionary.
* Remove `y/n` option for CLI flags, now if the flag is present it automatically means `y`
* Read multiple mount point disk usage stats, instead of just /
* Simplify package info parsing by replacing complex regular expressions with string splitting


### Challenges
It was very challenging to create a suitable app architecture, and this task is still 
not 100% complete.  

Initially, my mentor and I considered the possibility of skipping the creation 
of a Firebase backend entirely, instead delegating that functionality to users. 
This stemmed from our desire not to store user package information in the 
backend database. However, this approach has several significant limitations:
1. Availability: Without a backend, the app will only be functional if it's within the same local network as the Gentoo machine.
2. Security: To receive notifications from anywhere, users would need to open a port on their mobile device publicly, reducing the device's security.
3. Complexity: Users might choose to deploy a proxy server or an edge function to handle the notification delivery, but this could be a complex procedure.  

Given the aforementioned limitations, implementing a backend might actually be 
a correct approach. This could be done securely without storing any information, 
if its sole use is for notification delivery.  

During user authentication in the app (which unfortunately is unavoidable), the backend 
would register this app instance. Within the app's interface, the user could then register 
their Gentoo device by clicking a button. This action would generate a token usable for 
sending the report via curl using HTTPS. The report itself would be encrypted by `gentoo_update` 
and could only be decrypted by the app instance. This approach offers several advantages:  
1. User Data: User log information is stored on the Gentoo machine and the mobile device, while the backend only stores the username and app instance.
2. Availability: The app could receive notifications from anywhere.
3. Security: The backend would handle the delivery of the encrypted report, so users won't need to compromise their security.  

Also, there are a few alternative for Firebase that are developed by opensource communities, 
for example [appwrite](https://appwrite.io) and [supabase](https://supabase.com).  


### Plans for Week 10
By the end of the week 10 I plan to have a working UI for the mobile app.  

