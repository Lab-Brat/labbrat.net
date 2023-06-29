---
title: "GSoC 2023: Week 5 [Wednesday Update]"
date: 2023-06-29T21:55:14+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **week 4** of GSoC.  

Project is hosted on [Github](https://github.com/Lab-Brat/gentoo_update).  


### Progress on Week 5
Week started off be receiving some feedback from the community in the 
[forums](https://forums.gentoo.org/viewtopic-p-8793590.html#8793590). Here are nice ideas 
that I thought will be good to implement:
1. Fallback to the latest version of the package if an error is encountered during an update;
2. Add an option to control Portage niceness;
3. Estimate update time;
4. Notify users about obsolete USE flags;
5. Think of a way to make updater work on `binpkg` servers.

I will attempt to do 1-4 in the duration of Google Summer of Code.

### Challenges
I am stuck at the parser.

