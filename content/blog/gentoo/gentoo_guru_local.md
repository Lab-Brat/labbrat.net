---
title: "Gentoo: Set up GURU as a local repository"
date: 2023-03-08T09:31:43+03:00
tags:
    - gentoo
    - linux
    - portage
    - ebuild
categories:
    - Gentoo
showToc: true
---

This article is a summary of couple Gentoo Wiki articles that teaches 
how to get access to Gentoo's GURU overlay and start creating your own 
ebuilds or maintain existing ones.  

**Disclaimer:** I am not a Gentoo developer, and I just started interacting with Gentoo 
community and this is what I've learned so far. There are probably (definitely) better 
ways to do this, and I'll be glad to hear suggestions.

### Step 1: Request Access
This step is not mandatory if the goal is to create ebuilds locally 
without pushing them to the repository. However, SSH access is required 
to actually upload ebuilds to GURU repository. Since GURU is a community 
project, anyone can request access via a bug in Gentoo bugzilla.  
For example, this was my request: https://bugs.gentoo.org/899896  
Required [information](https://wiki.gentoo.org/wiki/Project:GURU/Information_for_Contributors#:~:text=Once%20you%27re%20ready%2C%20please%20file%20an%20access%20request%2C%20including%3A):
* Real name
* Email
* public SSH key (`~/.ssh/id_gentoo.pub``)

It also might be helpful to create GPG key at this point. 
Follow their [guide](https://wiki.gentoo.org/wiki/Project:Infrastructure/Generating_GLEP_63_based_OpenPGP_keys) until `Submit the new key to the keyserver` section. 
If everything was done correctly, there will be a set of GPG keys available on the system.


### Step 2: Clone GURU
Starting from this point, everything will be done on a Gentoo Linux machine.
GURU repository can be cloned without any authentication with:
```bash
git clone https://anongit.gentoo.org/git/repo/proj/guru.git
```

But this method has limitations. For example, it is not possible to make changes and push 
them to the repository. To be able to do that, SSH access (that was requested in Step 0) is required.  

If the access was granted, add SSH and Gentoo git information to `~/.ssh/config`:
```
Host git.gentoo.org
    HostName git.gentoo.org
    User git
    IdentityFile ~/.ssh/id_gentoo
```

After that, `git` will know how to connect to Gentoo's git server, and it will be possible to 
clone GURU repository with authentication.
Navigate to `/var/db/repos` and clone the GURU repository:
```bash
git clone -b dev git+ssh://git@git.gentoo.org/repo/proj/guru.git
```


### Step 3: Configure Local Repository
Now it's time to tell Portage that a new repository exists. First, change GURU repository's 
ownership to `portage:portage`:
```bash
chown -R portage:portage /var/db/repos/guru
```

Then, add the following lines to `/etc/portage/repos.conf/guru.conf`:
```ini
[guru]
location = /var/db/repos/guru
```

Finally, test to see if ebuilds from the new repository are available:
```
emerge --ask dev-python/sendgrid
```

**[ Small Tangent ]**  
Normally setting up local repo is a bit more complex than this. 
Here are the steps to create a local repo named `labbrat`:
```bash
mkdir -p /var/db/repos/labbrat/{metadata,profiles}
chown -R portage:portage /var/db/repos/labbrat
echo 'labbrat' > /var/db/repos/labbrat/profiles/repo_name 
echo 'masters = gentoo' >> /var/db/repos/labbrat/metadata/layout.conf
echo 'auto-sync = false' >> /var/db/repos/labbrat/metadata/layout.conf
echo '[labbrat]' >> /etc/portage/repos.conf/labbrat.conf
echo 'location = /var/db/repos/labbrat' >> /etc/portage/repos.conf/labbrat.conf
```

Then, to create an ebuild there:
```bash
cd /var/db/repos/labbrat/
mkdir -p app-misc/your-app
cd app-misc/your-app
vim your-app.ebuild # write an ebuild
ebuild your-app.ebuild manifest
```

And finally, install it!
```bash
emerge --ask app-misc/your-app
```

### Step 4: Making Changes
To create your own ebuilds and change existing ones, a string list of settings 
must be applied to the repository git config. In the repo, run:
```bash
git config --local pull.ff only
git config --local pull.rebase merges
git config --local commit.gpgsign 1
git config --local push.gpgsign 1
git config --local user.name "Your Full Name"
git config --local user.email "someone@example.com"
git config --local user.signingkey KEY-FINGERPRINT
```

The `KEY-FINGERPRINT` is the signing GPG key (`[S]`) that was created in Step 0. 
GURU repository does not verify the GPG key, but without it it's not possible to push changes. 
As the name suggests, this key will be used for signing commits, and to use it, run:
```bash
git commit -sS
```

The `-s` flag will add a `Signed-off-by` line to the commit message, and the `-S` flag will
sign the commit with the GPG key. Sometimes, especially when the GPG key has is password protected, 
this error might pop up in the terminal:
```
# error: gpg: signing failed: Inappropriate ioctl for device
```

To fix this, run:
```bash
export GPG_TTY=$(tty)
```


### References
* \[ [Link](https://wiki.gentoo.org/wiki/Project:GURU/Information_for_Contributors) \] Information for Contributors
* \[ [Link](https://wiki.gentoo.org/wiki/Creating_an_ebuild_repository) \] Creating an ebuild repository
* \[ [Link](https://wiki.gentoo.org/wiki/Project:Infrastructure/Generating_GLEP_63_based_OpenPGP_keys)] Generating OpenPGP keys
* \[ [Link](https://wiki.gentoo.org/wiki/Gentoo_git_workflow) \] Gentoo git workflow
