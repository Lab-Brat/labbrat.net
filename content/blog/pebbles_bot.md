---
title: "Pebbles_bot"
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
focus on showcasing it's functionalities.
 

