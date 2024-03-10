---
title: "Send Emails From The Terminal"
date: 2024-03-09T16:14:27+03:00
showToc: true
tags:
    - gentoo
    - linux
    - mail
    - gpg
    - oauth
categories:
    - Linux
---

## Intro
This article will show how to set-up a local email relay on a Linux machine 
with `msmtp` to send emails through Gmail. 
2 approaches will be shown:
1. Simple approach: `msmtp` and `App Password`.
2. More complex approach with `msmtp` and `mailctl` that uses OAuth 2.0 instead of a password.

Both of the methods will require a Google account and some patience since there 
will be some jumping through hoops to obtain correct credentials.  


## Configuration
### Google Account Configuration
The Google account password can't be directly used for SMTP authentication, 
this feature (`Allow Less Secure Apps`) was disabled in 2022. Instead, an
`App Password` should be created specifically for that. 
Alternatively, OAuth can also be used to authenticate and it's configuration 
will be shown in [optional](#mailctl-configuration-optional) section.  

To create an `App Password` follow the steps from the official 
[documentation](https://support.google.com/accounts/answer/185833?hl=en). 
If done correctly, you should receive a 16 character password which will be 
using the `msmtp`'s configuration file.  

### msmtp Configuration
Install msmtp:
```bash
sudo emerge --ask --quiet-build mail-mta/msmtp
```

Create a configuration file in `~/.msmtprc` with content below:
```
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account        gmail
host           smtp.gmail.com
port           465
tls_starttls   off
from           user-account@gmail.com
user           user-account@gmail.com
password       <app-password>

account default: gmail
```

### Client ID and Secret (Optional)
For this example, Google Cloud Provider will be used to register a client ID 
and a secret. Setting it up is pretty straightforward and there is no need to 
create a separate account it. Detailed instructions can be found Google's official 
[doc](https://support.google.com/cloud/answer/6158849), here is quick run down:
* Create a project and open APIs & Services page
* Configure Consent Screen
  * app name: email relay
  * support email: user-account@gmail.com
  * developer contact information: user-account@gmail.com
  * click on `Save and Continue`
  * Scopes: leave it empty since we will not be using the API
  * click on `Save and Continue`
  * publish the app
* Create a new OAuth 2.0 credentials
  * open `Credentials` tab
  * click on `+ Create New Credentials` >> Create OAuth client ID 
  * application type: Desktop app
  * name: email relay
  * click on `Create`
* Save the Client ID and secret

### mailctl Configuration (Optional)
`mailctl` (repo is being re-branded as `oama`) is a tool that handles the OAuth 
authentication flow for apps that doesn't have that functionality built-in, like 
`msmtp`. After the flow is completed it creates an `Access Token` that can be 
used to securely authenticate you Google account in `msmtp`. The token is then 
encrypted with a GPG key and stored securely in `~/.local/var/mailctl`.  

OAuth and GPG encryption can get very complicated and will not be covered in this 
article, be I found some great resources 
[here](https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc) and 
[here](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages), 
respectively, that does a great job in showing how it all works.  

Let's jump into `mailtcl` configuration. 
First, download `mailctl` binary, unpack it and create a symlink in `/usr/local/bin`:
```bash
wget https://github.com/pdobsan/oama/releases/download/0.9.2/mailctl-0.9.2-Linux-x86_64.tgz
tar xzfv mailctl-0.9.2-Linux-x86_64.tgz
sudo ln -s mailctl-0.9.2-Linux-x86_64/mailctl /usr/local/bin/mailctl
```

2 configuration files are required for it to work - `config.yaml` and `services.yaml`. 
First one defines how the app itself works and the second one defines google-specific 
settings. User is required to provide Client ID and secret from the previous steps 
and a GPG public key to encrypt the resulting access token. 
But first, let's make sure that all config directories exist:
```bash
mkdir -p ~/.config/mailctl
mkdir -p ~/.local/var/mailctl
```

and create a GPG key:
```bash
gpg --gen-key
```
The command above will prompt for some questions and will ask to create a password for the key. 
After it's done, save the public GPG key ID, it's a 17 character long string like this one `E69B67357XF632C8`. 
Now it's time to configure `mailctl`:
```yaml
# ~/.config/mailctl/config.yaml
services_file: ~/.config/mailctl/services.yaml

oauth2_dir: ~/.local/var/mailctl
encrypt_cmd:
  exec: gpg
  args:
    - --encrypt
    - --recipient
    - E69B67357XF632C8
    - -o
decrypt_cmd:
  exec: gpg
  args:
    - --decrypt
```

```yaml
# ~/.config/mailctl/services.yaml
google:
  auth_endpoint: https://accounts.google.com/o/oauth2/auth
  auth_http_method: POST
  auth_params_mode: query-string
  token_endpoint: https://accounts.google.com/o/oauth2/token
  token_http_method: POST
  token_params_mode: both
  redirect_uri: http://localhost:8080
  auth_scope: https://mail.google.com/
  client_id: <client_ID>
  client_secret: <client_secret>
```

To see if it works, first walk through the OAuth authentication flow:
```bash
mailctl authorize google user-account@gmail.com
```

This command will generate a link on localhost:8080 which will ask you 
to authenticate using the Google account that created the OAuth app. Then, 
run the command below to generate a token and encrypt it with the GPG key:
```bash
mailctl access user-account@gmail.com
```

Finally, update `mstmp` config file to use OAuth:
```
#  ~/.msmtprc
defaults
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account        gmail_oauth
auth           oauthbearer
host           smtp.gmail.com
port           465
tls_starttls   off
from           user-account@gmail.com
user           user-account@gmail.com
passwordeval   mailctl access user-account@gmail.com

account default: gmail_oauth
```


## Sending Emails
Pipe stdout of a command to `msmtp` to send emails, for example:
```bash
echo -e "Mail body text" | msmtp -a default <target-email>@gmail.com
```

No matter how long the output is, it most likely could be sent via the email relay:
```bash
echo -e "Subject: Gentoo Update Report\n\n$(gentoo-update report)" | msmtp -a default <target-email>@gmail.com
```


## Links
- [[Link](https://marlam.de/msmtp/documentation/)] - msmtp documentation
- [[Link](https://github.com/marlam/msmtp)] - msmtp GitHub
- [[Link](https://github.com/pdobsan/oama)] - mailctl GitHub
- [[Link](https://support.google.com/accounts/answer/185833?hl=en)] - Google's documentation on how to set up App Password
- [[Link](https://wiki.archlinux.org/title/msmtp)] - Arch Linux's article on msmtp
- [[Link](https://support.google.com/cloud/answer/6158849)] - Google's article on how to set up OAuth 2.0 in GCP
- [[Link](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)] - Digital Ocean's guide on GPG encryption
- [[Link](https://bence.ferdinandy.com/2023/07/20/email-in-the-terminal-a-complete-guide-to-the-unix-way-of-email/)] - A great article on email on Linux
- [[Link](https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc)] - A great article + YouTube video about OAuth
