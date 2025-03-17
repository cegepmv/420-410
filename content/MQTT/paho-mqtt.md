+++
title = 'Module paho-mqtt'
date = 2025-03-16T18:39:19-04:00
draft = false
weight = 52
+++

Le module _paho-mqtt_, développé par la fondation Eclipse, est la méthode la plus répandue pour utiliser le protocole MQTT en python. 

La documentation complète est ici: https://eclipse.dev/paho/files/paho.mqtt.python/html/client.html.

Le modue _paho_ utilise des fonctions de rappel ("callbacks") pour encoder les différentes tâches que le programme doit accomplir lors des différents évènements associés aux messages MQTT. Ceci permet d'éviter d'utiliser une boucle WHILE infinie pour contrôler le flot d'exécution du programme.

#### Publier un message
```python
import paho.mqtt.client as pmc

BROKER = "192.168.50.158"
PORT = 1883
TOPIC = "test"

def connexion(client, userdata, flags, code, properties):
    if code == 0:
        print("Connecté")
    else:
        print("Erreur code %d\n", code)

client = pmc.Client(pmc.CallbackAPIVersion.VERSION2)
client.on_connect = connexion

client.connect(BROKER,PORT)
client.publish(TOPIC,"allo")
client.disconnect()
```

Dans le programme précédent, la fonction `connexion` définit ce que le programme fait lors de la connexion au _broker_. Dans ce cas-ci, on affiche un message de succès / erreur. Notez que la fonction doit être associée à la propriété `on_connect` du client pour être appelée.

#### Recevoir des messages

```python
import paho.mqtt.client as pmc

BROKER = "192.168.50.158"
PORT = 1883
TOPIC = "test"

def connexion(client, userdata, flags, code, properties):
    if code == 0:
        print("Connecté")
    else:
        print("Erreur code %d\n", code)

def reception_msg(cl,userdata,msg):
    print("Reçu:",msg.payload.decode())

client = pmc.Client(pmc.CallbackAPIVersion.VERSION2)
client.on_connect = connexion
client.on_message = reception_msg

client.connect(BROKER,PORT)
client.subscribe(TOPIC)
client.loop_forever()
```
Dans cet exemple, on définit dans la fonction `reception_msg` ce qui doit être fait lorsqu'un message est reçu. Ici on ne fait qu'écrire le message à l'écran.

<!--
Dans un programme python qui utilise la librairie `paho-mqtt`, il faut appeler la méthode `Client.username_pw_set()` (avant la connexion) pour définir les identifiants à utiliser lors de la connexion. Voir https://eclipse.dev/paho/files/paho.mqtt.python/html/client.html pour plus de détails.
-->

## Exercices

1. Faire un programme qui envoie "10.10.10.X connecté" sur le canal `exercice1` (remplacez "10.10.10.X" par l'adresse IP de votre Pi), et qui s'abonne ensuite aux messages sur ce canal.
{{% expand "Solution" %}}
```python
import paho.mqtt.client as pmc

BROKER = "192.168.50.158"
PORT = 1883
TOPIC = "exercice1"

def connexion(client, userdata, flags, code, properties):
    if code == 0:
        print("Connecté")
        client.publish(TOPIC,"10.10.10.23 connecté")
    else:
        print("Erreur code %d\n", code)

def reception_msg(cl,userdata,msg):
    print("Reçu:",msg.payload.decode())

client = pmc.Client(pmc.CallbackAPIVersion.VERSION2)
client.on_connect = connexion
client.on_message = reception_msg

client.connect(BROKER,PORT)
client.subscribe(TOPIC)
client.loop_forever()
```
{{% /expand %}}

2. Faire un programme qui envoie le message "clic" au canal `exercice2` chaque fois que vous cliquez sur un bouton poussoir.
<!--
{{% expand "Solution" %}}
```python
```
{{% /expand %}}
-->
3. Faire un chat MQTT: au démarrage, le programme s'abonne au canal `exercice3` et affiche à l'écran tous les messages qui y passent. Le programme envoie aussi sur le même canal tous les messages écrits à la ligne de commande. 
<!--
{{% expand "Solution" %}}
```python
```
{{% /expand %}}
-->