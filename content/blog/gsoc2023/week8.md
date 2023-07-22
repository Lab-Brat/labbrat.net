---
title: "GSoC 2023: Weeks 8"
date: 2023-07-22T14:43:37+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **week 8** of GSoC.  

Project is hosted on [GitHub](https://github.com/Lab-Brat/gentoo_update), 
blog post can be also found on 
[Gentoo Blogs]().    


### Progress on Weeks 8
So far the updater supports 2 methods of notifications: IRC chat and email. 

I have also made some plans to incorporate more OOP into the app. For instance, 
right now the report is a Python dictionary, but I am planning the rewrite it 
to a dataclass.


### Challenges
The initial challenge involved figuring out an effective way to send the 
report to the IRC chat. The program has a short 10-second buffer to ensure 
the message is sent properly. However, with reports that could be tens or 
hundreds of lines long, this process can take a bit longer. The current 
solution is to send a brief report that merely indicates if the update was 
successful. After this, the bot will ask if a more detailed report is needed.

### Plans for Week 9
