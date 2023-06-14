---
title: "GSoC 2023: Week 3 [Wednesday update]"
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
`gentoo_update` finally received some stars!  

It also has received 2 issues ([#7](https://github.com/Lab-Brat/gentoo_update/issues/7) 
and [#8](https://github.com/Lab-Brat/gentoo_update/issues/8)). In `#7` some nice people, 
presumably GURU maintainers, were fixing my ebuild errors and asked to remove `update.sh` 
from being installed in the PATH, and only expose `gentoo-update` as entry point. `#8` 
was a question if the program will be resolving circular dependencies. I was more than 
happy to solve/answer both issues, and hope to see more in near future!  

Plans for the rest of the week:
1. remove `needrestart` as a mandatory dependency
2. do small fixes from pull request comments
3. post a blog post about `gentoo_update` on Gentoo forums
4. start working on the parser


### Challenges


### Plans for Week 4

