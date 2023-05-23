---
title: "Gentoo Package Sets"
date: 2023-03-19T13:54:45+03:00
tags:
    - gentoo
    - linux
    - portage
categories:
    - Gentoo
showToc: true
---

Gentoo's package manager Portage has an organizational feature called sets. 
A set is essentially a named list of packages that you can use to install 
or update multiple packages at once. There are predefined system sets 
like `@world` that contain all packages installed in the system, and it's 
also possible to create custom ones, for example for a specific application. 
This is a very useful feature, because it allows users to easily install 
uninstall and upgrade packages.  

To see all available sets, run:
```bash
emerge --list sets
```  

To actually see what's insie a set, run:
```bash
emerge --pretend --verbose @system
```  

Sets are used in the same way as normal package by emerge. To install all 
packages in a set, run:
```bash
emerge --ask --verbose @system
```  

Custom sets can be created by adding a file to `/etc/portage/sets/` with
the name of the set and the list of packages to be included. For example 
let's create `@dekstop_env` set to manage desktop environment packages:
```
#/etc/portage/sets/dekstop_env
x11-base/xorg-drivers
x11-base/xorg-server
x11-wm/qtile
```

Furthermore, there are also compound sets, which are sets that contain 
other sets. For example, `@world` is a compound set that contains `@system` 
and `@selected` sets, and dynamic sets, which are sets that are generated 
based on certain criteria, such as matching a specific keyword or USE flag. 
For example `dev-python/*:openssl` includes all packages in `dev-python` 
category that has the openssl USE flag enabled. 

With sets installing and upgrading packages becomes very easy becuase 
they can be split in categories and managed separately. Some useful 
categories might be: `@system`, `@desktop_env`, `@networking`, `@office`, 
`@multimedia`, `@games`, `@fonts`, `@development`, `@virtualization`, 
`@security`, `@hardware`, `@misc`, `@local` etc.
