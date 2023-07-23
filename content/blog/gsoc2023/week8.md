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
Currently, the updater supports two methods of notifications: IRC bot and email.

The IRC bot was built using Python's sockets library with SSL support. 
Although functional, it remains quite basic and encounters issues with sending out 
the report properly in approximately 20% of cases. The issue seems to occur during 
connection to `irc.libera.chat` servers, though the exact problem remains unclear.  

In addition, there's an option to send the report via email using SendGrid. 
This service was selected due to its free registration and simplicity of use because 
it only requires an API key.  


### Challenges
The initial challenge involved figuring out an effective way to send the 
report to the IRC chat. The program has a short 10-second buffer to ensure 
the message is sent properly. However, with reports that could be tens or 
hundreds of lines long, this process can take a bit longer. The current 
solution is to send a brief report that merely indicates if the update was 
successful. After this, the bot will ask if a more detailed report is needed.  

Future plans also involve setting up a local email relay using `sendmail` and `postfix`. 
However, this method is accompanied by several challenges. For instance, only one MTA 
(mail transfer agent) can be installed, which must be reflected in the ebuild. 
Also, configuring an email relay on Linux systems typically involves more steps, 
which requires writing a comprehensive documentation.  


### Plans for Week 9
This week, I plan to start working on the web app's design and architecture layout.

At the same time, there are several code enhancements that need to be implemented. 
For instance, the current logger only covers the updater script, neglecting the parser, 
reporter, and notifier. Thus, it needs to be extended to cover all components of the program.  

In terms of report formatting, the report is currently structured as a dictionary. 
However, it would be more beneficial to refactor it into a Python object, such as a dataclass.  

Lastly, the way `gentoo_updater` accepts its CLI flags could be improved. 
Currently, either `y` or `n` must be passed to the CLI, as in:
```bash
gentoo_update --update-mode full --read-logs y --read-news y
```
  
It looks a bit cubmersome, since if the flag is present then `y` is already implied.  

