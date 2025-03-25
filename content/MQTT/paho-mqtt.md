+++
title = 'Module paho-mqtt'
date = 2025-03-16T18:39:19-04:00
draft = false
weight = 52
+++

Le module _paho-mqtt_, développé par la fondation Eclipse, est la méthode la plus répandue pour utiliser le protocole MQTT en python. 

La documentation complète est ici: https://eclipse.dev/paho/files/paho.mqtt.python/html/client.html.

Le modue _paho_ utilise des fonctions de rappel ("callbacks") pour encoder les différentes tâches que le programme doit accomplir lors des différents évènements associés aux messages MQTT. Ceci permet d'éviter d'utiliser une boucle WHILE infinie pour contrôler le flot d'exécution du programme.

Pour installer _paho-mqtt_, lancez la commande suivante dans votre environnement virtuel:
```bash
(code) prof@fruit:~/code $ pip install paho-mqtt
```

#### Publier un message
```python
import paho.mqtt.client as pmc

BROKER = "mqttbroker.lan"
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

BROKER = "mqttbroker.lan"
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

#### Authentification
Lorsque le _broker_ demande aux clients de s'authentifier, il faut appeler la méthode `Client.username_pw_set()` (avant la connexion) pour définir les identifiants à utiliser lors de la connexion:
```python
...
client.username_pw_set("alice","abc-123")
client.connect(BROKER,PORT)
...
```

#### QoS
Pour activer les accusés de réception (QoS 1) et les confirmations d'envoi (QoS 2), il suffit de passer le paramètre `qos` à la méthode `Client.publish()` 
```python
client.publish(TOPIC,MESSAGE,qos=2)
```
Ces confirmations sont gérées par le module _paho.mqtt.client_: on ne peut pas y accéder à partir du programme. Donc, pour avoir l'assurance qu'un message a bien été reçu et/ou livré, il faut utiliser une fonction de rappel et la lier à l'évènement `on_publish`.

L'évènement `on_publish` est déclenché à 3 moments différentes selon le niveau de _QoS_:
- **QoS 0** : `on_publish` est déclenché lorsque le message est envoyé au _broker_
- **QoS 1** : `on_publish` est déclenché lorsque le message est reçu par le _broker_
- **QoS 2** : `on_publish` est déclenché lorsque le message est envoyé par le _broker_ aux autres clients.

Lorsqu'on utilise les niveaux de QoS 1 et 2, on doit s'attendre à recevoir un message chaque fois qu'on en envoit un: il faut donc structurer notre programme pour qu'il puisse recevoir ces messages. Le code suivant est problématique car il termine le programme avant que l'accusé de réception arrive:
```python
import paho.mqtt.client as pmc

BROKER = "mqttbroker.lan"
PORT = 1883
TOPIC = "test"

def publication(client, userdata, mid, code, properties):
    print("Envoi confirmé message #" + str(mid))

client = pmc.Client(pmc.CallbackAPIVersion.VERSION2)
client.on_publish = publication

client.connect(BROKER,PORT)
client.publish(TOPIC,"Voici un message")
```
Il faut donc ajouter deux choses au programme:
- `Client.loop_start()` : Démarre un _thread_ pour la réception des messages
- `MQTTMessageInfo.wait_for_publish()` : Attend la réception de l'accusé de réception pour poursuivre l'éxécution. Une instance de la classe _MQTTMessageInfo_ est retournée par `Client.publish()`.

Les modifications à apporter sont les suivantes:
```python
...
client.connect(BROKER,PORT)
client.loop_start()
msg_info = client.publish(TOPIC,"Voici un message",2)
msg_info.wait_for_publish(5)
...
```
Le paramètre `5` passé à *wait_for_publish()* est le nombre de secondes d'attente maximum. Si aucune valeur n'est passée, le programme attendra tant que la confirmation n'est pas reçue.

## Exercices

1. Faire un programme qui envoie "10.10.10.X connecté" sur le canal `exercice1` (remplacez "10.10.10.X" par l'adresse IP de votre Pi), et qui s'abonne ensuite aux messages sur ce canal.
{{% expand "Solution" %}}
```python
import paho.mqtt.client as pmc

BROKER = "mqttbroker.lan"
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
{{% expand "Solution" %}}
```python
import paho.mqtt.client as pmc
import pigpio

BROKER = "mqttbroker.lan"
PORT = 1883
TOPIC = "exercice2"
BTN = 26

def connexion(client, userdata, flags, code, properties):
    if code == 0:
        print("Connecté")
    else:
        print("Erreur code %d\n", code)

pi = pigpio.pi()
pi.set_mode(BTN,pigpio.INPUT)

client = pmc.Client(pmc.CallbackAPIVersion.VERSION2)
client.on_connect = connexion
client.connect(BROKER,PORT)

try:
    etat_bouton = 1
    while True:
        val_lue = pi.read(BTN)
        if val_lue != etat_bouton:
            etat_bouton = val_lue
            if etat_bouton == 0:
                client.publish(TOPIC,"clic")

except KeyboardInterrupt:
    pass
```
{{% /expand %}}
3. Faire un chat MQTT: au démarrage, le programme s'abonne au canal `exercice3` et affiche à l'écran tous les messages qui y passent. Le programme envoie aussi sur le même canal tous les messages écrits à la ligne de commande. 
<!--{{% expand "Solution" %}}
```python
import paho.mqtt.client as pmc
import threading

BROKER = "mqttbroker.lan"
PORT = 1883
TOPIC = "exercice3"

def connexion(client, userdata, flags, code, properties):
    if code == 0:
        print("Connecté")
        client.publish(TOPIC,"10.10.10.23 est arrivé")
    else:
        print("Erreur code %d\n", code)

def reception_msg(cl,userdata,msg):
    print("\t\t",msg.payload.decode())

def envoi_msg():
    while True:
        message = input()
        client.publish(TOPIC,message)

client = pmc.Client(pmc.CallbackAPIVersion.VERSION2)
client.on_connect = connexion
client.on_message = reception_msg
thread_entree_message = threading.Thread(target=envoi_msg, daemon=True)

try:
    client.connect(BROKER,PORT)
    client.subscribe(TOPIC)
    thread_entree_message.start()
    client.loop_forever()

except KeyboardInterrupt:
    client.publish(TOPIC,"10.10.10.23 a quitté")
    client.disconnect()
```
{{% /expand %}}
-->
4. Connectez un détecteur de luminosité sur votre Pi (n'oubliez pas d'utiliser le convertisseur ADS1115). Vous devez ensuite faire 2 programmes:
+ **ex4_pub**: (À faire en premier) Publie chaque 10 secondes dans la rubrique `/ex4/NOM_HOTE` la valeur de luminosité lue sur le senseur. Remplacez "NOM_HOTE" par la valeur de `hostname` de votre Pi;
+ **ex4_sub**: À partir des données lues sur tous les Pi, affiche le nom du Pi d'où est lue la valeur maximale, et la moyenne de toutes les dernières valeurs lues sur chaque Pi. Attention, vous devez prendre en compte que parfois un autre Pi peut ne pas envoyer d'informations dans sa période de 10 secondes: lorsque cela se produit, vous devez quand même calculer la moyenne et trouver le maximum parmi les informations que vous avez reçues... Pensez à l'algorithme de votre programme!
+ La valeur de luminosité envoyée par **ex4_pub** doit être un pourcentage (un nombre entier entre 0 et 100).
+ Les informations doivent s'afficher comme suit, à intervalles réguliers, sur la console lorsque vous exécutez le programme **ex4_sub**:
```
Max: denis (93)
Moy: 60.40
-------------
Max: WuJitsu (56)
Moy: 24.60
-------------
Max: Equipe 3 (72)
Moy: 34.20
-------------
```
<!--
{{% expand "Solution" %}}
```python
```
{{% /expand %}}
-->
5. Configurez un _broker_ pour qu'il contienne l'utilisateur `info` et le mot de passe `Password1234`. Ensuite faites un programme qui envoit un message au _topic_ "exercice5" lorsque l'utilisateur clique un bouton. Le message doit être envoyé avec QoS de niveau 2 et afficher "Message envoyé" à l'évènement `on_publish`.
<!--
{{% expand "Solution" %}}
```python
```
{{% /expand %}}
-->