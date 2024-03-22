---
layout: post
title:  "Wireguard Home Server in a Nutshell"
date:   2024-02-10 13:33:33 +0800
categories: test
---

## Installation

```
sudo apt update
sudo apt install wireguard
sudo apt install qrencode
```

## Server Set Up

### Generate Keys, Set Perms
```
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key

sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

### Enable Internet via VPN
Configure internet forwarding via VPN ``sudo nano /etc/sysctl.conf`` then add below code to bottom of configuration:
```
net.ipv4.ip_forward=1
```

check and confirm
``sudo sysctl -p``

### Setup Config with Firewall Rules
Open server configuration file and edit it via ``sudo nano /etc/wireguard/wg0.conf``

```
[Interface]
Address = 10.8.0.1/24
SaveConfig = true
PostUp = ufw route allow in on wg0 out on eno1
PostUp = iptables -t nat -I POSTROUTING -o eno1 -j MASQUERADE
PostUp = iptables -I FORWARD -i wg0 -o wg0 -j REJECT
PostUp = iptables -I FORWARD -i wg0 -s 10.8.0.101/32 -d 10.8.0.0/24 -j ACCEPT
PreDown = ufw route delete allow in on wg0 out on eno1
PreDown = iptables -t nat -D POSTROUTING -o eno1 -j MASQUERADE
PreDown = iptables -D FORWARD -i wg0 -o wg0 -j REJECT
PreDown = iptables -D FORWARD -i wg0 -s 10.8.0.101/32 -d 10.8.0.0/24 -j ACCEPT
ListenPort = 51820
PrivateKey = <INSERT YOUR SERVER PRIVATE KEY FROM private.key file>
```

Notes: ``iptables -I FORWARD -i wg0 -o wg0 -j REJECT`` prevents child nodes from seeing each other whereas ``iptables -I FORWARD -i wg0 -s 10.8.0.101/32 -d 10.8.0.0/24 -j ACCEPT`` specifically whitelists .101 node to be able to view other nodes. -D is just the inverse (delete) during shutdown. Also, need to check at client side's firewall in case if it blocks all traffic by default - then add an entry to grant permission to remote IP (i.e. 10.8.0.199) as needed.

### Firewall Port Forwarding
Allow Wireguard traffic through your selected port (default 51820) and restart firewall
```
sudo ufw allow 51820/udp
sudo ufw disable
sudo ufw enable
sudo ufw status
```

### Setup Wireguard as ``systemd`` service via ``wg-quick``

```
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
sudo systemctl status wg-quick@wg0.service
```
note: ``wg0`` is the service name which should correspond to my ``wg0.conf`` file name


## Client Set Up (iOS)

### Generate client keys
First, on the server side, prep the server public key via ``sudo wg show wg0``

Generate client keys from the server and print it to check.
```
(umask 077 && wg genkey > wg-private-client-1.key)
wg pubkey < wg-private-client-1.key > wg-public-client-1.key
cat wg-private-client-1.key && cat wg-public-client-1.key
```

### Create client config
create a client config file e.g. ``nano ~/wg-client-1.conf``
```
# define the local WireGuard interface (client)
[Interface]

# contents of wg-private-client.key
PrivateKey = <INSERT YOUR CLIENT PRIVATE KEY>

# the IP address of this client on the WireGuard network
Address=10.8.0.101/32

# define the remote WireGuard interface (server)

# DNS Resolver by our internal DNS server/router's IP address (e.g. 192.168.1.1)
DNS=<INSERT DNS IP>

[Peer]

# from `sudo wg show wg0 public-key`
PublicKey = <INSERT YOUR SERVER PUBLIC KEY>

# the IP address of the server on the WireGuard network
AllowedIPs = 0.0.0.0/0, 192.168.0.0/24

# public IP address and port of the WireGuard server (e.g. dns.example.com:51820)
Endpoint = <INSERT YOUR PUBLIC SERVER DOMAIN:PORT>

# Behind NAT and to keep connection alive
PersistentKeepalive = 25
```

### Add client to Server's peer list
Print client's public key via ``cat wg-public-client-1.key``

Edit server configuration file via ``sudo nano /etc/wireguard/wg0.conf`` and add ``[Peer]`` section at the bottom
```
[Peer]
PublicKey = <INSERT YOUR CLIENT PUBLIC KEY>
AllowedIPs = <DESIRED WIREGUARD CLIENT IP ADDRESS WITHIN SAME SUBNET & SPECIFIED IN CLIENT FILE e.g. 10.8.0.101/32>
```

Sync latest config change by restarting service.


### Print Client QR and setup on mobile
```echo "My VPN QR Code" && qrencode --read-from=wg-client-1.conf --type=UTF8```

## Automation with script
Simplified deployment of client so you don't have to do it manually every time

### Create script
Create the file i.e. ``nano script.sh``

### Modify Permission to make it executable
Run ``chmod +x script.sh``

### Open the script
Run ``nano script.sh`` and edit it, add below code, and modify it accordingly as needed
```
#!/bin/bash

while true; do
	read -p "Enter the user's name: " vpnusername
	read -p "Enter the device's type: " vpndevicetype
    read -p "Enter a number (0-255): " num
    if [[ $num =~ ^[0-9]+$ && $num -ge 0 && $num -le 255 ]]; then
        echo "Valid input: $num"
		
		# Generate the Keys
		(umask 077 && wg genkey > wg-private-client-$num.key)
		wg pubkey < wg-private-client-$num.key > wg-public-client-$num.key
		cat wg-private-client-$num.key && cat wg-public-client-$num.key
		
		# Generate text file with predefined text
		cat <<EOF > "wg-client-$num.conf"
#############################################
##   $vpnusername $vpndevicetype   
#############################################
# define the local WireGuard interface (client)
[Interface]

# contents of wg-private-client.key
PrivateKey = $(<"wg-private-client-$num.key")

# the IP address of this client on the WireGuard network
Address=10.8.0.$num/32

# define the remote WireGuard interface (server)

# DNS Resolver by our internal DNS server/router's IP address (e.g. 192.168.1.1)
DNS=<INSERT DNS IP>

[Peer]

# from `sudo wg show wg0 public-key`
PublicKey = <INSERT YOUR SERVER PUBLIC KEY>

# the IP address of the server on the WireGuard network
AllowedIPs = 0.0.0.0/0, 192.168.0.0/24

# public IP address and port of the WireGuard server (e.g. dns.example.com:51820)
Endpoint = <INSERT YOUR PUBLIC SERVER DOMAIN:PORT>

# Behind NAT and to keep connection alive
PersistentKeepalive = 25
EOF
		
		echo "File 'wg-client-$num.conf' has been saved."
		
		
		# Update Server Config?
		echo "Edit server configuration file via sudo nano /etc/wireguard/wg0.conf and add [Peer] section at the bottom"

		# Output the lines in a nice format
        cat <<EOF

[Peer]
PublicKey = $(<"wg-public-client-$num.key")
AllowedIPs = 10.8.0.$num/32

EOF
		
		while true; do
			read -p "Do you want me to add it for you? (Y/N): " choice
			if [[ $choice == "Y" || $choice == "y" ]]; then
				# Backup /etc/wireguard/wg0.conf
				sudo cp /etc/wireguard/wg0.conf ~/backups/server/wg0.bak.$(date +%Y%m%d%H%M%S)

				# Edit /etc/wireguard/wg0.conf
				sudo bash -c "cat <<EOF >> /etc/wireguard/wg0.conf

[Peer]
PublicKey = $(<"wg-public-client-$num.key")
AllowedIPs = 10.8.0.$num/32
EOF"

				echo "File /etc/wireguard/wg0.conf has been updated."
				break
			elif [[ $choice == "N" || $choice == "n" ]]; then
				echo "Skipping Server Config Edit..."
				break
			else
				echo "Please enter Y or N."
			fi
		done
		
		# Generate QR Code?
		echo "To simplify setup on mobile, I recommend you to Print Client QR so that you can scan it"

		while true; do
			read -p "Do you want me to generate QR Code? (Y/N): " choice
			if [[ $choice == "Y" || $choice == "y" ]]; then
				echo "QR Code for $vpnusername $vpndevicetype" && qrencode --read-from=wg-client-$num.conf --type=UTF8
				break
			elif [[ $choice == "N" || $choice == "n" ]]; then
				echo "Skipping QR Code..."
				break
			else
				echo "Please enter Y or N."
			fi
		done
		
		# End
		echo "We're at the end of program! Terminating..."
		
        break
    else
        echo "Invalid input. Please enter a number within the range of 0-255."
    fi
done
```
### Run the Script
``./script.sh``

## Quick Commands

Start/stop interface
```
sudo wg-quick up wg0
sudo wg-quick down wg0
```

Check Status
```
sudo wg show
```

Start/stop service (preferred)
```
sudo systemctl stop wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
sudo systemctl status wg-quick@wg0.service
```


## References
- https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04#step-4-adjusting-the-wireguard-server-s-network-configuration
- https://wireguard.how/client/ios/
- https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4
- https://www.wireguardconfig.com
- https://github.com/cyyself/wg-bench
- https://www.lautenbacher.io/en/lamp-en/wireguard-prohibit-communication-between-clients-client-isolation/
