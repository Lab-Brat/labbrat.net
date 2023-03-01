---
date: "2023-02-28T23:19:01+03:00"
tags:
    - irc
    - chat
categories:
    - self-hosting
title: Using IRC in 2023
ShowToc: true
---

I really didn't think that it will ever be a neccessary to use IRC
in moderen world. 
First of all, there are better alternatives like Discord and Matrix. 
Secondly, IRC is a technology from 1988. For a bit of context, in 
1988 Soviet Union was still a thing, and Guns and Roses just started 
making good music.  

Privilleged Gen-Z rhetoric aside, it's actually not all that bad. 
Yes, if you are old school you can still use IRC in the terminal, 
and constantly remind everyone how old days were much better than now, 
but that's not very productive.  

In this article I will take a look at a modern IRC client - The Lounge. 
It will be deployed as a Docker container and linked to a doman name.  

What is need for this installation:
* Domain name, in this case a working website running on Nginx
* VPS with a public IP address
* Docker and Docker Compose installed


### Step 1: Install The Lounge
Installation is very simple and straightforward, as it is with almost 
all conteinterized software. Navigate to any directory, and create a 
`docker-compose.yaml` with following content:
```yaml
version: '2'
services:
  thelounge:
    image: thelounge/thelounge:latest
    container_name: lng
    ports:
      - "9000:9000"
    restart: always
    volumes:
      - ~/.thelounge:/var/opt/thelounge
```

Couple things to note:
* The Lounge will be available on port 9000, so it should be allowed in firewall
* All chats will be saved in `~/.thelounge`

run Compose in the same directory to launch the container:
```bash
docker compose up -d
```

To check if The Lounge is available, try to open the web UI in the browser via 
`http://<server-ip>:9000`


### Step 2: Add users
The Lounge by default is in `Private` mode, and only allows created users to 
authenticate via SASL. To create a user (will be prompted for password):
```bash
docker exec --user node -it lng thelounge add <user>
docker exec --user node -it lng thelounge list
```

The second command will list existing users, and it can be used to verify that 
a new user was really created.  

At this point it's possible to authenticate in the web UI, but please remeber 
that when accessing site using `http`, all the information entered there is 
unencrypted, so after `https` is implemented the password should be changed. 
It can easily be done in the console:
```bash
docker exec --user node -it lng thelounge reset <user>
```

### Step 3: Connecting to domain
I went through the process of creating a website with TLS in the previous 
[blog post](https://labbrat.net/blog/hugo_nginx/).  

If there is a working website with TLS configured, we can set up a reverse 
proxy to forward all traffic on port 9000 to the container. To do this, 
add to Nginx server configuration:
```
    location ^~ /lng/ {
        proxy_pass http://127.0.0.1:9000/;
        proxy_http_version 1.1;
        proxy_set_header Connection "upgrade";
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;

        # by default nginx times out connections in one minute
        proxy_read_timeout 1d;
    }
```

and reload Nginx:
```bash
nginx -t
systemctl reload nginx
```

And that's it! IRC client should be available on `https://<domain>/lng`
