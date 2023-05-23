---
title: "Ansibe: Finding Public IP Addresses With Filter Plugin"
date: 2023-05-11T21:56:13+03:00
tags:
    - ansible
categories:
    - DevOps
showToc: true
---

Consider a large inventory file full of hosts, some of them
only have private IP addresses, but some of them have public 
IPs as well. The goal is to identify which hosts have a public 
IP and print it in a debug message.  

The process involves running a playbook with a custom filter 
plugin to examine every host. If a public IP exists on a host, 
the print a debug message to identify the host.  


### Filter Plugin
Unfortunately, there's no built-in way to filter public IP 
addresses in Ansible, because determining whether an IP address 
is public or private requires comparing it with various IP ranges, 
and Ansible facts don't provide this functionality directly.  

However, it's possible to create a custom Ansible filter plugin 
to perform this task. Ansible filter plugins manipulate data inside 
the playbook during the run.  

Here is the code for a plugin that filters out public IP addresses:
```python
from ansible.errors import AnsibleFilterError
import ipaddress


def filter_public_ips(ip_list):
    public_ips = []
    for ip in ip_list:
        try:
            ip_obj = ipaddress.ip_address(ip)
            if ip_obj.is_global:
                public_ips.append(ip)
        except ValueError as e:
            raise AnsibleFilterError("Invalid IP address: {}: {}".format(ip, e))
    return public_ips


class FilterModule(object):
    def filters(self):
        return {"filter_public_ips": filter_public_ips}
```

For the plugin to work, it needs to be placed in directory named `filter_plugins`, 
for example:
```
├── ansible.cfg
├── filter_plugins
│   └── ip_filter.py
├── inventory.yaml
└── site.yaml
```


### Ansible Playbook
To use the filter plugin, the playbook will first read the 
`ansible_all_ipv4_addresses` variable to obtain all IP addresses 
available on the host, and then apply the filter plugin on it:
```yaml
---
- name: Get public IP
  hosts: all
  gather_facts: yes

  tasks:
    - name: Get public IP addresses.
      set_fact:
        public_ips: "{{ ansible_all_ipv4_addresses | filter_public_ips }}"
    
    - name: Display public IP addresses.
      debug:
        msg: "{{ public_ips }}"
      when: public_ips | length > 0
```

The playbook will print out all available public IP addresses 
during execution on the console output. 

