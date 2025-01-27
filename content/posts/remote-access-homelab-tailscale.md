---
title: Remote Access to Homelab using Tailscale 
date: 2024-08-18
tags: cloud
---

For the last year I've been running a small Homelab on a Raspberry Pi 5. It runs a [Plex Media Server](https://plex.tv/), [Home Assistant](https://home-assistant.io), web servers and some Docker images.

Until now my Homelab was only accessible from my local network.

I was hesitant in making my Homelab remotely accessible (port forwarding , setting up my own VPN). Even though I do a lot of programming and software engineering related things, I'm not too comfortable working with networking side of things.

I don't want someone unauthorized having access to my Homelab...

But I still, it would be nice to have access to it when I'm not on my local network. This is when I came across [Tailscale](https://tailscale.com).

## Tailscale

The Tailscale VPN allows me to access my Homelab, and the applications running on it, from anywhere in the world.

So on my Raspberry Pi I ran the following.

```bash
sudo apt-get install apt-transport-https
```

Then we need to add the Tailscale singin key.

```bash
curl -fsSL https://pkgs.tailscale.com/stable/raspbian/bullseye.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg > /dev/null
curl -fsSL https://pkgs.tailscale.com/stable/raspbian/bullseye.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
```

Install Tailscale.

```bash
sudo apt update
sudo apt install tailscale
```

You can then run the following command to start Tailwind and authenticate.

```bash
sudo tailscale up
```

In the [Tailscale Dashboard](https://login.tailscale.com/admin/machines) you can see all your machines and there IP addresses.

Now add other machines to the same account, and as long as you're connected to the Tailscale VPN can access your machines remotely.

## Links

https://tailscale.com/learn/how-to-ssh-into-a-raspberry-pi

[[homelab]] [[vpn]]
