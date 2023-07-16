---
title: "GSoC 2023: Weeks 6 + 7"
date: 2023-07-10T10:13:15+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **weeks 6 and 7** of GSoC.  

Project is hosted on [GitHub](https://github.com/Lab-Brat/gentoo_update), 
blog post can be also found on 
[Gentoo Blogs](https://blogs.gentoo.org/gsoc/2023/07/16/week-67-report-automated-gentoo-system-updater/).    


### Progress on Weeks 6 + 7
These 2 weeks were spent on the parser and the reporter. 
During this time, I've added many features to it, but there 
are still much more things left to be done. Due to limited 
time of GSoC I will implement additional features after the 
program end.  

Here is a list of features that were implemented so far:
* If the update was successful, report will show:
    * updated package names
    * package versions in the format "old -> new"
    * USE flags of those packages
    * disk usage before and after the update
* If the emerge pretend has failed, report will show:
    * error type (for now only supports 'blocked packages' error)
    * error details (for blocked package it will show problematic packages)

And here are the errors that I plan to add support for in the future:
* Mutually exclusive USE flags
* Errors due to licenses
* OOM
* Not enough disk space
* Network issues during the update  

I also had a good idea about how to go about testing `gentoo_update`. 
Basically, I can set up a CI/CD pipeline that will detect newly published 
stage3 Docker containers, and whenever there a new container is detected run `gentoo_update` on it and check the output. Eventually, it will run 
into some errors that I will then use to improve `gentoo_update`. Pipeline itself can be set up with Jenkins, for example. This idea is a bit out of 
scope of my proposal, so I will work on it after GSoC :)  

### Challenges
While trying to find ways to generate errors in Portage I realized how hard 
it is to break Portage intentionally, and it's almost impossible without 
deliberately creating faulty ebuilds and USE flags, which of course is a 
good thing!  

So far I only managed to test out 'blocked package' error, 
here is how it was done:
* Create a simple [Bash script](https://github.com/Lab-Brat/gentoo_loctests) that prints out some ASCII art (prints an owl, in my case);
* Set up a local repository, and add an ebuild for this script;
* Install the script on the system;
* Version bump the script, for example 0.1 -> 0.2;
* Then add `RDEPEND="!net-print/cups"` to the ebuild, which will raise an error if cups is installed. cups is just a package that was installed on my system, any other package will do;
* Run update @world and look how Portage starts to throw errors :)  

What sounds like a couple simple steps actually took me about 2 days to 
figure out... Although challenging, it actually is very fun to find ways 
to break things :)  


### Plans for Week 8
Week 8 will be dedicated to writing code to send reports via emails 
and IRC chats. But before that, I need to do some more work to improve 
integration between the updater, parser and reporter.  

Ideally, I also need to spend some more time on error catching and 
improving overall stability of `gentoo_update`. 

