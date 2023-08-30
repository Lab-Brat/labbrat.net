---
title: "Final GSoC Report"
date: 2023-08-27T15:14:08+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

## Project Goals
Main goal of the project was to write an app that will automatically handle updates on 
Gentoo Linux systems and send notifications with update summaries. 
More specifically, I wanted to:
1. Simplify the update process for beginners, offering a safer and more intuitive method.
2. Minimize time experienced users expend on routine update tasks, decreasing their workload.
3. Ensure systems remain secure and regularly updated with minimal manual intervention.
4. Keep users informed of the updates and changes.
5. Improve the overall Gentoo Linux user experience.

## Progress
Here is a summary of what was done every week with links to my blog posts.

### [Week 1](https://labbrat.net/blog/gsoc2023/week1/)
Basic system updater is ready. Available functionality:
* update security patches
* update @world
* merge changed configuration files
* restart updated services
* do a post-update clean up
* read elogs
* read news  

Also prepared a Docker Compose file to run tests in containers.  

Links:
* Pull requests: [#2](https://github.com/Lab-Brat/gentoo_update/pull/2), [#3](https://github.com/Lab-Brat/gentoo_update/pull/3), [#4](https://github.com/Lab-Brat/gentoo_update/pull/4)
* Docker [tests](https://github.com/Lab-Brat/gentoo_update/blob/main/tests/compose.yaml)  

### [Week 2](https://labbrat.net/blog/gsoc2023/week2/)
Packaged Python code, created an ebuild and a GitHub Actions workflow 
that publishes package to PyPI when commit is tagged.  

Links:
* Pull requests: [#5](https://github.com/Lab-Brat/gentoo_update/pull/5) 
* ebuild [commit](https://github.com/Lab-Brat/gentoo_update_ebuild)
* GitHub Actions [workflow](https://github.com/Lab-Brat/gentoo_update/blob/main/.github/workflows/main.yml)  

### [Week 3](https://blogs.gentoo.org/gsoc/2023/06/17/week-3-report-automated-gentoo-system-updater/)
Fixed issue #7 and answered to issue #8 and fixed bug 908308. Added USE flags to 
manage dependencies. Improve Bash code stability.

Links:
* Issues: [#7](https://github.com/Lab-Brat/gentoo_update/issues/7), [#8](https://github.com/Lab-Brat/gentoo_update/issues/8)
* Bugs: [908308](https://bugs.gentoo.org/908308)
* Pull requests: [#6](https://github.com/Lab-Brat/gentoo_update/pull/6), [#9](https://github.com/Lab-Brat/gentoo_update/pull/9), [#10](https://github.com/Lab-Brat/gentoo_update/pull/10) 

### [Week 4](https://blogs.gentoo.org/gsoc/2023/06/25/week-4-report-automated-gentoo-system-updater/)
Fixed errors in ebuild, replaced USE flags with optfeature for dependency management. 
Wrote a blog post to introduce my app and posted it on forums. 
Fixed a bug in `--args` flag.

Links:
* Pull requests: [#11](https://github.com/Lab-Brat/gentoo_update/pull/11)
* [request](https://github.com/gentoo/guru/commit/bfffbe1a4bcd10a5e6a20d3ef314ac31cd00641f) to fix ebuild
* Blog [post](https://blogs.gentoo.org/gsoc/2023/06/25/gentoo_update-introduction/) and Forum [post](https://forums.gentoo.org/viewtopic-p-8793590.html#8793590)

### [Week 5](https://blogs.gentoo.org/gsoc/2023/07/02/week-5-report-automated-gentoo-system-updater/)
Received some feedback from forums. Coded much of the parser (`--report`). 
Improved container testing environment.

Links:
* Improved [dockerfiles](https://github.com/lab-Brat/gentoo_dockerfiles)

### [Weeks 6 and 7](https://blogs.gentoo.org/gsoc/2023/07/16/week-67-report-automated-gentoo-system-updater/)
Completed parser (`--report`). Available functionality:
* If the update was successful, report will show:
    * updated package names
    * package versions in the format “old -> new”
    * USE flags of those packages
    * disk usage before and after the update
* If the emerge pretend has failed, report will show:
    * error type (for now only supports ‘blocked packages’ error)
    * error details (for blocked package it will show problematic packages)

Also added disk usage calculation before and after the update.  

Links:
* Pull requests: [#12](https://github.com/Lab-Brat/gentoo_update/pull/12), [#13](https://github.com/Lab-Brat/gentoo_update/pull/13)

### [Week 8](https://blogs.gentoo.org/gsoc/2023/07/23/week-8-report-automated-gentoo-system-updater/)
Add 2 notification methods (`--send-reports`) - irc bot and emails via sendgrid.

Links:
* Pull requests: [#14](https://github.com/Lab-Brat/gentoo_update/pull/14), [#15](https://github.com/Lab-Brat/gentoo_update/pull/15)

### [Week 9-10](https://blogs.gentoo.org/gsoc/2023/08/07/week-910-report-automated-gentoo-system-updater/)
Improved CLI argument handling. 
Experimented with different mobile app UI layouts and backend options. 
Fixed issue #17. 
Started working on mobile app UI, decided to use Firebase for backend.  

Links:
* Pull requests: [#16](https://github.com/Lab-Brat/gentoo_update/pull/16)
* Issues: [#17](https://github.com/Lab-Brat/gentoo_update/issues/17)

### [Week 11-12](https://blogs.gentoo.org/gsoc/2023/08/20/week-1112-report-automated-gentoo-system-updater/)
Completed mobile app (UI + backend). Available functionality:
* UI
    * Login screen: Anonymous login
    * Reports screen: Receive and view reports send from CLI app.
    * Profile screen: View token, user ID and Sign Out button.
* Backend
    * Create anonymous users (Cloud Functions)
    * Create user tokens (Cloud Functions)
    * Receive tokens in https requests, verify them, and route to users (Cloud Functions)
    * Send push notifications (FCM)
    * Secure database access with Firestore security rules

Created a plan to migrate to a custom backend based on Django+MongoDB+Nginx,
added `--send-reports mobile` option to CLI.

Link:
* Pull requests: [#18](https://github.com/Lab-Brat/gentoo_update/pull/18)
* Mobile app [repository](https://github.com/Lab-Brat/gentoo_update_flutter)

### Final week
Added token encryption with Cloud Functions.  
Packaged mobile app with GitHub Actions and 
published to Google Play Store (2023-08-27 app review is still ongoing).  

Recorded a demo video and 
wrote gentoo_update User Guide that covers both CLI and mobile app.

Links:
* Demo [video](https://youtu.be/go6SJZBgpgg?si=bgC2xLA22_aeikOE)
* [gentoo_update User Guide](https://blogs.gentoo.org/gsoc/2023/08/27/gentoo_update-user-guide/)
* Packaging GitHub Actions [workflow](https://github.com/Lab-Brat/gentoo_update_flutter/blob/main/.github/workflows/main.yml)
* Release [page](https://github.com/Lab-Brat/gentoo_update_flutter/releases)


## Project Status
I would say I'm very satisfied with the current state of the project. 
Almost all tasks were completed from the proposal, and there is a product that 
can already be used.  

To summarize, here is a list of deliverables:
1. [Source code](https://github.com/Lab-Brat/gentoo_update) for gentoo_update CLI app
2. gentoo_update CLI app [ebuild](https://github.com/gentoo/guru/tree/master/app-admin/gentoo_update) in GURU repository
3. gentoo_update CLI app package in [PyPi](https://pypi.org/project/gentoo-update/)
4. [Source code](https://github.com/Lab-Brat/gentoo_update_flutter) for mobile app
5. Mobile app for Andoid in [APK](https://github.com/Lab-Brat/gentoo_update_flutter/releases/tag/1.0.1)
6. Mobile app for Android in [Google Play](https://play.google.com/store/apps/details?id=net.labbrat.gentoo_update)

## Future Improvements
I plan to add a lot more features to both CLI and mobile apps. 
Full feature lists can be found in readme's of both repositories:
* CLI app upcoming [features](https://github.com/Lab-Brat/gentoo_update)
* mobile app upcoming [features](https://github.com/Lab-Brat/gentoo_update_flutter)

## Final Thoughts
These 12 weeks felt like a hackathon, where I had to learn new technologies very 
quickly and create something that works very fast. I faced many challenges and 
acquired a range of new skills.  

Over the course of this project, I coded both Linux CLI applications using Python and Bash, 
and mobile apps with Flutter and Firebase. 
To maintain the quality of my work, I tested the code in Docker containers, virtual machines 
and physical hardware. 
Additionally, I built and deployed CI/CD pipelines with GitHub Actions to automate packaging. 
Beyond the technical side, I engaged actively with Gentoo community, 
utilizing IRC chats and forums. Through these platforms, I addressed and resolved issues on 
both GitHub and Gentoo Bugs, enriching my understanding and refining my skills.

I also would like to thank my mentor, Andrey Falko, for all his help and support. 
I wouldn't have been able to finish this project without his guidance.  

In addition, I want to thank Google for providing such a generous opportunity for 
open source developers to work on bringing forth innovation.   

Lastly, I am grateful to Gentoo community for the feedback that's helped me to 
improve the project immensely. 	

