---
title: "Pebbles Linux Bot [WIP]"
date: 2023-05-01T16:58:09+03:00
tags:
    - Linux
    - Telegram
    - bot
categories:
    - self-hosted
ShowToc: true
---

Pebbles is a Telegram bot designed to address a specific challenge: 
the difficulty of executing commands on Linux servers from a mobile device.  

While there are already excellent CLI tools for mobile devices, 
such as [Termux](https://termux.dev/en/), it didn't quite fit my needs. 
Typing CLI keys on a small touch screen can be cumbersome, 
especially in crowded spaces like a subway.  

A Telegram bot emerged as an ideal solution since I was already an avid 
Telegram and Python user. With the bot, I could effortlessly send commands 
as if I were simply texting someone.  


### Overview
In summary, Pebbles is effortlessly installed on a server using pip (or deployed 
within a container). Afterward, users simply initiate a conversation with the bot 
through Telegram and provide it with the desired commands to execute.  

Running a bot with open access to a Linux server can pose a risk, as anyone on 
the internet could potentially attempt to use it if they discover the bot's handle. 
Therefore, Pebbles incorporates a whitelisting mechanism, which restricts command 
execution to only specific, approved user IDs. While this makes Pebbles generally 
safe to use, it is advisable to exercise caution when using it on production servers.  

Pebbles offers a wide range of functionalities, such as executing shell commands on 
local or remote Linux systems, sending server notifications, facilitating team 
collaboration through group chats, and many more diverse capabilities.  

The detailed installation process can be found on the bot's 
[Github](https://github.com/Lab-Brat/pebbles_bot) page, while this blog post will 
focus on showcasing it's functionalities. To do that, 2 virtual machines will be 
used:
```
VM_Name   VM_IP          VM_OS         VM_Role
peb1.lab  192.168.56.30  AlmaLinux 9   bot is deployed here
peb2.lab  192.168.56.31  Ubuntu 22.04  bot will connect via SSH to here 
```

### Whitelisting Users
By default nobody is allowed to send commands to Pebbles. To change this, 
`PEBBLES_USER_WHITELIST` variable should be defined in `~/.bashrc`. But first 
let's find the user's Telegram ID.  

To do that, find [@userinfobot](https://github.com/nadam/userinfobot) in Telegram 
and initiate a conversation. Bot will send a message with `id` and `name`.  

![Pebbles: Get user ID and add to whitelist](/img/lb_pebbles_uinfo.png)

`id` then can be used to create a variable:
```bash
export PEBBLES_USER_WHITELIST='12345678910,0000111122'
```


### Run Commands Locally
The most basic usage of Pebbles is to pass commands to run on the machine it's 
deployed at. It's done by calling `/run` commnd followed by the command:

![Pebbles: Run Commands Locally](/img/lb_pebbles_runloc.png)

### Run Commands Remotely
To be able to succsessfully run commands on the remote servers, make sure to 
first config the `~/.ssh/config` file first. Password authentication is not 
supported at all! Here is how config file looks on `peb2.lab`:
```
# ~/.ssh/config file on peb1.lab
Host peb2.lab
	User vagrant
	Port 22
	IdentityFile /vagrant/vagrant_key
```

then run `/mode` and switch to `Remote` mode.   

After a SSH connection is established, commands can be run using `/run` command.

![Pebbles: Run Commands Locally](/img/lb_pebbles_runrem.png)

