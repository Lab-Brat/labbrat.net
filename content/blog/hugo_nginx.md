---
date: "2023-02-18T17:57:22Z"
tags:
- nginx
- hugo
- website
title: Hosting a Hugo website with Nginx on RHEL.
---

### Table of Contents
- [Step 1: Buying a domain](#step-1-buying-a-domain)
- [Step 2: Installing Nginx](#step-2-installing-nginx)
- [Step 3: Installing Hugo](#step-3-installing-hugo)
- [Step 4: Configuring TLS](#step-4-configuring-tls)
- [Step 5: Launching the site](#step-5-launching-the-site)


### Step 1: Buying a domain
Buying domain name can be tricky. I chose Cloudflare because:
* Best prices overall if you take into account long term usage (>3 years).
* Has built-in reverse proxy functionaliy.
* Supports TLS certificate management.
* Captcha can be turned on in case of a DDOS attack.
* Apart from the domain name costs all other features are free!


### Step 2: Installing Nginx
The choice of a cloud provider and operating system is completely subjective, 
anything will basically do. I chose AlmaLinux becuase I am a big fan of RHEL 
Linux systems (not trying to start a fight about which distro is better).  

Before doing anything though, it's best to update OS and configure the firewall.    
**OS update and firewalld configuration**
```bash
dnf update -y
dnf install -y firewalld
systemctl start --now firewalld
firewall-cmd --add-service http --add-service https
firewall-cmd --add-service http --add-service https --permanent
```  

AlmaLinux's repository doesn't have the latest Nginx version, so we're 
going to grab it from their official repository.  
**adding Nginx repository**
```bash
cat <<EOF > /etc/yum.repos.d/Nginx.repo
[nginx]
baseurl = https://nginx.org/packages/centos/$releasever/$basearch/
enabled = 1
gpgcheck = 0
name = nginx repo
EOF

dnf install -y nginx
```

### Step 3: Installing Hugo
Since `hugo` is not present in the repository, and becuase we are not 
looking for simple solutions, we will build and compile hugo from source code. 
It's actually just 2 commands, but it might take some time for the build to complete.  
**download and build Hugo**
```bash
dnf install -y go gcc g++ git
go install github.com/gohugoio/hugo@latest
CGO_ENABLED=1 go install --tags extended github.com/gohugoio/hugo@latest
```  

After `hugo` is done building, it's time to create a website and import a theme. 
Website is create with one command, and theme for this project will be 
[PaperMod](https://github.com/adityatelange/hugo-PaperMod), 
and it will be loaded as a git submodule.  
**create site and install theme**
```bash
cd /opt
hugo new site hugo_site
cd hugo_site
chown nginx:nginx hugo_site
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
echo 'theme = "PaperMod"' >> config.toml
hugo
```

The last command will render static content into `public` directory, which will be used 
as location for Nginx config in the next step.


### Step 4: Configuring TLS

### Step 5: Launching the site