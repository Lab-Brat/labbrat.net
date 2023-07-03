---
title: "GSoC 2023: Week 5"
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
project during **week 5** of GSoC.  

Project is hosted on [GitHub](https://github.com/Lab-Brat/gentoo_update), 
blog post can be also found on 
[Gentoo Blogs](https://blogs.gentoo.org/gsoc/2023/06/17/week-3-report-automated-gentoo-system-updater/).    


### Progress on Week 5
Week started off by receiving some feedback from the community in the 
[forums](https://forums.gentoo.org/viewtopic-p-8793590.html#8793590). Here are some nice ideas 
that community have suggested to implement:
1. Fallback to the latest version of the package if an error is encountered during an update;
2. Add an option to control Portage niceness;
3. Estimate update time;
4. Notify users about obsolete USE flags;
5. Think of a way to make updater work on `binpkg` servers.

I will attempt to do 1-4 in the duration of Google Summer of Code.  

There were also some suggestions on improving the workflow and many different opinions were voiced. 
The discussion is still ongoing, but it has already yielded some positive results.  

I've made some progress on the Parser, it can now detect whether update has ended in an error or not. 
Log format and general output flow was modified to simplify parsing. Most noticeable change was 
the way how the `updater.sh` is launched. Before the whole script (~250 lines of Bash) were 
launched all at once, and now each function from the script is being launched separately. Additional 
flag (`--report`) was added to utilize the parser, it can now parse the last log from the log directory.  

Furthermore, I spent sometime on organizing testing a bit better. I updated container versions and 
created a better naming convention for my 
[containers](https://github.com/Lab-Brat/gentoo_dockerfiles) to not get lost in them. 08-05-2023 
Desktop image on openrc is being used to test `glsa-check`, and most recent openrc basic image is 
used to test updating functionality.  


### Challenges
Parsed has turned out to be much harder than I anticipated. First of all, I had to make some changes 
to both Python and Bash code to create simpler log output (reduce number of if/else statements), and 
secondly there were some motivation issues.  

Secondly, there were some motivation issues. It was a bit hard to focus on the parser, because a much 
better approach is to add machine readable output from Portage instead of parsing logs. I talked to my 
mentor about it and we decided to continue working on the parser, mainly because modifying Portage in 
any significant way take waay too much time.  


### Plans for Week 6
On week 6 the plan is to add error parsing and comprehension to the parser. This means I will have to 
find some different ways to cause Portage to break, and then try to make parser understand the errors 
that have occurred.  

After that is done, I can focus on using this information to create nice-looking update reports. 

