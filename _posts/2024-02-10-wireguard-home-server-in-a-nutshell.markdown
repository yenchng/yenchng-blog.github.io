---
layout: post
title:  "Wireguard Home Server in a Nutshell"
date:   2024-02-10 13:33:33 +0800
categories: test
---

# Installation

```
sudo apt update
sudo apt install wireguard
```

# Server Set Up

## Generate Keys, Set Perms
```
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key

sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

## Setup Config
```
sudo nano /etc/wireguard/wg0.conf
```

## Enable Internet via VPN
``sudo nano /etc/sysctl.conf``

add to bottom of config
```
net.ipv4.ip_forward=1
```

check and confirm
``sudo sysctl -p``




[References]
https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04#step-4-adjusting-the-wireguard-server-s-network-configuration
https://wireguard.how/client/ios/
https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4
https://www.wireguardconfig.com
