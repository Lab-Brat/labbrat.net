---
title: "Easily Deploy Virtual Machines in Hetzner Cloud Using Terraform: A Step-by-Step Guide"
date: 2025-05-04T19:20:00Z
tags:
    - terraform
    - hetzner
    - iac
categories:
    - DevOps
ShowToc: true
---

Deploying virtual machines can be manual, boring, and repetitive, but with the right tools, it becomes a seamless process. In this article, I’ll guide you through using Terraform to deploy 5 virtual machines in Hetzner Cloud. All the infrastructure component will be defined in Terraform files and stored in Git, adhering to Infrastructure As Code (IAC) principles.

All commands below are run on Ubuntu 24.04 OS, but it should be same on any other Linux distro or MacOS.

> also published 👉[here](https://blog.devops.dev/easily-deploy-virtual-machines-in-hetzner-cloud-using-terraform-a-step-by-step-guide-78730ff874cd)

{{< details title="**TLDR**" >}}
  * Get the prerequisites - Hetzner Cloud account, create a project there, register an API token and generate a SSH key. 
  * Install Terraform.
  * Clone the repository with Terraform files.
  ```
  git clone https://github.com/Lab-Brat/terraform-hz-k8s.git && cd terraform-hz-k8s
  ```
  * Create `terraform.tfvars` by copying the template, and edit it and `vars.tf` with your information.
  ```
  cp terraform.tfvars.template terraform.tfvars
  ```
  * Init and deploy.
  ```
  terraform init
  terraform plan
  terraform apply
```
{{< /details >}}

## Prerequisites

Here are the prerequisites before we start:

1.  **Hetzner Cloud Account**: Create a Hetzner Cloud account. Then, create a project there where virtual machines will be deployed in.
2.  **API Token**: Generate an API token with read and write permissions. Obtain this from the Hetzner Cloud console under API Tokens in the new project.
3.  **SSH Public Key**: Create a SSH key pair that will be used to connect to VMs. SSH key pair can be created like this: `ssh-keygen -P '' -f ~/.ssh/key_for_hz_vms -t ed25519 -q`

## Setting Up Terraform

### Installing Terraform

Begin by installing Terraform on your operating system. Detailed OS-specific instructions can be found in the Terraform's [official documentation](https://developer.hashicorp.com/terraform/install). Confirm the installation by running the command:

```
➜  ~ terraform -version
Terraform v1.11.4
on linux_amd64
```

This should display the installed version of Terraform.

### Cloning The Repository With Terraform Files

To save some time of copy-pasting the code, clone and navigate to my GitHub repository where everything is already prepared:

```
➜  git clone https://github.com/Lab-Brat/terraform-hz-k8s.git
Cloning into 'terraform-hz-k8s'...
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 8 (delta 0), reused 8 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (8/8), done.
➜  cd terraform-hz-k8s
```

### Terraform files overview

There are several files in the repo, each defines a specific object:

-   `providers.tf` - Terraform providers are defined here. Think of them as library imports in programming languages.
-   `vars.tf` - Declares variables used in other files, and sets default values for non-sensitive vars.
-   `terraform.tfvars.template` - Template for a special file (`terraform.tfvars`), holds variables with sensitive information, such as the API token.
-   `firewall.tf` - Defines Hetzner Cloud firewall rules.
-   `main.tf` - Main logic that creates virtual machines and assigns IP addresses to them.

#### providers.tf

For this project we will only need [Hetzner Cloud provider](https://registry.terraform.io/providers/hetznercloud/hcloud/latest/docs), but we can add as many as we need. For example, if for each VM a domain name is required, then [Cloudflare provider](https://registry.terraform.io/providers/cloudflare/cloudflare/latest) could be added as well.

```
terraform {
  required_providers {
    hcloud = {
      source  = "hetznercloud/hcloud"
      version = "~> 1.50"
    }
  }
}
```

#### vars.tf

Variable declarations are stored here. Each variable has a `description` key and if it's value is non-sensitive, then it will be defined via `default` key:

```
variable "hcloud_token" {
  description = "API token with edit rights"
  sensitive   = true
}

variable "ssh_pub_key" {
  description = "Path to public SSH key that will be copied on the VM"
}

variable "firewall_allowed_ips" {
  description = "List of IP address ranges that are allowed through firewall"
}

variable "location" {
  description = "Hetzner Cloud location name"
  default     = "nbg1"
}

variable "instances" {
  description = "Amount of VMs to deploy"
  default     = "2"
}

variable "datacenter" {
  description = "Datacenter ID"
  default     = "nbg1-dc3"
}

variable "server_type" {
  description = "Type of server, i.e how much CPU/RAM/Disk it uses"
  default     = "cx22"
}

variable "os_type" {
  description = "Virtual Machine operating system"
  default     = "alma-9"
}
```

The first 3 variables only have a description because they all contain sensitive information and will be declared in the next section. Such design allows us to safely commit `vars.tf` file to the repository without worrying about leaking the token because `terraform.tfvars` file has been added to `.gitignore`. `hcloud_token` has an additional `sensitive` parameter to ensure it's not accidentally printed in the output.

List of location names for `location` and `datacenter` variables can be found in the [Hetzner Cloud documentation](https://docs.hetzner.com/cloud/general/locations/#what-datacenters-are-there). And all currently available `sever_type` can either be checked on [their main website](https://www.hetzner.com/cloud), or obtained via an [API call](https://docs.hetzner.cloud/#server-types) using the same token we'll use for Terraform:

```
➜  ~ export HZ_TOKEN="xxxxxxxxx"
➜  ~ curl -s -H "Authorization: Bearer $HZ_TOKEN" "https://api.hetzner.cloud/v1/server_types" | jq '.server_types[].name'
"cpx11"
"cpx21"
.....
"cx42"
"cx52"
```

Same for `os_type`, we can see available options via [API](https://docs.hetzner.cloud/#images):
```
➜  ~ curl -s -H "Authorization: Bearer $HZ_TOKEN" "https://api.hetzner.cloud/v1/images" | jq '.images[].name'
.....
"ubuntu-20.04"
"ubuntu-22.04"
"alma-8"
"alma-9"
```

#### terraform.tfvars

`terraform.tfvars` is not present in the repo because it's excluded in `.gitignore`. So to create it, just copy the `terraform.tfvars.template` with the name `terraform.tfvars`, like so:
```
➜  terraform-hz-k8s git:(main) cp terraform.tfvars.template terraform.tfvars
```

Here is how it should look like inside:
```
hcloud_token = ""
ssh_pub_key  = ""
firewall_allowed_ips = [
    "1.1.1.1/32",
]
```

Add your Hetzner Cloud API token to `hcloud_token` variable, path to the public SSH key to `ssh_pub_key`, and replace `1.1.1.1/32` with your IP address or address ranges from which SSH connections to VMs will be made.

#### firewall.tf

2 firewall rules will be defined to make sure VMs are safe on the internet:

1.  Allow ICMP from all addresses. This will allow us to ping these resources.
2.  Allow SSH connections on port 22 only from pre-defined addresses. This will help us prevent brute force attacks and any other unwanted traffic.

Here is the configuration:
```
resource "hcloud_firewall" "cloud_firewall" {
  name = "k8s_cluster_firewall"

  rule {
    description = "PING"
    direction   = "in"
    protocol    = "icmp"
    source_ips  = [
      "0.0.0.0/0"
    ]
  }

  rule {
    description = "SSH"
    direction   = "in"
    protocol    = "tcp"
    port        = "22"
    source_ips = concat(
      var.firewall_allowed_ips,
      [for ip in hcloud_primary_ip.primary_ips : ip.ip_address]
    )
  }
}
```

Note that `source_ips` is declared using a `concat` function. This function concatenates 2 lists with IP addresses. 1st list is taken from `terraform.tfvars` file, and the 2nd list will contain addresses of the virtual machines after they are created. This ensures that SSH connections between VMs will also be possible.

#### main.tf

Finally, the file where the main deployment logic is defined. Let's go through it piece by piece. First the provider is declared with the token variable:
```
provider "hcloud" {
  token   = var.hcloud_token
}
```

Then we tell Terraform to find our SSH key and add it to project settings. This key will be copied to virtual machines after their creation:
```
resource "hcloud_ssh_key" "default" {
  name       = "hetzner_key"
  public_key = file(var.ssh_pub_key)
}
```

Then, for each VM a primary IP is defined. This step is not strictly required, but it has certain benefits. For example if a VM is accidentally deleted and it doesn't have a primary IP, after it is recreated IP address might be different which could break compatibility of the existing services.

Here is the code:
```
resource "hcloud_primary_ip" "primary_ips" {
  count         = var.instances
  name          = "primary-ip-${count.index}"
  type          = "ipv4"
  assignee_type = "server"
  datacenter    = var.datacenter
  auto_delete   = false
}
```

In the last part we are creating virtual machines:
```
resource "hcloud_server" "nodes" {
  count         = var.instances
  name          = "node-${count.index}"
  image         = var.os_type
  server_type   = var.server_type
  location      = var.location

  firewall_ids  = [hcloud_firewall.cloud_firewall.id]
  ssh_keys      = [hcloud_ssh_key.default.id]

  public_net {
    ipv4 = hcloud_primary_ip.primary_ips[count.index].id
    ipv4_enabled = true
    ipv6_enabled = false
  }

  labels = {
    type = "k8s-node"
  }
```

## Deployment

With all prerequisites met and Terraform files ready, it's time to deploy!

### Applying Terraform Configuration

Execute the command `terraform init` within the project directory to initialize Terraform and download the necessary provider plugins.

Then run `terraform plan` to preview the changes Terraform will implement based on the configuration files. This ensures that the configurations align with your expectations before committing to the changes.

To apply the configuration after verifying the plan, run `terraform apply`. Review the command output to track the progress of the resource creation. Ensure that all resources, namely Hetzner servers, primary IPs and firewall, are provisioned correctly.

### Verifying the Deployment
At this point VMs should be visible in the Hetzner Cloud UI, but below I'll show how to get the information about them using Terraform.  

To list the currently existing resources, run:
```
➜  terraform-hz-k8s git:(main) ✗ terraform state list
hcloud_firewall.cloud_firewall
hcloud_primary_ip.primary_ips[0]
hcloud_primary_ip.primary_ips[1]
hcloud_primary_ip.primary_ips[2]
hcloud_primary_ip.primary_ips[3]
hcloud_primary_ip.primary_ips[4]
hcloud_server.nodes[0]
hcloud_server.nodes[1]
hcloud_server.nodes[2]
hcloud_server.nodes[3]
hcloud_server.nodes[4]
hcloud_ssh_key.default
```

Then, using `terraform state show <resource-ID>.<object>` we can retrieve information about the resources. Let's get the assigned IP address of the 5th VM:
```
➜  terraform-hz-k8s git:(main) terraform state show 'hcloud_primary_ip.primary_ips[4]'
# hcloud_primary_ip.primary_ips[4]:
resource "hcloud_primary_ip" "primary_ips" {
    assignee_id       = 0
    assignee_type     = "server"
    auto_delete       = false
    datacenter        = "nbg1-dc3"
    delete_protection = false
    id                = "88xxxxxx"
    ip_address        = "167.235.xxx.xxx"
    name              = "primary-ip-4"
    type              = "ipv4"
}
```

To get all IP addresses, we can query this information using a for loop:
```
➜  terraform-hz-k8s git:(main) for i in $(seq 0 4); do terraform state show "hcloud_primary_ip.primary_ips[$i]" | grep ip_address | awk '{print $3}' | sed 's/"//g'; done
91.99.xxx.xxx
91.99.xxx.xxx
116.203.xxx.xxx
49.12.xxx.xxx
167.235.xxx.xxx
```

or run the `show_ips.sh` script in the repo, it will read the instance count from `vars.tf` and display node ID and IP:
```
➜  terraform-hz-k8s git:(main) bash show_ips.sh
node-0,91.99.xxx.xxx
node-1,91.99.xxx.xxx
node-2,116.203.xxx.xxx
node-3,49.12.xxx.xxx
node-4,167.235.xxx.xxx
```

Finally, let's try to connect to a host via SSH with the key you definted in `terraform.tfvars`:
```
➜  terraform-hz-k8s git:(main) ssh -i ~/.ssh/key_for_hz_vms.pub root@49.12.xxx.xxx
Last login: Sat May 10 11:53:51 2025 from xxx.xxx.xxx.xxx
[root@node-3 ~]#
```

