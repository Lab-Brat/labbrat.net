---
title: "GSoC 2023: Week 10"
date: 2023-08-06T22:58:38+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

This article is a summary of all the changes made on 
[Automated Gentoo System Updater](https://wiki.gentoo.org/wiki/Google_Summer_of_Code/2023/Ideas/Automated_Gentoo_system_updater) 
project during **week 10** of GSoC.  

Project is hosted on GitHub (
[gentoo_update](https://github.com/Lab-Brat/gentoo_update) and 
[mobile app](https://github.com/Lab-Brat/gentoo_update_flutter)), 
blog post can be also found on 
[Gentoo Blogs]().


### Progress on Week 10
I have finalized app architecture, here are the details:  

The app's main functionality is to receive notification from the push server. 
For each user, it will create a unique API token after authentication 
(there is an Anonymous option). This token will be used by gentoo_update to send 
the encrypted report to the mobile device using a push server endpoint. Update reports 
will be kept only on the mobile device, ensuring privacy.  

After much discussion, I decided to implement app’s backend in Firebase. Since GSoC 
is organized by Google, it seems appropriate to use their products for this project. 
However, future plans include the possibility of implementing a self-hosted web server, 
so that instead of authentication user will just enter server public IP and port.  

Example usage will be something like:
1. Download the app and sign-in.
2. App will generate a token, 1 token per 1 account.
3. Save the token into an environmental variable on Gentoo Linux.
4. Run `gentoo_update --send-report mobile`
5. Wait until notification arrives on the mobile app.

I have also made some progress on the app code. I've decided to host it in 
another [repository](https://github.com/Lab-Brat/gentoo_update_flutter) because it 
doesn't require direct access to `gentoo_update`, and this way it will be easier to 
manage versions and set up CI/CD.  

Splitting tasks for the app into `UI` and `Backend` categories was not very 
efficient in practice, since two are very closely related. Here is what I have done 
so far:
* Create an app layout
* Set up Firebase backend for the app
* Set up database structure for storing tokens
* Configure anonymous authentication
* UI elements for everything above

### Challenges
I'm finding it somewhat challenging to master the use of Flutter and design an 
modern-looking app. My comfort zone lies more in coding backend and automation tasks 
rather than focusing on the intricacies of UI components. Despite these challenges, 
I am 60% sure that in end app will look half-decent.  

### Plans for Week 11
After week 11 I plan to have a mechanism to deliver update reports from a 
Gentoo Linux machine.
