---
title: Setting up a VPS on DigitalOcean with Terraform and Tailscale
date: 2025-02-15
---

In this guide, I'll walk you through deploying a secure virtual private server (VPS) on DigitalOcean using Terraform for infrastructure as code and Tailscale for private networking. This approach combines the convenience of cloud infrastructure with enhanced security through private networking.

## Setting Up Terraform for DigitalOcean

First create a `secrets.auto.tfvars` file securely store your DigitalOcean access token and SSH key path.

```sh
do_token = "digital_ocean_access_token"
pvt_key = "~/.ssh/your_private_key"
```

Next, create a `provider.tf` file to configure the DigitalOcean provider in terraform. 

```terraform
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

variable "do_token" {
  description = "DigitalOcean Personal Access Token"
  sensitive   = true
}

variable "pvt_key" {
  description = "Path to your private SSH key"
}

provider "digitalocean" {
  token = var.do_token
}

data "digitalocean_ssh_key" "terraform" {
  name = "key_name_in_do" # Replace with SSH key name in DigitalOcean
}
```

Initialize your Terraform working directory:

```sh
terraform init
```
    
## Creating the DigitalOcean Droplet

Create a  `droplet.tf` file to define your server configuration and provisioning. Add a SSH connection so that you're able to provision, and add the path the scripts used for provisioning.
```terraform
resource "digitalocean_droplet" "droplet" {
  image    = "ubuntu-20-04-x64"
  name     = "droplet-1"
  region   = "ams3"
  size     = "s-1vcpu-512mb-10gb"
  backups  = false
  ssh_keys = [data.digitalocean_ssh_key.terraform.id]

  # SSH connection for provisioning
  connection {
    host        = self.ipv4_address
    user        = "root"
    type        = "ssh"
    private_key = file(var.pvt_key)
    timeout     = "2m"
  }

  provisioner "remote-exec" {
    scripts = [
      "./scripts/update.sh",
      "./scripts/tailscale.sh"
    ]
  }
```

## Setting Up Provisioning Scripts

Create a `scripts` directory with the following two scripts files: 

`./scripts/update.sh` will update the system packages, without the need for user interaction.

```sh
#!/usr/bin/bash
sudo apt update
echo 'openssh-server openssh-server/sshd_config boolean true' | sudo debconf-set-selections
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y
echo "System packages updated successfully."
```

`./scripts/tailscale.sh` will install tailscale. 

> Note that `tailscale up` will request to authenticate and add the machine to your account by visiting and url.

```sh
#!/usr/bin/bash
sudo curl -fssl https://tailscale.com/install.sh | sh
echo "Starting Tailscale..."
sudo tailscale up
echo "Tailscale installed and running. Note the Tailscale IP from above."
```

## Configuring Security with Tailscale and Firewall

Create a `firewall.tf` to define your firewall rules. We want only allow the traffic from the tailscale network, and block all the other incoming traffic.

Read more about what ports ports tailscale uses [here](https://tailscale.com/kb/1082/firewall-ports).

```terraform
resource "digitalocean_firewall" "firewall" {
  name = "tailscale"

  droplet_ids = [digitalocean_droplet.droplet.id]

  # Tailscale required ports
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

  # Allow all outbound traffic
  outbound_rule {
    protocol              = "tcp"
    port_range            = "1-65535"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }

  outbound_rule {
    protocol              = "udp"
    port_range            = "1-65535"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }
}
```

## Deploying Your Infrastructure

Run Terraform to create your infrastructure:

```sh
terraform plan    # Preview the changes
terraform apply   # Apply the changes
```

## Connecting to Your Server

After deployment, you'll need to connect to your server using its Tailscale IP rather than its public IP. This provides an additional layer of security, as your SSH port is only accessible through the private Tailscale network.

```sh
ssh -i ~/.ssh/your_private_key root@100.x.y.z  # Replace with your Tailscale IP
```

- [https://tailscale.com/kb/1082/firewall-ports](https://tailscale.com/kb/1082/firewall-ports)
- [https://www.digitalocean.com/community/tutorials/how-to-use-terraform-with-digitalocean](https://www.digitalocean.com/community/tutorials/how-to-use-terraform-with-digitalocean)
- [https://docs.digitalocean.com/products/droplets/getting-started/recommended-droplet-setup](https://docs.digitalocean.com/products/droplets/getting-started/recommended-droplet-setup)
- [https://sergeykibish.com/blog/tailscale-based-vpn-on-digitalocean-droplet](https://sergeykibish.com/blog/tailscale-based-vpn-on-digitalocean-droplet)
