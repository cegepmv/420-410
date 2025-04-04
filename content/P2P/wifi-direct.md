+++
title = 'Wifi Direct'
date = 2025-04-04T07:17:23-04:00
draft = true
weight = 61
+++

Le Wi-Fi Direct est une technologie qui permet à des appareils de se connecter entre eux sans avoir besoin d’un routeur ou d’un point d’accès sans fil. 

Elle est similaire à Bluetooth car elle permet une communication "Peer-to-Peer", cependant elle offre une portée beaucoup plus grande et des débits beaucoup plus élevés.

Wifi direct est en mode client-serveur, il faut configurer le RaspberryPi en mode serveur, ensuite un ou plusieurs clients pourront s'y connecter comme s'il s'agissait d'un réseau Wifi normal.

## Installation des logiciels
Les paquets suivants seront utilisés:
- **hostapd**: le service qui fournit le point d'accès Wifi
- **dnsmasq**: le service qui fournit les adresses IP aux clients
- **dhcpcd**: le service qui permet de configurer localement les paramètres réseau du Pi. On pourrait utiliser *Network Manager* (qui est déjà installé), mais *dhcpcd* est un peu plus simple. 

```
sudo apt update
sudo apt install hostapd dnsmasq dhcpcd5
```

Lancer le service:
```
sudo systemctl unmask hostapd
```

sudo mv /etc/dhcpcd.conf /etc/dhcpcd.conf.backup
sudo nano /etc/dhcpcd.conf:
interface wlan0
    static ip_address=10.10.1.1/24
    nohook wpa_supplicant


sudo nano /etc/dnsmasq.conf:
interface=wlan0
dhcp-range=10.10.1.2,10.10.1.50,255.255.255.0,24h


sudo nano /etc/hostapd/hostapd.conf:
interface=wlan0
driver=nl80211
ssid=NOM_DU SSID
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=abcd-1234
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP


sudo nano /etc/default/hostapd:
DAEMON_CONF="/etc/hostapd/hostapd.conf"


sudo nano /etc/resolv.conf:
nameserver 8.8.8.8
nameserver 8.8.4.4


EN LOCAL!!
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
sudo systemctl enable dhcpcd
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq
sudo systemctl start dhcpcd
sudo systemctl start hostapd
sudo systemctl start dnsmasq

Tester:
- ip a devrait montrer que l'adresse de wlan0 est changée
- 