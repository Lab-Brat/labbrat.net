---
title: "Installing Protonvpn on Gentoo Linux"
date: 2023-09-24T23:26:48+03:00
tags:
    - gentoo
    - protonvpn
categories:
    - Gentoo
showToc: true
---

Protonvpn CLI tool differs a bit from other distributions. 
In Gentoo, after installing `net-vpn/protonvpn-cli` package 
with Portage, `protonvpn-cli` command won't be available, 
instead `protonvpn` command will be used. This is because 
upstream of the 
[ebuild](https://packages.gentoo.org/packages/net-vpn/protonvpn-cli) 
is actually a CLI tool developed by the 
[community](https://github.com/Rafficer/linux-cli-community), 
and it's not the official ProtonVPN CLI.  

### Installation
Repository has the required package, so it can be easily emerged:
```bash
emerge --ask net-vpn/protonvpn-cli
```

It can also be installed via pip (not preferred):
```bash
python -m venv protonvpn
source protonvpn/bin/activate
python -m pip install protonvpn-cli --break-system-packages>
alias protonvpn /home/labbrat/protonvpn/bin/activate protonvpn
```

### Configuration
`protonvpn` tool doesn't have `login` option and authentication 
is done via `init`. Main difference between the 2 is `login` uses 
Proton credentials as login, and `init` uses OpenVPN credentials, 
which can be found in account [settings](https://account.protonvpn.com/account).  

Find OpenVPN settings and enter them in init:
```bash
protonvpn init
```

Everything else is pretty much the same. To connect just run:
```bash
protonvpn c
```

### Links
* [[Link](https://github.com/Rafficer/linux-cli-community/blob/master/USAGE.md)] - Community CLI user guide
- [[Link](https://protonvpn.com/support/official-linux-client)] - Official tool
