+++
title = 'TCP et UDP'
date = 2024-02-06T12:05:07-05:00
weight = 20
draft = false
+++

Pour établir une communication entre un Raspberry Pi et un autre hôte sur le réseau, on peut adopter l'approche client-serveur: 
+ Le _serveur_ est celui qui reçoit les messages
+ Le _client_ est celui qui envoit les messages

Le Raspberry Pi peut être n'importe lequel des deux. 

Deux protocoles réseau peuvent être utilisés pour envoyer des messages entre un client et un serveur, soit _TCP_ et _UDP_.
+ TCP: une connexion est établie entre client et serveur. Cette connexion est ensuite utilisée pour que les deux s'échangent des messages. Lorsqu'un des deux termine la connexion, celle-ci est fermée.
+ UDP: le client envoit des messages au serveur sans établir de connexion préalable. Il n'est donc pas possible de savoir si le serveur est en ligne et prêt à recevoir les messages.

Pour communiquer en utilisant TCP ou UDP, il faut utiliser les _sockets_.

Un socket représente le point terminal d'une communication entre deux hôtes sur un réseau. Il peut être la source ou la destination d'un message. Les sockets sont définis par une adresse IP et un port et sont utilisés par les programmes pour envoyer et recevoir des messages.

## Client
#### UDP
Le programme suivant envoit un message par UDP sur un réseau. L'adresse de destination est `127.0.0.1` et le port est `8888`:
```python
import socket

PORT = 8888
DEST_IP = "127.0.0.1"
MESSAGE = "abcdefg!\n"

# Créer le socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Créer un objet pour l'adresse
dest_addr = (DEST_IP, PORT)

# Envoyer le message
sock.sendto(MESSAGE.encode(), dest_addr)

# Fermer le socket
sock.close()
```

Vous pouvez tester ce programme comme suit: à partir d'un hôte sur le même réseau que le Pi, ouvrez un port UDP en mode "listen" avec la commande `nc`. Cet hôte joue donc le rôle de serveur. Par exemple, pour ouvrir le port UDP 8888 la commande est la suivante :
```bash
nc -ulp 8888
``` 
> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`.

#### TCP
Le programme suivant envoit un message par TCP sur un réseau. L'adresse de destination est `127.0.0.1` et le port est `8888`:
```python
import socket

PORT = 8888
DEST_IP = "127.0.0.1"
MESSAGE = "abcdefg!\n"

# Create the socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Initialize the destination address
dest_addr = (DEST_IP, PORT)

# Create the connection
sock.connect(dest_addr)

# Send the message and close the connection
sock.send(MESSAGE.encode())
sock.close()
```
Vous pouvez tester ce programme avec la commande `nc`. À partir d'un hôte sur le même réseau que le Pi (qui aura le rôle de serveur), ouvrez un port TCP en mode "listen" au port TCP 8888:
```bash
nc -lp 8888
``` 
> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`; ensuite exécutez-le.


## Serveur
#### UDP
Le programme suivant ouvre un socket au port UDP 8888 et affiche les messages entrants:
```python
import socket

PORT = 8888
BUFFER_SIZE = 1024

# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
adr_local = ('', PORT)

# Lier le socket à l'adresse
socket_local.bind(adr_local)

print(f"On attend les messages UDP entrants au port {PORT}...")

# Réception des messages
while True:
    # Réception des données
    buffer, adr_dist = socket_local.recvfrom(BUFFER_SIZE)
    
    # Décodage binaire -> texte
    message = buffer.decode()

    print(f"Message reçu: {message}", end='')
```
Pour tester ce programme à partir d'un autre hôte (qui agit comme client), lancez la commande `nc` (remplacez 1.2.3.4 par l'adresse IP du serveur) et tapez le message que vous voulez envoyer:
```bash
nc -u 1.2.3.4 8888
abcdefg!
```

#### TCP
Le programme suivant ouvre un socket au port TCP 8888 et attend les connexions entrantes. Lorsque la connexion est établie il affiche les messages et termine la connexion lorsque le message reçu est vide (taille 0):

```python
import socket

PORT = 8888
BUFFER_SIZE = 1024

# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  

# Lier le socket à l'adresse
socket_local.bind(address)

# Attendre une connexion
socket_local.listen(3)
print(f"Serveur TCP en écoute au port {PORT}...")

socket_dist, client_address = socket_local.accept()
print(f"Socket distant: {client_address}")

# Réception des messages
while True:
    # Données
    data = socket_dist.recv(BUFFER_SIZE)
    if not data:
        print("Déconnecté")
        break
    else:
        # Décoder + afficher les message
        message = data.decode()
        print(f"Reçu: {message}", end='')

socket_dist.close()
socket_local.close()
```
Pour tester ce programme à partir d'un autre hôte (qui agit comme client), lancez la commande `nc` (remplacez 1.2.3.4 par l'adresse IP du serveur) et tapez le message que vous voulez envoyer:
```bash
nc 1.2.3.4 8888
abcdefg!
```
