---
title: "Migrating Nginx to Docker"
date: 2023-03-01T22:01:15+03:00
tags:
    - nginx
    - docker
    - compose
categories:
    - devops
---

There are many ways to install a Nginx web server on Linux. 
It can be installed using OS's packager manager, either from 
distro's repositories on from Nginx's official repos. It can 
be compiled from source. But my favorite way is definitely 
in a Docker container.

Main benefits are:
* Nothing is installed on the system
* Many different versions are available (without adding external repositories)
* It's easy to transition from a normal install to Docker


### Step 1: Preparing Nginx configuration file
I already wrote about creating a basic Nginx configuration file
[here](https://labbrat.net/blog/hugo_nginx/), so I will just briefly 
point out what was changed to be used in Docker.
```
...

    ssl_certificate        /etc/ssl/<domain_name>_cert.pem;
    ssl_certificate_key    /etc/ssl/<domain_name>_key.pem;
    ssl_client_certificate /etc/ssl/<domain_name>.pem;
    ssl_verify_client on;

    server_name <domain_name> www.<domain_name>;

    root /<domain_name>;  <----- read files from root directory
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
...
```  
**Note** <domain_name> is a valid domain name, and <--- points to changed part

The only thing really changed in the path to website contents. It 
had to change because `nginx` Docker image doesn't have `/var/www` path.  


### Step 2: Preparing Docker Compose file
To keep everything organized, let's create a directory to store docker and 
Nginx configuration files.
```bash
mkdir /opt/compose
touch <domain_name>.conf  # add Nginx config here
touch docker-compose.yaml
```

Open docker-compose.yaml with a text editor and add following content:
```yaml
version: '3.9'
services:
  nginx:
    image: nginx
    container_name: nginx
    volumes:
      - ./<domain_name>.conf:/etc/nginx/conf.d/<domain_name>.conf
      - /etc/ssl/:/etc/ssl/
      - /opt/website/public/:/<domain_name>
    ports:
      - "80:80"
      - "443:443"
```

The most import part is volumes, here we are:
* Mounting Nginx configuration files to Nginx folder in the container
* Mounting directory with TLS certification to the container
* And mounting website files to container's root directory


### Step 3: Launching Docker Compose
Make sure that Nginx config and compose file are in the same directory, 
and then launch Compose:
```
docker compose up -d nginx
```

In case of an error, check logs:
```
docker logs nginx
```
