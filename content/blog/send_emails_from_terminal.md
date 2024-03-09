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
with `msmtp` to send emails through Gmail. 2 aproaches will be show - first 
one will use `msmtp` and `App Password`, the second one will use `msmtp` and
`mailctl` to use OAuth 2.0 instead of passwords. Both of the methods will 
require a Google account.  


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

Create a configuation file in `~/.msmtprc` with content below:
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
  * pubilsh the app
* Create a new OAuth 2.0 credentials
  * open `Credentials` tab
  * click on `+ Create New Credentials` >> Create OAuth client ID 
  * application type: Desktop app
  * name: email relay
  * click on `Create`
* Save the Client ID and secret

### mailctl Configuration (Optional)
Download `mailctl` binary, unpack it and create a symlink in `/usr/local/bin`:
```bash
wget https://github.com/pdobsan/oama/releases/download/0.9.2/mailctl-0.9.2-Linux-x86_64.tgz
tar xzfv mailctl-0.9.2-Linux-x86_64.tgz
sudo ln -s mailctl-0.9.2-Linux-x86_64/mailctl /usr/local/bin/mailctl
```

2 configuration files are required for it to work - `config.yaml` and `services.yaml`. 
First one defines how the app itself works and the second one defines google-specific 
settings. User is required to provide Client ID and secret from the previous steps 
and a GPG public key to encrypt the resulting token. But first, let's make sure that 
all config directories exist and create a GPG key (more on GPG keys 
[here](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)):
```bash
mkdir -p ~/.config/mailctl
mkdir -p ~/.local/var/mailctl
gpg --gen-key
```

Save the public GPG key ID, it's a 17 character long string like this one `E69B67357XF632C8`. 
Now it's time to configure the `mailctl`:
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
to authorize using the Google account that created the OAuth app. Then, 
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
- [[Link](https://github.com/marlam/msmtp)] - msmtp Github
- [[Link](https://github.com/pdobsan/oama)] - mailctl Github
- [[Link](https://support.google.com/accounts/answer/185833?hl=en)] - Google's documentation on how to set up App Password
- [[Link](https://wiki.archlinux.org/title/msmtp)] - Arch Linux's article on mstmp
- [[Link](https://support.google.com/cloud/answer/6158849)] - Google's article on how to set up OAuth 2.0 in GCP
- [[Link](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)] - Doc on GPG keys
- [[Link](https://bence.ferdinandy.com/2023/07/20/email-in-the-terminal-a-complete-guide-to-the-unix-way-of-email/)] - Just a great article on email on Linux
