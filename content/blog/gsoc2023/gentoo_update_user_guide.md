---
title: "gentoo_update User Guide"
date: 2023-08-26T09:54:05+03:00
tags:
    - opensource
    - gsoc
categories:
    - GSoC2023
ShowToc: true
---

## Introduction
This article will go through the basic usage of `gentoo_update` CLI 
tool and the mobile app.  

But before that, here is a demo of this project: {{< youtube go6SJZBgpgg >}}

&nbsp; 

## gentoo_update CLI App
### Installation
`gentoo_update` is available in GURU overlay and in PyPI. 
Generally, installing the program from GURU overlay is the preferred method, 
but PyPI will always have the most recent version.  

Enable GURU and install with emerge:
```bash
eselect repository enable guru
emerge --ask app-admin/gentoo_update
```

Alternatively, install from PyPI with pip:
```bash
python -m venv .venv_gentoo_update
source .venv_gentoo_update/bin/activate
python -m pip install gentoo_update
```

### Update 
`gentoo_update` provides 2 update modes - full and security. Full mode updates @world, 
and security mode uses glsa-check to find security patches, and installs them if something 
is found.  

By default, when run without flags security mode is selected:
```bash
gentoo-update
```

To update @world, run:
```bash
gentoo-update --update-mode full
```

Full list of available parameters and flags can be accessed with the `--help` flag. 
Further examples are detailed in the repository's 
[readme file](https://github.com/Lab-Brat/gentoo_update).  

Once the update concludes, a log file gets generated at 
`/var/log/portage/gentoo_update/log_<date>` (or whatever $PORTAGE_LOGDIR is set to). 
This log becomes the basis for the update report when the `--report` flag is used, 
transforming the log details into a structured update report.  

### Send Report
The update report can be sent through three distinct methods: IRC bot, email, or mobile app.

**IRC Bot Method**  
Begin by registering a user on an IRC server and setting a nickname as outlined in the 
[documentation](https://libera.chat/guides/registration). 
After establishing a chat channel for notifications, 
define the necessary environmental variables and execute the following commands:
```bash
export IRC_CHANNEL="#<irc_channel_name>"
export IRC_BOT_NICKNAME="<bot_name>"
export IRC_BOT_PASSWORD="<bot_password>"
gentoo-update --send-report irc
```

**Email via Sendgrid**  
To utilize Sendgrid, register for an account and generate an 
[API key](https://docs.sendgrid.com/ui/account-and-settings/api-keys)). 
After installing the Sendgrid Python library from GURU, 
save the API key in the environmental variables and use the commands below:
```bash
emerge --ask dev-python/sendgrid
export SENDGRID_TO='recipient@email.com'
export SENDGRID_FROM='sender@email.com'
export SENDGRID_API_KEY='SG.****************'
gentoo-update --send-report email
```

Notifications can also be sent via the mobile app. 
Details on this method will be elaborated in the following section.

&nbsp; 

## gentoo_update Mobile App
### Installation
Mobile app can either be installed from Github or Google Play Store.  

**Note:** As of 2023.08.26, the app is only available for Android. 
The app is also pending approval in the Google Play Store, which might take up to two weeks. Until it get approved, 
see below for manual installation instruction.  

**Manual Installation**  
For manual installation on an Android device, download the APK file from  
[Releases](https://github.com/Lab-Brat/gentoo_update_flutter/releases/tag/1.0.1) 
tab on Github. Ensure you've enabled installation from 
[Unknown Sources](https://www.applivery.com/docs/mobile-app-distribution/android-unknown-sources/) 
before proceeding.  

### Usage
The mobile app consists of three screens: Login, Reports, and Profile.

Upon first use, users will see the Login screen. 
To proceed, select the Anonymous Login button. 
This action generates an account with a unique user ID and token, 
essential for the CLI to send reports.  

The Reports screen displays all reports sent using a specific token. 
Each entry shows the update status and report ID. 
For an in-depth view of any report, simply tap on it.  

On the Profile screen, users can find their 8-character token, 
which needs to be saved as the GU_TOKEN variable on the Gentoo instance. 
This screen also shows the AES key status, crucial for decrypting the 
client-side token as it's encrypted in the database. 
To log out, tap the Sign Out button.  
**Note:** Since only Anonymous Login is available, once logged out, 
returning to the same account isn't possible.  

![Gentoo Update Mobile App Screens](/img/gentoo_update_mb_screens.png/) 

&nbsp; 

## Contacts
Preferred method for getting help or requesting a new feature for both CLI 
and mobile apps is by creating an issue in Github:
* gentoo_update CLI [issues page](https://github.com/Lab-Brat/gentoo_update/issues)
* Mobile app [issues page](https://github.com/Lab-Brat/gentoo_update_flutter/issues)

Or just contact me directly via [labbrat_social@pm.me](mailto:labbrat_social@pm.me) and 
IRC. I am in most of the #gentoo IRC groups and my nick is #LabBrat.  

&nbsp; 

## Links
- [[Link](https://github.com/Lab-Brat/gentoo_update)] - gentoo_update CLI repository
* [[Link](https://github.com/Lab-Brat/gentoo_update_flutter)] - Mobile App repository
