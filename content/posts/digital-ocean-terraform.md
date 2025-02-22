---
title: Deploy VPS on Digital Ocean using Terraform
date: 2025-02-15
tags: cloud
---

This post demonstrates how to setup a virtual private server on Digital Ocean using terraform and tailscale.

## Configuring Terraform for DigitalOcean

To deploy on DigitalOcean using Terraform you will need a Personal Access Token. This allows terraform you manage resources on DigitalOcean. Set this secret in the `secrets.auto.tfvars`. Also add the path to my ssh key so that.

```sh
do_token = ""
pvt_key = "~/.."
```

First we need to specify DigitalOcean provider and configure credentials in the `provider.tf`. We define some variables for your Personal Access Token and private key, which we reference on runtime.

```terraform
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

variable "do_token" {}
variable "pvt_key" {}

provider "digitalocean" {
  token = var.do_token
}
```

Finally, I want Terraform automatically add my SSH key to my resource. Provide the name of the SSH key you added to DigitalOcean.

```terraform
data "digitalocean_ssh_key" "terraform" {
  name = "key_name_in_do"
}
```

Initialize terraform by running:

```sh
terraform init
```
    
## Defining the Server

We can now create a Droplet using terraform and install software on it. In this example I will use the smallest instance and install tailscale on it.

Create a new terraform configuration file called `droplet.tf`, which will hold the configuration for the Droplet:


```terraform
resource "digitalocean_droplet" "droplet" {
  image    = "ubuntu-20-04-x64"
  name     = "droplet-1"
  region   = "ams3"
  size     = "s-1vcpu-512mb-10gb"
  backups  = false
  ssh_keys = [data.digitalocean_ssh_key.terraform.id]
```

We now need to setup an connection using SSH that terraform can use. Add the following lines to  droplet resource.

```terraform
  connection {
    host        = self.ipv4_address
    user        = "root"
    type        = "ssh"
    private_key = file(var.pvt_key)
    timeout     = "2m"
  }
```

After the connection setup, we can use the `remote-exec` provisioner to upgrade packages and install tailscale.

```terraform
  provisioner "remote-exec" {
    scripts = [
      "./scripts/update.sh",
      "./scripts/tailscale.sh"
    ]
  }
```

The `./scripts/update.sh` and `./scripts/tailscale.sh` scripts respectively.

```sh
#!/usr/bin/bash
sudo apt update
echo 'openssh-server openssh-server/sshd_config boolean true' | sudo debconf-set-selections
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y
```

```sh
#!/usr/bin/bash
sudo curl -fssl https://tailscale.com/install.sh | sh
sudo tailscale up
```

## Hardening with tailscale and firewall

We now want create firewall rule which will only accept traffic from `tailscale` network in `firewall.tf`.

```terraform
resource "digitalocean_firewall" "firewall" {
  name = "tailscale"

  droplet_ids = [digitalocean_droplet.droplet.id]

  inbound_rule {
    protocol         = "udp"
    port_range       = "3478"
    source_addresses = ["100.64.0.0/10"]
  }

  inbound_rule {
    protocol         = "udp"
    port_range       = "41641"
    source_addresses = ["100.64.0.0/10"]
  }

  outbound_rule {
    protocol              = "tcp"
    port_range            = "1-65535"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }
}
```

You will not be able to connect to the machine via SSH using normal IP. But you will be able to SSH over `tailscale` IP.

## Links

- [https://www.digitalocean.com/community/tutorials/how-to-use-terraform-with-digitalocean](https://www.digitalocean.com/community/tutorials/how-to-use-terraform-with-digitalocean)
- [https://docs.digitalocean.com/products/droplets/getting-started/recommended-droplet-setup](https://docs.digitalocean.com/products/droplets/getting-started/recommended-droplet-setup)
- [https://sergeykibish.com/blog/tailscale-based-vpn-on-digitalocean-droplet](https://sergeykibish.com/blog/tailscale-based-vpn-on-digitalocean-droplet)
