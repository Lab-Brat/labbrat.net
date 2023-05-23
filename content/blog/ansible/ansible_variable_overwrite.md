---
title: "Ansible: Combining one variable from multiple sources"
date: 2023-04-05T22:15:28+03:00
tags:
    - ansible
categories:
    - DevOps
showToc: true
---

Case study: Imagine that firewalld needs to be configured on 
multiple servers with Ansible. Different servers might have 
different ports and services are allowed through the firewall. 
But at the same time some settings are same across all servers, 
for example the default zone. This article will attempt to 
provide the best way to configure `host_vars` and `group_vars` 
for firewalld configuration.  

Since the main focus of this article is on variables, 
only localhost will be used in examples below.  

Basic directory tree structure for all examples:
```
├── ansible.cfg
├── group_vars
│   └── all.yaml
├── host_vars
│   └── localhost.yaml
└── site.yaml
```

Configuration file for Ansible:
```ini
[defaults]
host_key_checking = False
stderr_callback = debug
```

Contents of variable files:
```yaml
# group_vars/all.yaml
firewalld:
  default_zone: internal

# host_vars/localhost.yaml
firewalld: 
  ports:
    - 8080/tcp
  services:
    - http
    - https
```

Contents of the main playbook:
```yaml
---
- hosts: localhost
  gather_facts: yes
  become: true

  tasks:
  - name: Print 'firewalld' variable.
    debug:
      var: firewalld
```  

When playbook is run, `host_vars` will overwrite `group_vars` 
and the `default_zone: internal` variable will be erased. 
Sample output:
```json
"firewalld": {
    "ports": [
        "8080/tcp"
    ],
    "services": [
        "http",
        "https"
    ]
}
```
Below are a couple of methods to go about this problem.  


### Method 1: Merge host_vars and group_vars
By default whenever Ansible encounters same variables it only 
keep the last one it encountered (and overwrite the old one). 
This can be changed by setting `hash_behaviour` variable in the 
config file:
```ini
[defaults]
host_key_checking = False
stderr_callback = debug
hash_behaviour = merge
```

Now playbook output will look like this:
```json
"firewalld": {
    "default_zone": "internal",
    "ports": [
        "8080/tcp"
    ],
    "services": [
        "http",
        "https"
    ]
}
```

Drawbacks of this method:
1. This method will merge all variables, not just the ones 
   that are in `host_vars` and `group_vars`. This might not 
   be a problem with a small number of variables, but it 
   becomes a problem for larger playbooks. Furthermore, this 
   option makes the playbook non portable.
   More info in [documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-hash-behaviour)
2. `hash_behaviour` is [deprecated](https://github.com/ansible/ansible/issues/73089) (although it still works).

Due to the 2 main drawback Ansible developers recommend 
using `combine` filter instead.  


### Method 2: combine filter
[combine](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/combine_filter.html) 
filter is one of the built-in 
[filter plugins](https://docs.ansible.com/ansible/latest/plugins/filter.html) in Ansible. 
As the name suggests, it takes 2 (or more) different dictionaries and combines them 
into 1. First, the `firewalld` variable names has to be changed to something unique, 
and after that `combine` filter can be used to merge them:
```yaml
# group_vars/all.yaml
firewalld_standard:
  default_zone: internal

# host_vars/localhost.yaml
firewalld_specific: 
  ports:
    - 8080/tcp
  services:
    - http
    - https

firewalld: "{{ firewalld_specific | combine(firewalld_standard) }}"
```

Combined `firewalld` variable is placed in `host_vars` file 
because it is read last. 


### Method 3: Read directly from group_vars
Perhaps the most straightforward way combine the two variables 
is to read the `group_vars` file directly in the `host_vars`:
```yaml
# group_vars/all.yaml
firewalld_standard:
  default_zone: internal

# host_vars/localhost.yaml
firewalld: 
  default_zone: "{{ firewalld_standard['default_zone'] }}"
  ports:
    - 8080/tcp
  services:
    - http
    - https
```  

This way no configuration changes are made and 
no filters are used.


### References
* [[Link]](https://docs.ansible.com/ansible/latest/reference_appendices/config.html) Ansible Configuration Settings
* [[Link]](https://serverfault.com/questions/1034221/ansible-avoid-duplicates-between-group-and-host-vars) StackOverflow: Ansible: Avoid duplicates between group and host vars
