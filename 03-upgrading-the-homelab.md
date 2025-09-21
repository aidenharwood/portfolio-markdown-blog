---
title: Homelab Upgrades - New Hardware!
date: 09-17-2025
slug: homelab-upgrades-new-hardware
excerpt: Moving the homelab from an old laptop to a dedicated machine
---

# The New Hardware

With the laptop hosting my homelab k8s cluster being relatively weak even when it was new, the need for beefier hardware became apparent quite quickly - an older mid-range Pentium is pretty restrictive even with (the maxiumum possible) 16GB RAM. Luckily, a recent rig upgrade left me with an unused i7-4790k (which after over a decade is still holding up against modern titles!) and 32GB RAM.

This allows much better flexibility; I can actually use the laptop for "laptop things" now, and host the homelab on a dedicated OS (Ubuntu Server 24.04). Without having to worry about the laptop turning into a spicy pillow under the desk (due to being plugged in and running under load 24/7).


# The Cluster Grows

The single-node k0s setup has now been replaced with a small [multipass](https://canonical.com/multipass) VM cluster running [k3s](https://k3s.io/). This means that the cluster is now compartmentalised to a series of VMs rather than directly on the host.

This was made possible with ease thanks to [k3m](https://github.com/eznix86/k3m)
> **Note:** k3m doesn't appear to separate control-plane and worker nodes, use with caution

**NB:** This has now actually been revered to a running k3s directly on the host, due to multi-node/VM clusters being severely limited by IOPS (the host only has a HDD for the time being, this will change once I have an SSD installed)

# Additions
## VPN
[WireGuard VPN](https://www.wireguard.com/) is also configured on the server; this allows for two main things: Remote access to the homelab, and ad-blocking which is already set up on my home network via [OpenWRT](https://openwrt.org/)

## Living Room Integration
The homelab is now connected to the living room TV, leaving room for further projects such as media center and streaming capabilities