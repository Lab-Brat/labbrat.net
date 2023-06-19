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
`gentoo_udpate` (Github [repo](https://github.com/Lab-Brat/gentoo_update)) 
is a tool that automatically updates Gentoo Linux.  

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


### Usage
At the moment `gentoo_update` can only install GLSA and `@world` updates and store 
the output to a dedicated directory. It resides in [GURU](https://wiki.gentoo.org/wiki/Project:GURU) 
overlay in [app-admin/gentoo_update](https://github.com/gentoo/guru/tree/master/app-admin/gentoo_update).  

After enabling GURU overlay it can be installed via:
```bash
emerge --ask app-admin/gentoo_update
```

Here are some use cases:
**Security update**
```bash
gentoo-update
```

**@world update**
Run full system update, merge all new configuration files, restart all services that were 
updated and display elogs.
```bash
gentoo-update --update-mode full --config-update-mode merge --daemon-restart y --read-logs y
```

update with `--keep-going` flag.
```
gentoo-update --update-mode full --args  "keep-going=y"
```

After an udpate a log file will be created in `/var/log/portage/gentoo_udpate/log_<timestamp>`.

