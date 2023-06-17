---
title: "GSoC 2023: Week 3"
date: 2023-06-16T08:21:05+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **week 3** of GSoC.  

Project is hosted on [Github](https://github.com/Lab-Brat/gentoo_update), 
blog post can be alson found on 
[Gentoo Blogs](https://blogs.gentoo.org/gsoc/2023/06/17/week-3-report-automated-gentoo-system-updater/)  


### Progress on Week 3
`gentoo_update` finally received some Github stars!  

It also has received 2 issues ([#7](https://github.com/Lab-Brat/gentoo_update/issues/7) 
and [#8](https://github.com/Lab-Brat/gentoo_update/issues/8)). In `#7` someone suggested 
to remove `update.sh` from being installed in the PATH, and only expose `gentoo-update` 
as entry point.`#8` was a question if the program will be resolving circular dependencies. 
I was more than happy to solve/answer both issues, and hope to see more in near future!  

ebuild that I have submitted to GURU repository apparently didn't pass some CI tests and 
generated a bunch of errors which I received by email. Luckily for me, some nice maintainers 
found and fixed the issues, and those problems are solved now. More details in bugs 
[908307](https://bugs.gentoo.org/908307) and [908308](https://bugs.gentoo.org/908308). 
I also received some recommendations about how to avoid similar issues in future from 
the GURU's IRC chat.  

I have further improved the updater program code, here is the changelog:
* remove `updater.sh` from PATH
* Read PORTAGE_LOGDIR variable from make.conf and use it to store logs. 
  By default it will use `/var/log/portage/gentoo_update`
* Before running `needrestart`, `eclean` or `revdep-rebuild` check if it's installed, print 
  error message if not
* Add type hints to Python functions and methods
* Change `set -e` to `set -euo pipefail` to improve stability of Bash scripts  

ebuild also received some improvements. Now it defines 2 
[USE flags](https://github.com/Lab-Brat/gentoo_update_ebuild/blob/main/gentoo_update-0.1.5.ebuild#LL16C1-L21C2) 
that will install optional dependencies.  

Lastly, I started coding the parser. Parser's end goal is to read the log file created by the 
updater and to create an easy-to-read report that will briefly describe what changes were 
made on the system.  

Oh, and I finally learned how to post blogs on https://blogs.gentoo.org/gsoc, it wasn't that 
hard after all.  


### Challenges
I found it very hard to debug ebuild issues, the 
[documentation](https://devmanual.gentoo.org/ebuild-writing/index.html) 
is actually very helpful but it's also gigantic a takes much time to go through. Thankfully 
GURU team came through to solve the issues I was having.  

Apart from that, I am thinking on ways to automate version bumping the updater in GURU repository. 
Here are the steps I am taking now to version bump the ebuild:
* Push a tag in the Github repository
* Sync GURU overlay
* Modify ebuild, usually just version bump, but this week I also added USE flags
* Run test with -> `ebuild gentoo_update-0.1.5.ebuild test`
* Update Manifest if tests were passed -> `ebuild gentoo_update-0.1.5.ebuild manifest`
* Commit changes to dev branch

I think all of these steps can be automated with Github Actions.  


### Plans for Week 4
The first thing to do is to write a post on Gentoo Forums about the updater. This task is 
already a bit delayed because of technical issues with the updater and the ebuild, but now 
everything is ready 🦾😊  

To maximize the usefulness of `gentoo_update` it's very important to get some feedback from 
community as soon as possible, and it will also be nice to have some more Github issues to 
work on. I'm planning to post on forums before next Tuesday.  

Then, there of course is the parser. I plan to add following features:
* Split log into multiple sections, i.e updated programs, what needs restarting etc.
* Summarize sections and create a report
* If updater exits with error, crate a separate error report    

If there will be free time left, it would be great to work on Github Actions workflow to 
automate version bumping the ebuild.  

