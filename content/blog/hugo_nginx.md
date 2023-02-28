---
date: "2023-02-18T17:57:22Z"
tags:
- nginx
- hugo
- website
title: Hosting a Hugo website with Nginx on RHEL.
ShowToc: true
---


### Step 1: Buying a domain
Buying domain names can be tricky. I chose Cloudflare because:
* Best prices overall if you take into account long term usage (>3 years).
* Has built-in reverse proxy functionality.
* Supports TLS certificate management.
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

AlmaLinux's repository doesn't have the latest Nginx version, so we're 
going to grab it from their official repository.  
```bash
# adding Nginx repository
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
Unfortunately, `hugo` is not present in the AlmaLinux repository. 
We can go nuts and compile hugo from source code 
(it's actually just 2 [commands](https://github.com/gohugoio/hugo#:~:text=Hugo%20documentation.-,Build%20and%20Install%20the%20Binary%20from%20Source%20(Using%20the%20Go%20toolchain),-Prerequisite%20Tools) + some compile time), but for the sake of simplicity we will use the official pre-compiled binary. 
Here is a small bash script to download and install the latest Hugo release:
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

Script will place `hugo` executable to `/usr/local/bin`, which should be in `$PATH` variable already. 
To test it run:
```
hugo --help
```

Now it's time to create a website and import a theme. 
Website is create with one command, and theme for this project will be 
[PaperMod](https://github.com/adityatelange/hugo-PaperMod), 
and it will be loaded as a git submodule.  
```bash
# create site and install theme
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


### Step 4: Launching Website
Since Nginx is installed and static files were genereated by Hugo, it's possible to launch the 
website right away and see how it looks!  
For that, let's create a symlink in `/var/www` directory to point to the static files, 
and then we can go and create a config file for Nginx:
```bash
# create symlink and Nginx config
ln -s /opt/hugo_site/public /var/www/domain_name.com
vim /etc/nginx/conf.d/domain_name.com.conf
```
```
# Nginx configruation file content
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
```
After that, check if Nginx syntax is valid, and reload Nginx service:
```bash
# validate Nginx configs syntax and reload service
nginx -t
systemctl reload nginx
```
If there were no errors, then the website should be available in the browser at 
`http://domain_name.com`. Or it can be tested with curl without leaving the server:
```bash
# get the web page
curl https://www.domain_name.com
```  


### Step 5: Configuring TLS
Encrypting traffic is a must these days, even if it's a simple static website. 
Thankfully, with Cloudflare creating certificates is as easy as clicking couple 
buttons in the web admin panel.  

Is TLS encryption even required if Cloudflare proxy is turned on? Of course it is! 
Cloudflare proxy is encrypting traffic between client and the proxy, but traffic 
between proxy and the server is still unencrypted.  

We can start by creating Origin Certificate - a TLS certificate signed by Cloudflare 
to authenticate your server.  
Steps: `<zone_name>` -> `SSL/TLS` -> `Origin Server` -> `Create Certificate`  
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

To make the website even more secure, it's possible to add an additional certificate to
verify that all traffic from the web server is received from Cloudflare's infrastructure. 
Steps: Steps: `<zone_name>` -> `SSL/TLS` -> `Origin Server` -> `Authenticated Origin Pulls`  

After turning origin pulls toggle on, go back to the server and download Cloudflare's 
certificate that will be used for the verification.  
**downloading and installing cloudflare's certificate**
```bash
# download Cloudflare's certificate
wget https://developers.cloudflare.com/ssl/static/authenticated_origin_pull_ca.pem -O /etc/ssl/cloudflare.pem
chmod 600 /etc/ssl/cloudflare.pem
```

After this step, all requests to the website that are not from Cloudflare will get 400 error. 
This can be tested by entering `https:<server_ip>.<tld>` into the browser.  

Lastly, let's update Nginx's confguration:
```
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
After service reload the URL `https://domain_name.com` should render the website 
with TLS encryption enabled. It can be verifying by checking the lock icon in the 
browser next to the URL input field. 
