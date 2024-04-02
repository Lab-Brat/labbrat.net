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

Proton team does not build packages for Gentoo Linux, so we have to 
figure out how to install it on our own. From what I have gathered, 
there are 3 ways to go about it: download OpenVPN and Wireguard 
configuration files from the ProtonVPN UI, install GUI and CLI apps 
from sources or use community CLI.  

## Configuration Files


## Install From Sources


## Community CLI
> ⚠️ Since March 2023 Proton team stopped allowing API calls from 
> the community project, details: https://github.com/Rafficer/linux-cli-community/issues/365

[Community CLI](https://github.com/Rafficer/linux-cli-community) 
can be installed via `net-vpn/protonvpn-cli` package. 
After installation, run `protonvpn` command to establish to connect to VPN. 

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

## Links
- [[Link](https://github.com/Rafficer/linux-cli-community/blob/master/USAGE.md)] - Community CLI user guide
- [[Link](https://protonvpn.com/support/official-linux-client)] - Official tool

