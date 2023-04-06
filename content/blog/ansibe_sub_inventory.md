---
title: "Ansibe: Create Sub Inventory"
date: 2023-04-06T23:11:24+03:00
tags:
    - Ansible
categories:
    - devops
showToc: true
---

Case study: There is a large inventory with many hosts. 
The task is to create a sub-inventory with only the hosts 
that have a specific package installed. So the playbook 
has to run on all hosts and check if Docker is installed, 
and if it is, then add the host to the sub-inventory.  

How this might be useful?  
In the current situation, when the large inventory was 
compiled hosts with Docker were not organized into a 
separate group. So the tasks below will find the hosts 
with Docker, and later they can be used to create a group 
in the inventory file.  

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
