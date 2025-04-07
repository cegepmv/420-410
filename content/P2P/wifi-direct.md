+++
title = 'Wifi'
date = 2025-04-04T07:17:23-04:00
draft = false
weight = 61
+++

Une manière simple d'établir une connexion P2P entre deux Pi est d'en configurer un pour être un point d'accès. Il suffit ensuite qu'un autre Pi (ou un PC, ou un téléphone) se connecte au réseau créé par le point d'accès. Le gestionnaire de connexions *NetworkManager* déjà installé sur le Pi est l'utilitaire qui permet cette configuration.

Attention, le Pi qui devient un point d'accès ne peut donc plus accéder à internet par wifi.

Aussi, il est toujours utile d'avoir le service DHCP actif sur un réseau: ceci permet d'éviter aux clients de définir leur propre adresse IP statique. Pour cela nous installerons le service *dnsmasq*. 

Il faudra également donner une adresse IP statique au Pi qui sera le point d'accès.

## Configuration *NetworkManager*
Certains paramètres de *NetworkManager* sont différents selon qu'on soit client wifi ou point d'accès. Il faut donc modifier la configuration de ce service.

#### Modification du fichier de configuration
Pour ne pas perdre la configuration client, faites une copie du fichier de configuration général:
```bash
sudo cp /etc/NetworkManager/NetworkManager.conf /etc/NetworkManager/NetworkManager.conf.sauv
```
Ensuite modifiez `/etc/NetworkManager/NetworkManager.conf` pour qu'il contienne les valeurs suivantes:
```conf
[main]
plugins=keyfile
dns=none

[ifupdown]
managed=true
```

#### Création du point d'accès
La configuration de *NetworkManager* se fait à l'aide de l'utilitaire `nmcli`. Il y a plusieurs paramètres à définir et ils sont passés comme arguments à la commande.

Pour créer la connection (remplacez les valeurs entre crochets par les vôtres):
```bash {wrap="false"}
nmcli connection add type wifi ifname wlan0 mode ap con-name [NOM_CONNEXION] ssid [NOM_RÉSEAU_WIFI]
```
> Si la connexion a été créée, la commande `nmcli connection show` devrait l'afficher parmi toutes les connexions existantes.

Il faut ensuite définir les paramètres de la connexion avec `nmcli connection modify...`:
```bash
sudo nmcli connection modify [NOM_CONNEXION] 802-11-wireless.band bg
sudo nmcli connection modify [NOM_CONNEXION] 802-11-wireless.channel 6
sudo nmcli connection modify [NOM_CONNEXION] wifi-sec.key-mgmt wpa-psk
sudo nmcli connection modify [NOM_CONNEXION] wifi-sec.psk "abcd-1234"
```
> Suite à ces commandes, un fichier portant le nom de la connexion devrait avoir été créé dans le répertoire `/etc/NetworkManager/system-connections`.

#### Définition de l'adresse IP
Pour donner une adresse IP statique au point d'accès, la commande est celle-ci:
```bash
nmcli connection modify [NOM_CONNEXION] ipv4.method manual ipv4.addresses [ADRESSE/MASQUE]
```
<!--
Ces deux lignes semblent inutiles:
sudo nmcli connection modify ap_fruit ipv4.gateway 10.10.1.1
sudo nmcli connection modify ap_fruit ipv4.dns 8.8.8.8
-->

## Configuration du service DHCP
Le service *dnsmasq* est très léger et permet de remplir des fonctionnalités de cache DNS et de serveur DHCP de base. Nous allons donc l'utiliser.

#### Installation
```bash
sudo apt update
sudo apt install dnsmasq
```

#### Définition de la plage d'adresses
Il faut définir principalement deux paramètres: la **plage d'adresses** dans lesquelles le serveur peut sélectionner les adresses qu'il distribue, et l'**interface** sur laquelle il doit attendre les requêtes DHCP.

Il faut donc créer un fichier qui contient ces informations; nous lui donnerons le même nom que la connexion. Créez donc le fichier `/etc/dnsmasq.d/[NOM_CONNEXION].conf` et mettez-y le contenu suivant:
```conf
interface=wlan0
dhcp-range=[ADRESSE_IP_MIN],[ADRESSE_IP_MAX],12h
domain-needed
bogus-priv
```

## Activation du point d'accès
Tous les services sont maintenant configurés, il reste donc à les (re)démarrer. 

La premère étape est d'arrêter la connexion *wifi client*:
```bash
sudo nmcli connection down preconfigured
```

Ensuite on démarre le connexion *point d'accès* (il est probable qu'elle démarre toute seule):
```bash
sudo nmcli connection up [NOM_CONNEXION]
```

Enfin on démarre le service DHCP:
```bash
sudo systemctl restart dnsmasq
```

## Désactivation du point d'accès
Pour arrêter le point d'accès et revenir à un état de client wifi, les étapes inverses doivent être réalisées, soit:
- Revenir au fichier de configuration d'origine de *NetworkManager*
- Désactiver la connexion du point d'accès
- Réactiver la connexion de client wifi

Les commandes sont les suivantes:
```bash {wrap="false"}
sudo cp /etc/NetworkManager/NetworkManager.conf /etc/NetworkManager/NetworkManager.conf.point_acces
sudo cp /etc/NetworkManager/NetworkManager.conf.sauv /etc/NetworkManager/NetworkManager.conf
sudo nmcli connection down [NOM_CONNEXION]
sudo nmcli connection up preconfigured
sudo systemctl restart NetworkManager
```