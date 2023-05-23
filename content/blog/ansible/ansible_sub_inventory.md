---
title: "Ansibe: Create Sub Inventory"
date: 2023-04-06T23:11:24+03:00
tags:
    - ansible
categories:
    - DevOps
showToc: true
---

Consider a large inventory file full of hosts. 
The goal is to create a smaller list, a 'sub-inventory', 
containing only hosts with a specific service - Docker, in this case.  

The process involves running a playbook to examine every host. 
If Docker exists on a host, this host gets added to the smaller list.  

What's the significance of this action?  

During the initial creation of the large inventory, 
hosts with Docker weren't categorized separately. 
By implementing these steps, Docker hosts will be easily identifiable 
and will improve organization of the inventory.

Tasks to achieve this:
```yaml
---
- name: Gather package information.
  package_facts:
    manager: "auto"

- name: Get hosts with installed docker.
  set_fact:
    docker_host: "{{ inventory_hostname }} ansible_host={{ ansible_host }}"
  when: "'docker' in ansible_facts.packages"

- local_action: "shell echo {{ docker_host }} >> /tmp/docker_hosts"
  when: docker_host is defined
```

First, the playbook starts by gather host package information. 
`manager` is set to `auto` so that the package manager (apt, dnf etc.)
will be detected automatically.  

Then, the task `Get hosts with installed docker` will create a variable 
`docker_host` with the inventory host name and IP address. This format 
could be later used directly in the inventory file. Variable will be 
created only if the host has the `docker` package installed.  

The last task will write the variable `docker_host` to a file on the
local machine in the `/tmp/docker_hosts` temporary file.  

Sample output of the `/tmp/docker_hosts` file:
```ini
host1 ansible_host=1.2.3.4
host2 ansible_host=8.8.8.8
```

After checking the file, it can be used to create a group in the 
inventory file:
```bash
echo '[docker]' >> inventory
cat /tmp/docker_hosts >> inventory
```
