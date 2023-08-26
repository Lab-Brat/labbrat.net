---
title: "Hosting a Hugo website with Nginx on RHEL"
date: 2023-02-18T17:57:22Z
tags:
    - nginx
    - hugo
    - website
categories:
    - Self-Hosting
ShowToc: true
---

For me the toughest and most time consuming part of web development was 
always front end. All the centering divs, box models and CSS inheritance 
stuff. And even if I get all the code to work, website still looks like 
highschool student's project from the 90s.  

Well, no more. Hugo framework allows users to create websites using 
Markdown language, and takes care of rendering Markdown files into 
HTML and CSS code.  

In this article I will show how to set up a Nginx web server on RHEL 
(Alma Linux) and host a Hugo website on it.   


### Step 1: Buying a domain
From all the available domain regstrars I chose **Cloudflare** because:
* Best prices overall if you take into account long term usage (>3 years).
* Easy and very cutstomizable DNS management.
* It supports proxied DNS, which means that you can use Cloudflare's 
  CDN to speed up your website and get better security.
* Supports TLS certificatis creation and renewal.
* Captcha can be turned on in case of a DDOS attack.
* Apart from the domain name costs all other features are free!  


### Step 2: Installing Nginx
The choice of a cloud provider and operating system is completely subjective, 
anything will basically do. I chose AlmaLinux because I am a big fan of RHEL 
Linux systems (not trying to start a fight about which distro is better).  

Before doing anything though, it's best to update OS and configure the firewall.  
```bash
# OS update and firewalld configuration
dnf update -y
dnf install -y firewalld
systemctl start --now firewalld
firewall-cmd --add-service https
firewall-cmd --add-service https --permanent
```

AlmaLinux's repository, just like most of the most popular Linux distributions, 
doesn't have the latest Nginx version, 
so we're going to grab it from the official repository.  
```bash
# adding Nginx repository
cat <<EOF > /etc/yum.repos.d/Nginx.repo
[nginx]
baseurl = https://nginx.org/packages/centos/\$releasever/\$basearch/
enabled = 1
gpgcheck = 0
name = nginx repo
EOF

dnf install -y nginx
```

### Step 3: Installing Hugo
Unfortunately, `hugo` is also not present in the AlmaLinux repository. 
We can go nuts and compile hugo from source code 
(it's actually just 2 [commands](https://github.com/gohugoio/hugo#:~:text=Hugo%20documentation.-,Build%20and%20Install%20the%20Binary%20from%20Source%20(Using%20the%20Go%20toolchain),-Prerequisite%20Tools) + some compile time), but for the sake of simplicity we will use the official pre-compiled binary. 
Here is a small `bash` script to download and install the latest Hugo release:
```bash
#!/bin/bash
# Download the latest tar.gz archive
git_api_url=https://api.github.com/repos/gohugoio/hugo/releases/latest
curl -s $git_api_url \
        | grep -E "browser_.*_Linux-64bit.tar.gz" \
        | head -n 1 \
        | cut -d : -f 2,3 \
        | tr -d \" \
        | wget -qi - -O ./hugo_latest.tar.gz
echo "[Hugo binary downloaded]"

# Unpack the archive
mkdir hugo_latest
tar xf hugo_latest.tar.gz -C ./hugo_latest
cp ./hugo_latest/hugo /usr/local/bin
echo "[Hugo binary unpacked and installed]"

# Clean-up
rm -rf hugo_latest*
echo "[Residual files deleted]"
```

Script places `hugo` executable to `/usr/local/bin`, which should be in `$PATH` variable already. 
To test if installation was successful, run:
```bash
hugo --help
```

Now it's time to create a website. 
Website folder will be located in `/opt/hugo_site` directory, so first navigate to `/opt`, 
create a new site with `hugo` comand and change ownership of the `public` directory to `nginx` user. 
This is necessary because Nginx will be serving static files from this directory.
```bash
# create site and change 'public' directory ownership
cd /opt
hugo new site hugo_site
cd hugo_site
chown nginx:nginx public
```

Hugo supports a lot of [themes](https://themes.gohugo.io) and it might be hard to choose one. 
Generally, it's best to choose a theme that is well maitaing (has recent commits in Github repo) 
and has a good documentation. 
For this project I chose [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 
and it will be installed as a git submodule.  
```bash
# install Hugo theme
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
echo 'theme = "PaperMod"' >> config.toml
hugo
```

The last command will render static content into `public` directory, which will be used 
as location for Nginx config in the next step.  


### Step 4: Launching Website
With Nginx installed and static files created in `/opt/hugo_site/public`, 
let's launch the website right away and see how it looks!  
For that, create a symlink in `/var/www` directory to point to the static files, 
and then create a basic Nginx configuration file for the website.
```bash
# create symlink and Nginx config
ln -s /opt/hugo_site/public /var/www/domain_name.com
cat <<EOF > /etc/nginx/conf.d/domain_name.com.conf
server {
    listen 80;
    listen [::]:80;

    server_name domain_name.com www.domain_name.com;

    root /var/www/domain_name.com;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
EOF
```

After that, check if Nginx syntax is valid, and reload Nginx service:
```bash
# validate Nginx configs syntax and reload service
nginx -t
systemctl reload nginx
```
If there were no errors, website should be available in the browser at 
`http://domain_name.com`. Or it can be tested with curl without leaving the server:
```bash
# get the web page
curl http://www.domain_name.com
```  


### Step 5: Configuring TLS
Encrypting traffic is a must these days, even if it's a simple static website. 
Thankfully, Cloudflare makes creating certificates as easy as clicking couple 
buttons in the web admin panel.  

Is TLS encryption even required if Cloudflare proxy is turned on? Of course it is! 
Cloudflare proxy is encrypting traffic between client and the proxy, but traffic 
between proxy and the server is still unencrypted.  

Let's start by creating Origin Certificate - a TLS certificate signed by Cloudflare 
to authenticate your server.  
In the Cloudfare admin page go to:  
 `<zone_name>` -> `SSL/TLS` -> `Origin Server` -> `Create Certificate`  
After following default steps Cloudflare will generate the certificate and a private key. 
Both of them should be copied to the server with correct permissions.  
```bash
# copy certificates
/etc/ssl/domain_name.com_cert.pem # copy certificate content to this file
/etc/ssl/domain_name.com_key.pem  # copy key content to this file 
chmod 600 /etc/ssl/domain_name.com_{cert,key}.pem
```

After certificate and key files are placed on the server, full encryption mode should 
be turned on.  
Steps: `<zone_name>` -> `SSL/TLS` -> `Origin Server` -> `Overview` -> `Full (strict)`  

To make the website even more secure, let's add an additional certificate to
verify that all traffic from the web server is received from Cloudflare's infrastructure. 
Steps: `<zone_name>` -> `SSL/TLS` -> `Origin Server` -> `Authenticated Origin Pulls`  

After turning origin pulls, go back to the server and download Cloudflare's 
certificate that will be used for the verification.  
```bash
# download Cloudflare's certificate
wget https://developers.cloudflare.com/ssl/static/authenticated_origin_pull_ca.pem -O /etc/ssl/cloudflare.pem
chmod 600 /etc/ssl/cloudflare.pem
```

After this step, all requests to the website that are not from Cloudflare will get 400 error.  

Lastly, let's update Nginx's configuration file to use TLS encryption:
```bash
# Nginx configuration file with TLS settings
server {
    listen 80;
    listen [::]:80;

    server_name domain_name.com www.domain_name.com;
    return 302 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
  
    ssl_certificate        /etc/ssl/domain_name.com_cert.pem;
    ssl_certificate_key    /etc/ssl/domain_name.com_key.pem;
    ssl_client_certificate /etc/ssl/cloudflare.pem;
    ssl_verify_client on;
  
    server_name domain_name.com www.domain_name.com;
  
    root /var/www/domain_name.com;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Validate Nginix config syntax and reload the service if there are not mistakes
```bash
# validate Nginx configs syntax and reload service
nginx -t
systemctl reload nginx
```

Enter website URL `https://domain_name.com` in the browser and check if the 
website is working with TLS encryption. If it works a green lock icon should 
appear in the browser next to the URL input field.  


### References
* \[[Link](https://blog.cloudflare.com/registrar-for-everyone/)\] Registrar for Everyone 
* \[[Link](https://gohugo.io/getting-started/quick-start/)\] Hugo Quickstart 
* \[[Link](https://github.com/adityatelange/hugo-PaperMod/wiki)\] PaperMod Theme Wiki 
* \[[Link](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)\] Nginx Repository Installation 
* \[[Link](https://www.digitalocean.com/community/tutorials/how-to-host-a-website-using-cloudflare-and-nginx-on-ubuntu-20-04)\] How To Host a Website Using Cloudflare and Nginx on Ubuntu 20.04 
