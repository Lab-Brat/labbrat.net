---
date: "2023-02-18T17:57:22Z"
tags:
- nginx
- hugo
- website
title: Hosting a Hugo website with Nginx on RHEL.
---
### [work in progress]
### Table of Contents
- [Step 1: Buying a Domain](#step-1-buying-a-domain)
- [Step 2: Installing Nginx](#step-2-installing-nginx)
- [Step 3: Installing Hugo](#step-3-installing-hugo)
- [Step 4: Launching Website](#step-4-launching-website)
- [Step 5: Configuring TLS](#step-5-configuring-tls)  


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
Since `hugo` is not present in the repository, and because we are not 
looking for simple solutions, we will build and compile hugo from source code. 
It's actually just 2 commands, but it might take some time for the build to complete.  
```bash
# download and build Hugo
dnf install -y go gcc g++ git
go install github.com/gohugoio/hugo@latest
CGO_ENABLED=1 go install --tags extended github.com/gohugoio/hugo@latest
```  

After `hugo` is done building, it's time to create a website and import a theme. 
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
