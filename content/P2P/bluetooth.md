+++
title = 'Bluetooth'
date = 2025-04-21T20:26:55-04:00
draft = false
weight = 62
+++

_Bluetooth_ a été inventé pour remplacer les câbles qu'on utilisait pour connecter les appareils et leurs périphériques, comme par exemple les écouteurs, les souris ou les manettes de jeu. 

Il est basé sur une architecture client-serveur, mais permet seulement une communication de 1 à 1 entre deux composantes: les multiples clients d'un même serveur ne peuvent pas utiliser bluetooth pour communiquer entre eux.

En plus de la version habituelle du protocole, aussi nommée _classique_, il existe une version allégée nommée _Bluetooth LE_ (aussi _BLE_, pour "Bluetooth low energy"). Celle-ci a été conçue expressément pour les objets connectés et se caractérise par les aspects suivants:
+ Faible consommation d'énergie
+ Taux de transferts plus bas
+ Mieux adaptée aux transferts de petites quantités de données

Même si de nombreuses composantes supportent les deux versions de Bluetooth, les deux sont assez différents pour être considérés comme deux protocoles distincts. Il est donc possible que des composantes supportent l'un mais pas l'autre.

Dans cette section nous allons tester la communication _blutooth classique_ entre un RaspberryPi et d'autres composantes.

En raison de son universalité, le protocole bluetooth doit satisfaire de nombreux cas d'usage et plusieurs types de périphériques différents. Pour cette raison il est assez complexe à utiliser. 

Dans un premier temps nous allons voir comment utiliser le programme *bluetoothctl* pour gérer les connexions sur le Raspberry Pi.

## bluetoothctl
Ce programme donne accès à un *shell*, c'est-à-dire un interpréteur dans lequel on peut lancer des commandes spécifiques à la configuration bluetooth:
```
prof@pi:~ $ sudo bluetoothctl
Agent registered
[bluetooth]# 
```
Quelques commandes utiles:
- `help`: affiche la liste des commandes du *shell*. Les éléments en bleu donnent accès à des sous-menus.
- `show`: affiche les informations du contrôleur (le programme qui contrôle bluetooth sur le Pi)
- `devices`: affiche la liste des périphériques appariés
- `scan` : suivi de `on` ou `off`, active ou désactive la recherche de périphériques
- `info` : suivi de l'adresse MAC d'un périphérique, affiche ses informations
- `trust` : définit un périphérique comme étant "de confiance" ce qui évite de devoir donner une confirmation chaque fois qu'il se connecte
- `pair` : suivi de l'adresse MAC d'un périphérique, ajoute celui-ci à la liste des périphériques connus
- `remove` : supprime un item de la liste des périphériques connus
- `connect` : connecte un périphérique
- `disconnect` : déconnecte un périphérique

#### Phases du protocole
Pour établir une connexion bluetooth il faut réaliser plusieurs phases. 

Un péripérique bluetooth peut se connecter à d'autres sans permettre aux autres de se connecter à lui. Il a donc la possibilité d'être **découvrable** ou non.

Avant d'établir une connexion par bluetooth, les deux périphériques doivent se reconnaître. On dit alors qu'ils sont **appariés**. C'est dans cette phase qu'il est possible de mettre un mécanisme d'authentification: ceci permet ainsi aux périphériques de s'authentifier une seule fois pour plusieurs connexions à venir. 

Un périphérique peut être de **confiance**, ce qui permet d'éviter de confirmer chaque fois qu'il se connecte.

#### Pour se connecter
Dans le shell bluetoothctl, faites les commandes suivantes:
```
power on 
discoverable on
pairable on
agent on
default-agent
```
Ces commandes on les effets suivants:
###### power on
Active le contrôleur bluetooth.
###### discoverable on
Met le contrôleur en mode découvrable, ce qui le rend visible aux autre périphériques bluetooth. Attention, au bout de 5 minutes le contrôleur désactive automatiquement ce mode.
###### pairable on
Accepte les requêtes d'appariement venant des autres périphériques. 
###### agent on
Active l'agent bluetooth, qui est le programme chargé de l'authentification / autorisation durant le processus d'appariement. Les options de l'agent permettent de se connecter sans NIP, avec visualisation seulement d'un NIP, avec une entrée manuelle d'un NIP, etc.
###### default-agent
Il existe plusieurs types d'agents qui dépendent de l'implémentation de bluetooth utilisée par le système. Chaque implémentation a cependant toujours un agent par défaut: cette commande permet de le sélectionner. 

Ensuite vous devriez voir votre Pi parmi les périphériques accessibles sur votre téléphone. Au moment de la connexion, vous aurez à confirmer un NIP sur les deux périphériques:
```
[NEW] Device 74:74:46:CB:BF:37 Pixel 5
Request confirmation
[agent] Confirm passkey 375299 (yes/no): yes
```
<!--
Faire les commandes:

devices : voir les paired
info : voir les infos du device
disconnect
devices : voir les paired
connect MAC: voir qu'il faut confirmer
disconnect
trust MAC
connect : voir qu'on n'a plus besoin de confirmer
-->

## BlueDot
BlueDot est une application Android assortie d'une librairie qui permet de facilement contrôler des programmes python sur des périphériques bluetooth. L'application existe aussi au format graphique sur RaspberryPi.

La référence: https://bluedot.readthedocs.io/en/latest/index.html

Pour installer la partie client sur un téléphone Android, allez sur le Google Store.

#### BueDot serveur
Pour installer la partie serveur sur votre Pi, la procédure est la suivante:
```
sudo apt-get install libdbus-glib-1-dev libdbus-1-dev
pip3 install dbus-python
pip3 install bluedot
```

Le programme suivant affiche "bonjour" sur la console du Pi chaque fois que le bouton est appuyé dans l'application Android:
```python
from bluedot import BlueDot

def bonjour():
	print("bonjour")

bd = BlueDot()
bd.when_pressed = bonjour

while True:
	pass
```

#### BlueDot client
Pour installer la partie client sur votre Pi, vous devez installer la librairie *PyGame*. La commande est celle-ci:
```
pip3 install pygame
```

> ATTENTION: La partie _serveur_ du programme doit s'exécuter avant que la partie client essaie de se connecter.

## Exercices
1. Utilisez l'évènement `when_pressed` pour allumer une LED lorsqu'on appuie sur le bouton.
<!--
{{% expand "Solution" %}}
```python
from bluedot import BlueDot
import pigpio

LED = 26

pi = pigpio.pi()
pi.set_mode(LED,pigpio.OUTPUT)

def allume():
	pi.write(LED, 1)

bd = BlueDot()
bd.when_pressed = allume

while True:
	pass
```
{{% /expand %}}
-->
2. Dans l'exercice précédent, la LED ne s'éteint pas lorsqu'on relâche le bouton. Modifiez votre programme pour que ce soit la cas. Consultez la documentation pour savoir quel évènement utiliser.
<!--
{{% expand "Solution" %}}
```python
from bluedot import BlueDot
import pigpio

LED = 26

pi = pigpio.pi()
pi.set_mode(LED,pigpio.OUTPUT)

def allume():
	pi.write(LED, 1)

def eteint():
    pi.write(LED, 0)

bd = BlueDot()
bd.when_pressed = allume
bd.when_released = eteint

while True:
	pass
```
{{% /expand %}}
-->
3. Faites un programme qui affiche 3 boutons sur l'appli BlueDot (rouge, vert et bleu) et allume les couleurs correspondantes sur une LED RGB du Pi d'une autre équipe.
<!--
{{% expand "Solution" %}}
```python
from bluedot import BlueDot
import pigpio

R,G,B = 26,19,13

def rouge():
    pi.write(R,0)
    pi.write(G,1)
    pi.write(B,1)

def vert():
    pi.write(R,1)
    pi.write(G,0)
    pi.write(B,1)

def bleu():
    pi.write(R,1)
    pi.write(G,1)
    pi.write(B,0)       

def eteindre():
    pi.write(R,1)
    pi.write(G,1)
    pi.write(B,1)     

bd = BlueDot(rows=3)
bd[0,0].color = "red"
bd[0,1].color = "green"
bd[0,2].color = "blue"
bd[0,0].when_pressed = rouge
bd[0,1].when_pressed = vert
bd[0,2].when_pressed = bleu

pi = pigpio.pi()
pi.set_mode(R,pigpio.OUTPUT)
pi.set_mode(G,pigpio.OUTPUT)
pi.set_mode(B,pigpio.OUTPUT)

while True:
	try:
        pass
    except KeyboardInterrupt:
        eteindre()
```
{{% /expand %}}
-->
