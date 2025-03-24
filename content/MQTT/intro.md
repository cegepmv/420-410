+++
title = 'Description'
date = 2024-03-13T09:49:18-04:00
draft = false
weight = 51
+++

Le protocole MQTT, pour _Message Queuing Telemetry Transport_, est un protocole conçu pour les objets connectés: il est simple à mettre en place et à utiliser et consomme peu de ressources et de bande passante. Il permet une communication facile entre un ensemble d'objets connectés et les différentes applications qui les utilisent. 

## Modèle de communication
En MQTT, les objets connectés et les applications sont des **clients**; il se connectent à un **serveur** (aussi appelé "broker" ou _agent_) qui gère la communication entre eux:

![mqttschema](/420-410/images/mqttschema.png)

Les clients peuvent émettre des données ou encore recevoir des données. Lorsqu'ils génèrent des données, ils envoient un message **publish** à l'agent; lorsqu'ils souhaitent reçevoir des données, ils envoient un message **subscribe** à l'agent, et celui-ci leur enverra les données à mesure qu'elles seront publiées par les autres clients.

Les messages envoyés et reçus doivent être identifiés par une **rubrique** (ou "topic"). Un client qui s'abonne à une rubrique ne recevra que les messages envoyés à cette rubrique.

Imaginez par exemple une application web qui affiche des données environnementales (température, humidité, luminosité, etc.) lues sur des senseurs situés à différents endroits dans votre maison. Les contrôleurs (RaspberryPi, Arduino, etc.) sur lesquels sont connectés les senseurs publieront des données à intervalles réguliers en les envoyant à l'agent sur la rubrique "maison". Votre application contactera l'agent pour s'abonner à la rubrique "maison", et recevra ainsi toutes les données dont elle a besoin, sans avoir besoin de se connecter individuellement sur chaque contrôleur.

## Quelques mots sur les _topics_
Les rubriques MQTT sont représentées par des chaînes de caractères et peuvent être organisées en hiérarchie. Par exemple on peut avoir une rubrique nommée "maison", mais aussi "maison/salon", "maison/chambre1", "maison/chambre1/luminosité", etc.

#### Caractères spéciaux
Les caractères `+` et `#` peuvent être utilisés dans les désignations de rubriques.

Imaginez une maison entièrement connectée où on utilise les rubriques MQTT suivantes:
+ `maison/salon/temp`
+ `maison/salon/lumi`
+ `maison/salon/bruit`
+ `maison/cuisine/temp`
+ `maison/cuisine/lumi`
+ `maison/cuisine/bruit`

Le caractère `+` peut remplacer un niveau afin de s'abonner à des sous-rubriques communes à plusieurs niveaux. Par exemple, si un client s'abonne à `maison/+/lumi`, il recevra les messages de `maison/salon/lumi` et `maison/cuisine/lumi`.

Le caractère `#` ne peut être utilisé qu'à la fin d'une rubrique, et permet de s'abonner à toutes les sous-rubriques. Par exemple, un client qui s'abonne à `maison/salon/#` recevra les messages de `maison/salon/temp`, `maison/salon/lumi` et `maison/salon/bruit`.

{{% notice primary "Attention!" %}}
Les caractères `#` et `+` ne peuvent pas être utilisés pour publier des messages. On ne peut s'en servir que dans les abonnements.
{{% /notice %}}

#### $SYS
La plupart des implémentations de MQTT ont une rubrique `$SYS` qui permet d'accéder à des informations sur l'agent. Par exemple, `$SYS/broker/clients/maximum` donne le nombre maximum de clients connectés; `$SYS/broker/publish/messages/received` donne le nombre de messages publiés par des clients sur ce broker; etc.

Il n'est pas possible de publier sous cette rubrique.

## QoS
_QoS_, pour "Quality of Service", est un terme utilisé en informatique pour désigner des contraintes sur la distribution des messages sur un réseau. Par exemple, on peut définir des règles de QoS sur les routeurs dans un réseau afin de donner une priorité plus grande aux paquets qui contiennent de la vidéo en streaming; ou encore on peut limiter la bande passante utilisée par des paquets du protocole bitTorrent; etc.

En MQTT, il y a 3 niveaux de QoS. Chaque niveau désigne les caractéristiques reliées au traitement des messages; plus précisément, au niveau de fiabilité de la transmission:
+ `0`: basse ("Fire and Forget")
+ `1`: moyenne ("Acknowledge Deliver")
+ `2`: haute ("Assured Delivery")

Un client qui envoit un message avec un QoS de 0 ne reçoit aucun accusé de réception.

Un client qui envoit un message envoyé avec un QoS de 1 reçoit un accusé de réception.

Un client qui envoit un message envoyé avec un QoS de 2 reçoit un accusé de réception, et une confirmation lorsque le message est livré aux abonnés.

La valeur du QoS est déterminée lors de l'_envoi_ d'un message (côté émetteur) et lors de l'_abonnement_ (côté récepteur).

#### Cas d'utilisation
+ **QoS 0**: À utiliser lorsque la connexion est assez stable ou que la perte occasionnelle de messages n'est pas un problème.
+ **Qos 1**: À utiliser lorsqu'on souhaite avoir tous les messages, mais qu'on accepte d'avoir des messages répétés à l'occasion.
+ **QoS 2**: À utiliser lorsqu'on souhaite avoir tous les messages et qu'on ne veut pas avoir de messages qui se répètent.

## Programmes en ligne de commande
Les utilitaires **mosquitto_sub** et **mosquitto_pub** permettent d'utiliser le protocole mosquitto à partir de la ligne de commande linux. Pour les installer:
```bash
sudo apt install mosquitto-clients
```

##### mosquitto_sub
Pour s'abonner à une rubrique sur un _broker_ donné. La syntaxe de base est la suivante:
```
mosquitto_sub -h BROKER -t TOPIC 
```
Par exemple, si l'adresse du _broker_ est `10.10.10.22` et le sujet est `meteo`, la commande est celle-ci:
```bash
mosquitto_sub -h 10.10.10.22 -t "meteo"
```

##### mosquitto_pub
Pour publier un message dans une rubrique donnée. La syntaxe de la commande est:
```
mosquitto_pub -h BROKER -t TOPIC -m MESSAGE
```
Par exemple:
```bash
mosquitto_pub -f 10.10.10.22 -t "meteo" -m "Il neige"
```

#### Authentification
MQTT supporte des fonctionnalités d'authentification. Lorsqu'on les active, tous les clients devront fournir un identifiant et un mot de passe à l'agent au moment de la connexion.

Dans les cas où le _broker_ demande une authentification, les paramètres suivants doivent être passés aux programmes *mosquitto_pub* et *mosquitto_sub*:
- `-u` : Nom de l'utilisateur
- `-P` : Mot de passe

Par exemple:
```bash
mosquitto_pub -h 192.168.0.10 -u "sophie" -P "abc-123" -t "sujet" -m "Bonjour" 
```

{{% notice primary "Attention" %}}
MQTT ne définit pas de méthodes de chiffrement. Ainsi les mots de passe et identifiants sont transmis en clair lors de la connexion. Si on souhaite chiffrer les communications, il faut utiliser des outils tiers comme par exemple le protocole TLS.
{{% /notice %}}





