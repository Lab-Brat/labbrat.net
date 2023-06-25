---
title: "GSoC 2023: Week 4"
date: 2023-06-23T22:57:16+03:00
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


### Progress on Week 4
Started the week by discovering that my updates to ebuild were 
[not accepted](https://github.com/gentoo/guru/commit/bfffbe1a4bcd10a5e6a20d3ef314ac31cd00641f#comments) 
in the GURU overlay. The issue arose due to a misuse of USE flags feature in the ebuild. 
Maintainers of GURU (big thanks to **antecrescent**!) pointed out my mistake and explained how to fix it, 
which I did by submitting 2 more commits (
[commit1](https://github.com/gentoo/guru/commit/bc46fa1c58c1d493892d6fb339b34f58636b3846) 
and 
[commit2](https://github.com/gentoo/guru/commit/ee6e79850b9189da680fbfdc091ab355574f9180)).  

Then I proceeded to write an introductory blog post. It can currently be read in 
[Gentoo GSoC blog](https://blogs.gentoo.org/gsoc/2023/06/25/gentoo_update-introduction/). 
I've delayed posting about it on forums because I was waiting for the newest ebuild version to 
be merged to the main branch in GURU overlay (and because I was a bit anxious to be honest 😰). 
But in the end I decided to stop waiting, and just mentioned in the blog post that `gentoo_update` 
can be also installed via pip.  
Forum post can be found 
[here](https://forums.gentoo.org/viewtopic-p-8793590.html#8793590).  

Updater also received some improvements overall. I found errors in `--args` flag (used for passing custom 
parameters to `emerge`), in some cases it was not reading all parameters correctly. To fix the problem I 
changed the input type, now it receives a string of space separated parameters instead of a list, 
for example `"quiet-build=y color=y"`, and the problem was fixed.  

Also the packaging with Python's setuptools was improved, now there are no warnings during wheel building.  

Finally, I started working on the parser. Right now it can only split the output to different categories.  

Overall, the week was not a very productive one, but many bugs and imperfections were discovered and fixed 
which is great because I can now focus on the parser!  


### Challenges
It was a bit challenging to understand the reason why USE flags were not a good solutions in my case, 
but after I got it it suddenly became obvious 🤓  

I used USE flags to install optional dependencies for the updater. 
However, USE flags are typically meant to guide Portage in the program's build 
and compilation processes, and in this case USE flags don't change the outcome of how the program is built. 
If these flags are ever removed, it would trigger an unnecessary recompilation of the updater. 
The proper management of optional dependencies, as 
[recommended](https://github.com/gentoo/guru/commit/bfffbe1a4bcd10a5e6a20d3ef314ac31cd00641f#comments) 
by antecrescent, involves using the 
[optfeature eclass](https://devmanual.gentoo.org/eclass-reference/optfeature.eclass/index.html). 
This approach provides users with dependency information and prompts them to consider installing dependencies 
by themselves.  

Then it was a bit tricky to get rid of warnings from setuptools (it feels like I'm struggling with setuptools 
every week 😔). Warning were saying that `updater.sh` and even `tests` directories were treated as Python 
packages, which was a problem because `update.sh` is a Bash script, and `tests` contains scripts and Docker 
compose file used for tests, and both of them were not meant to be a package. I found a solution in Gentoo's 
[Python Guide](https://projects.gentoo.org/python/guide/qawarn.html#stray-top-level-files-in-site-packages) 
which suggested a proper way to exclude the packages to avoid issues with Portage.  


### Plans for Week 5
Mostly I plan to work on the parser the whole week. Here is the checklist from last week:
- [x] Split log into multiple sections, i.e updated programs, what needs restarting etc.
- [ ] Summarize sections and create a report
- [ ] If updater exits with error, crate a separate error report    

