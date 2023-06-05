---
title: "GSoC 2023: Week 1"
date: 2023-06-04T09:21:52+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **week 1** of GSoC.  

Project is hosted on [Github](https://github.com/Lab-Brat/gentoo_update)  


### Progress on Week 1
The most basic version of the updater program is ready. 
By default it only installs security patches from GLSA using 
`glsa-check`, but it also allows users to update `@world` 
with their custom update flags. Additionally, after an update 
users can choose to:
* merge changed configuration files
* restart updated services
* do a post-update clean up
* read elogs
* read news

After an update a log file is created in `/var/log/gentoo_update/log_$timestamp`. 
This file will be used at later stages for parsing and notification sending.  

Code was tested in custom Gentoo Linux stage3 containers. 
The environment I used is defined by a Docker Compose file in 
[tests/compose.yaml](https://github.com/Lab-Brat/gentoo_update/blob/main/tests/compose.yaml) 
in the repository. It was also tested on a VM and a old 
Acer Swift laptop :)  


### Challenges
It's actually very tricky to run Bash scripts through Python. 
I used `subprocess` library, it has tools for splitting the output 
stream very precisely. Splitting `stdout` and `stderr` and processing 
it separately will make parsing logs much easier because Bash already 
decided which output contains errors and which not.  

I also found packaging very tricky. This project so far has 1 main 
Python and 1  Bash script, and it's not very clean how to bundle it 
together correctly. I defined everything in `setup.cfg` and created a 
distribution which was uploaded to PyPi. However, to my dismay, I 
discovered that something has changed in how `pip` installs packages 
on the system. Now you will not be able to install anything without 
`--break-system-packages` flag:
```
08cf39cb61f9 / # pip install gentoo_update --break-system-packages
Collecting gentoo_update
  Downloading gentoo_update-0.1-py3-none-any.whl (6.9 kB)
Installing collected packages: gentoo_update
Successfully installed gentoo_update-0.1
```

I get the point, `pip` does have too much authority as a secondary 
package manager, so it was probably done for security reasons.  

Furthermore, there are some errors in path definition and Bash 
script is getting lost along the way somewhere:
```console
08cf39cb61f9 / # gentoo-update       
[05-Jun-23 19:07:16 ERROR] ::: sh: /usr/lib/python3.11/site-packages/updater.sh: No such file or directory
[05-Jun-23 19:07:16 ERROR] ::: /usr/lib/python3.11/site-packages/updater.sh exited with error code {script_stream.returncode}
Standard error output:
sh: /usr/lib/python3.11/site-packages/updater.sh: No such file or directory
```

And I haven't even gotten to the `ebuild` part yet....


### Plans for Week 2
During the second week I plan to fix all issues with packaging and 
create an `ebuild` to avoid eery error messages from `pip`.  

When `gentoo-update` will be packaged decently I plan to do some 
minor fixes and do more tests, and ideally by the end of the week 
write a blog post in Gentoo forums announcing my project. It would 
be nice to receive feedback from the community early as possible.  

I will start to work on the parser from week 3.

