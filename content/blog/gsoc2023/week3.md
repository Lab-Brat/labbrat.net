---
title: "GSoC 2023: Week 3 [Friday update]"
date: 2023-06-14T08:10:05+03:00
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

Project is hosted on [Github](https://github.com/Lab-Brat/gentoo_update)  


### Progress on Week 3
`gentoo_update` finally received some Github stars!  

It also has received 2 issues ([#7](https://github.com/Lab-Brat/gentoo_update/issues/7) 
and [#8](https://github.com/Lab-Brat/gentoo_update/issues/8)). In `#7` some nice people, 
presumably GURU maintainers, were fixing my ebuild errors and asked to remove `update.sh` 
from being installed in the PATH, and only expose `gentoo-update` as entry point. `#8` 
was a question if the program will be resolving circular dependencies. I was more than 
happy to solve/answer both issues, and hope to see more in near future!  

ebuild that I have submitted to GURU repository apparently didn't pass some CI tests and 
generated a bunch of error which I received by email. Luckily for me, some nice maintainers 
found and fixed the issues, and those problems are solved now. More details in bugs 
[908307](https://bugs.gentoo.org/908307) and [908308](https://bugs.gentoo.org/908308).

Plans for the rest of the week:
1. post a blog post about `gentoo_update` on Gentoo forums
2. start working on the parser
3. add installing needrestart with a USE flag.


### Challenges


### Plans for Week 4

