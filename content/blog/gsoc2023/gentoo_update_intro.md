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
`gentoo_update` (Github [repo](https://github.com/Lab-Brat/gentoo_update)) 
is a tool that automatically updates Gentoo Linux.  

**Motivation**  
Gentoo Linux gives users maximum flexibility and control over the system. 
A great example of this is the OS upgrade process. Users have a large selection 
of different command utilities and a bunch of configuration options to choose 
from to tailor the upgrade process to their needs. Here is the list of some 
tools that are commonly used during an upgrade: 

```yaml
[
    eix, equery, emaint, euse, etc-update, dispatch-conf,  
    eselect, elogv, needrestart, eclean, eclean-kernel, 
    qcheck, revdep-rebuild, glsa-check, layman
]
```

For a successful upgrade, knowing how to use many of these tools is essential. 
While experienced users might find this manageable, it can be overwhelming for 
new or inexperienced users.
![Gentoo Update Meme](/img/lb_gentoo_update_meme.jpg#center)

Additionally, users often delay updates due to the time required, which can 
compromise system security. Regular updates are vital for maintaining security, 
so it is recommended to update the system daily.  

This project addresses both issues:
1. complicated update process 
2. potential security issues caused by the lack of regular upgrades

&nbsp; 

### Functionality
Here are some of the things that `gentoo_update` will be able to do:
* Install only security updates from Gentoo Linux Security Advisory by default.
* Optionally run a full system upgrade (`@world`) with different parameters.
* Detect and handle update errors.
* Schedule updates.
* Generate a post-update report and send it via email and/or IRC chat.
* Send push notifications to a mobile app.  

The program comprises three core components: the updater, the parser, and the 
notification sender. The 
[updater](https://github.com/Lab-Brat/gentoo_update/blob/main/gentoo_update/scripts/updater.sh) 
is a Bash script that executes `emerge` to update the system and generates detailed 
logs for each action performed. Upon successful completion of the updater, the parser 
reads the logs and compiles an update report. The notification sender then dispatches 
this report to users.  

&nbsp; 

### Usage
At the moment `gentoo_update` can only install GLSA and `@world` updates and store 
the output to a dedicated directory. It is available in [GURU](https://wiki.gentoo.org/wiki/Project:GURU) 
overlay in [app-admin/gentoo_update](https://github.com/gentoo/guru/tree/master/app-admin/gentoo_update), 
and in [PyPI](https://pypi.org/project/gentoo-update/). Generally, installing the program 
from GURU overlay is the preferred method, but PyPI will always have the most recent version 
(at the time of writing the newest version is 0.1.6).  

After [enabling GURU overlay](https://wiki.gentoo.org/wiki/Project:GURU/Information_for_End_Users) 
it can be installed via:
```bash
emerge --ask app-admin/gentoo_update
```

Alternatively, it can be installed with pip:
```bash
emerge --ask dev-python/pip
pip install gentoo_update --break-system-packages
```

&nbsp; 

Here are some use cases:  
**Security update**  
Running command without specifying `--update-mode` will use `glsa-check` to install security patches.
```bash
gentoo-update
```

&nbsp; 

**@world update**  
Run full system update, merge all new configuration files, restart all services that were 
updated and display elogs:
```bash
gentoo-update --update-mode full --config-update-mode merge --daemon-restart y --read-logs y
```

Override default behavior and show build logs:
```bash
gentoo-update --update-mode full --args "quiet-build=n"
```

After an update a log file will be created in `/var/log/portage/gentoo_update/log_<timestamp>`.  

&nbsp; 

### Conclusion
`gentoo_update` aims to be a useful tool that will automate and simplify updating Gentoo Linux. 
By default it only installs updates from GLSA, but can also be used to update `@world`, and it 
can be installed from GURU or PyPI.  

I would love to receive some feedback and/or suggestions for this project, feel free to reach out 
to me via 
[Github](https://github.com/Lab-Brat/gentoo_update/tree/main), 
[email](mailto:stepan_kk@pm.me) or IRC (LabBrat).

