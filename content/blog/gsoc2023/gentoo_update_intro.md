---
title: "gentoo_update Introduction"
date: 2023-06-19T22:39:27+03:00
tags:
    - opensource
    - gsoc
    - gentoo
categories:
    - GSoC2023
ShowToc: true
---

### Introduction
`gentoo_udpate` is a tool that automatically updates Gentoo Linux.  

**Motivation**  
Gentoo Linux gives users maximum flexibility and control over the system. 
A great example of this is the OS upgrade process. Users have a large choice 
of different command utilities and a bunch of configuration options to choose 
from to tailor the upgrade process to their needs. Here is the list of just 
some tools that are commonly used during an upgrade: 

```yaml
[
    eix, equery, emaint, euse, etc-update, dispatch-conf,  
    eselect, elogv, needrestart, eclean, eclean-kernel, 
    qcheck, revdep-rebuild, glsa-check, layman 
]
```

For a successful upgrade, knowing how to use many of these tools is essential. 
While experienced users might find this manageable, new or inexperienced users 
may find it overwhelming.  
![Gentoo Update Meme](/img/lb_gentoo_update_meme.jpg#center)

Additionally, users often delay updates due to the time required, which can 
compromise system security. Regular updates are vital for maintaining security, 
so it is recommended to update the system daily.  

This project addresses both issues:
1. complicated update process 
2. potential security issues caused by the lack of regular upgrades


### Functionality
Here are some of the things that gentoo_update will be able to:
* By default install only security updates from Gentoo Linux Security Advisory.
* Optionally run a full system upgrade (`@world`) with different parameters.
* Detect and handle update errors.
* Schedule updates.
* Generate a post-update report and send it via email and/or IRC chat.
* Send push notifications to a **mobile app**.  

The program comprises three core components: the updater, the parser, and the 
notification sender. The updater is a Bash script that executes emerge to update 
the system and generates detailed logs for each action performed. Upon successful 
completion of the updater, the parser reads the logs and compiles an update report. 
The notification sender then dispatches this report to users.  

