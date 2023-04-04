---
title: "Gentoo: Upgrading the System"
date: 2023-03-18T15:01:59+03:00
tags:
    - Gentoo
    - Linux
    - Portage
categories:
    - Gentoo
showToc: true
---

Gentoo Linux, being a true meta-distribution, give users maximum flexibility and control 
over the system. A stark example of this is the OS upgrade process. Users have a large 
choice of different command utilities and a bunch of configuration option to choose from 
to tailor the upgrade process to their needs. This guide will attempt to combine and 
distill the best practices and recommendations from the Gentoo Wiki/Forums and other 
sources into a single guide.  
**TLDR**:
```bash
emaint sync -a
emerge --ask --verbose --update --newuse --deep @world
dispatch-conf
emerge --ask --depclean
revdep-rebuild
eclean -d distfiles
```

### Preqrequisites
**@world**  
The `@world` set is a special set that contains all the packages that are installed on the system. It is used to update the system.  
I did an overview of sets in [Gentoo World Set](https://labbrat.net/blog/gentoo_sets/).

**Use flags**  
Use flags are used to control the compilation of packages. They are used to enable or disable certain features of a package. For example, the `X` use flag is used to enable or disable the X11 support in a package.  
I did an overview of use flags in [Gentoo USE Flags](https://labbrat.net/blog/gentoo_use_flags/).


**Command-line tools that might be used during the upgrade**
Tecnhically `emerge` can do all the basic steps, but there are other tools that might be useful:
* `eix` - search for packages
* `equery` - query information about packages
* `emaint` - maintenance tool for Portage
* `euse` - manage USE flags
* `etc-update` - update configuration files (old program)
* `dispatch-conf` - update configuration files
* `eselect` - manage system configuration
* `elogv` - view the latest entries in the Portage logs
* `needrestart` - check if a restart is needed after update
* `eclean` - clean up obsolete files
* `eclean-kernel` - clean up old kernels
* `qcheck` - check for broken packages
* `revdep-rebuild` - rebuild packages that depend on a package that has been updated
* `glsa-check` - check Gentoo Linux Security Advisory for security updates
* `layman` - manage overlays

### Step 1: Sync Portage Tree
The first step is to sync the Portage tree. This is done by running the following command:
```bash
emaint --auto sync
```  

It downloads the latest version of the Portage tree and the metadata from the Gentoo mirrors. 

### Step 2: Update the System
There are many ways to update the system. 
To do the most basic full update run:
```bash
emerge --ask --verbose --update --newuse --deep @world
```  

Flags:
* `--ask` - ask before performing the action
* `--verbose` - show more information
* `--update` - update to the best available version (not neweset)
* `--newuse` - include installed packages where USE flags have changed since compilation
* `--deep` - update entire dependency tree, even libraries that are not directly listed in the dependencies of a package

Other options:
* `--keep-going` - continue updating even if a package fails to update
* `--oneshot` - install package without adding it to the world file

### Step 3: Update the Configuration Files
Sometimes after an update configuration files might change. When this happens, Portage create a new config at 
`/etc/portage/config/._cfg0000_etc_file_name_` and leaves the old config at `/etc/file_name`. 
`dispatch-conf` can be used to interactively update the configuration files.  

### Step 4: Clean up obsolete packages
After multiple updates, some packages might become obsolete (stop being other package's dependencies). 
To clean them up run:
```bash
emerge --ask --depclean
```

### Step 5: Verify system consistency after cleanup
In some rare case libraries and dependencies might not be installed or be broken. 
To re-emerge them run:
```bash
revdep-rebuild
```

### Step 6: Clean up source code
After multiple updates, some package source files might become obosolete. A common example is Linux kernel source. 
To clean them up run:
```bash
emerge --ask --verbose --depclean
```

### Step 7: Additional (optional) steps
Update Portage itself when needed:
```bash
emerge -1v portage
```

Read the news:
```bash
eselect news read
```

Read update information from logs:
```bash
elogv
```

Check if a restart is needed:
```bash
needrestart
```

Remove old kernels:
```bash
eclean-kernel -n 1
```  

### References
* Gentoo Wiki
  * [ [Link](https://wiki.gentoo.org/wiki/Upgrading_the_system) ] Upgrading the System
* Gentoo Forums
  * [ [Link](https://forums.gentoo.org/viewtopic-t-807345-start-0.html) ] seven steps to upgrade Gentoo System
  * [ [Link](https://forums.gentoo.org/viewtopic-t-1162086-highlight-safe.html)  ] oneshot - need clarification 
  * [ [Link](https://forums.gentoo.org/viewtopic-t-1064844-start-0.html) ] glsa-check
* Reddit
  * [ [Link](https://www.reddit.com/r/Gentoo/comments/2f3v8f/how_to_upgrade_gentoo_without_breaking_anything/) ] How to upgrade Gentoo without breaking anything?
* Fitzcarraldo's Blog
  *  [ [Link](https://fitzcarraldoblog.wordpress.com/2020/03/07/my-%EF%BB%BFsystem-upgrade-procedure-for-gentoo-linux/) ] My system upgrade procedure for Gentoo Linux
  *  [ [Link](https://fitzcarraldoblog.wordpress.com/2019/04/07/automatically-clearing-the-usr-tmp-portage-directory-in-gentoo-linux/) ] Automatically clearing the /usr/tmp/portage directory in Gentoo Linux
* Stack
  * [ [Link](https://serverfault.com/questions/9936/optimal-procedure-to-upgrade-gentoo-linux) ] Optimal procedure to upgrade Gentoo Linux?
* Github
  * [ [Link] ](https://github.com/sakaki-/genup) genup
  * [ [Link] ](https://github.com/alicela1n/gentoo-update) gentoo-update
 