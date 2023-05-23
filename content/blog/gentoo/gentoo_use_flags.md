---
title: "Gentoo USE Flags"
date: 2023-03-19T13:54:13+03:00
tags:
    - gentoo
    - linux
    - portage
categories:
    - Gentoo
showToc: true
---

USE flags that allow users to customize the way packages 
are built and installed on a system. They are essentially 
a set of optional features that can be enabled or disabled 
for each package, depending on the specific needs and preferences.  

For example, some packages may offer optional support for 
certain file formats or network protocols. By enabling the 
appropriate USE flags for these features the package will be 
built with support for those features.  

USE flags can be customized by editing the `/etc/portage/make.conf` 
file or by creating a package-specific configuration file in the 
`/etc/portage/package.use/` directory. For example, to 
install `qtile` properly on xorg, the following lines has to 
be added to the configuration `/etc/portage/package.use/qtile`:
```
x11-libs/cairo X glib opengl svg
>=x11-libs/gdk-pixbuf-2.42.10-r1 jpeg
>=sys-auth/pambase-20220214 elogind
```  

Here are some USE flags that can be used for qtile:
* X: Enables support for X11 window system.
* dbus: Enables support for the D-Bus message bus system.
* doc: Installs documentation files.
* elogind: Enables support for the systemd-compatible login manager elogind.
* examples: Installs example configuration files.
* jpeg: Add JPEG image support.
* screenshot: Enables support for taking screenshots.
* sound: Enables support for sound systems such as ALSA or PulseAudio.


List all use flags:
```bash
portageq envvar USE | xargs -n 1
```  

List all use flags for a specific package:
```bash
euse -I X
```
