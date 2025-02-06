+++
title = 'Exercices'
date = 2025-01-26T15:07:06-05:00
draft = false
weight = 21
+++


### UDP

Créez un **serveur** UDP sur votre PI qui s'attend à recevoir l'une des trois commandes suivante: `allume`, `ferme`, `exit`.

- `allume` : Allume une led
- `ferme` : Éteint la led
- `exit` : S'assure de fermer la led et de bien fermer le serveur.

Créez ensuite le **client** UDP sur votre ordinateur (pas le Pi). Le client doit pouvoir envoyer ces trois messages. 


### TCP

1. Faites deux programmes (`serveur1.py` et `client1.py`) ayant les mêmes fonctionnalités que celui de l'exercice précédent, mais en utilisant une connexion TCP. Au message _exit_, la connexion TCP est fermée des deux côtés.
{{% expand "Solution" %}}
client1.py
```python
import socket

PORT = 9090
DEST_IP = "10.10.20.245"

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
dest_addr = (DEST_IP, PORT)
sock.connect(dest_addr)

while True:
    message = input("> ")
    sock.send(message.encode() + b'\n')
    
    if message == "exit":
        break

sock.close()
```
serveur1.py
```python
import socket
import pigpio

LED_PIN = 17
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

socket_local.listen(3)
print(f"Écoute au port: {PORT}...")

socket_desc, client_address = socket_local.accept()
print(f"Socket distant: {client_address}")

while True:
    data = socket_desc.recv(BUFFER_SIZE)
    if data:
        message = data.decode().strip()
        print(f"< {message}")
        
        if message == "allume":
            pi.write(LED_PIN, 1)
        elif message == "ferme":
            pi.write(LED_PIN, 0)
        elif message == "exit":
            pi.write(LED_PIN, 0)
            pi.stop()
            break

socket_desc.close()
socket_local.close()
```
{{% /expand %}}
2. Ajoutez le fonctionnalité suivante: le serveur répond "OK" au client si la commande est _allume_, _ferme_ ou _exit_ ou "ERR" autrement (`serveur2.py` et `client2.py`).
{{% expand "Solution" %}}
client2.py
```python
import socket

PORT = 9090
DEST_IP = "192.168.50.190"
BUFFER_SIZE = 1024

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
dest_addr = (DEST_IP, PORT)
sock.connect(dest_addr)

while True:
    message = input("> ")
    sock.send(message.encode() + b'\n')
    
    # Réception de la réponse
    data = sock.recv(BUFFER_SIZE)
    if data:
        reponse = data.decode().strip()
        print("<",reponse)
        
    if message == "exit":
        break

sock.close()
```
serveur2.py
```python
import socket
import pigpio

LED_PIN = 17
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

socket_local.listen(3)
print(f"Écoute au port: {PORT}...")

socket_desc, client_address = socket_local.accept()
print(f"Socket distant: {client_address}")

while True:
    data = socket_desc.recv(BUFFER_SIZE)
    if data:
        message = data.decode().strip()
        print(f"< {message}")
        
        reponse = "OK"
        if message == "allume":
            pi.write(LED_PIN, 1)
        elif message == "ferme":
            pi.write(LED_PIN, 0)
        elif message == "exit":
            pi.write(LED_PIN, 0)
            pi.stop()
        else:
            reponse = "ERR"
        
        # Envoyer la réponse
        socket_desc.send(reponse.encode() + b'\n')

        if message == "exit":
            pi.stop()
            break
        
socket_desc.close()
socket_local.close()
```
{{% /expand %}}
3. Modifiez votre programme: lorsque le client envoit _exit_, la connexion TCP est terminée, le client se termine, mais le serveur continue à attendre d'autres connexions TCP (`serveur3.py` et `client3.py`).
{{% expand "Solution" %}}
client3.py
```python
import socket

PORT = 9090
DEST_IP = "192.168.50.190"
BUFFER_SIZE = 1024

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
dest_addr = (DEST_IP, PORT)
sock.connect(dest_addr)

while True:
    message = input("> ")
    sock.send(message.encode() + b'\n')
    
    # Réception de la réponse
    data = sock.recv(BUFFER_SIZE)
    if data:
        reponse = data.decode().strip()
        print("<",reponse)
        
    if message == "exit":
        break

sock.close()
```
serveur3.py
```python
import socket
import pigpio

LED_PIN = 17
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

# Ajout d'une boucle pour écouter les demandes de connexions entrantes
while True:
    socket_local.listen(3)
    print(f"Écoute au port: {PORT}...")

    socket_desc, client_address = socket_local.accept()
    print(f"Socket distant: {client_address}")

    while True:
        data = socket_desc.recv(BUFFER_SIZE)
        if data:
            message = data.decode().strip()
            print(f"< {message}")
            
            reponse = "OK"
            if message == "allume":
                pi.write(LED_PIN, 1)
            elif message == "ferme":
                pi.write(LED_PIN, 0)
            elif message == "exit":
                pi.write(LED_PIN, 0)
                # Sortir de le boucle WHILE interne 
                break
            else:
                reponse = "ERR"
            
            # Envoyer la réponse
            socket_desc.send(reponse.encode() + b'\n')

    # Fermer la connexion
    socket_desc.close()
```
{{% /expand %}}
4. Ajoutez une 2e LED sur votre Pi. Modifiez le programme serveur pour que les commandes envoyées permettent de spécifier laquelle des 2 LED allumer (les messages "led1" et "led2" doivent être envoyés au serveur pour qu'il sache quelle LED allumer ou éteindre). Le programme client n'a pas besoin d'être modifié. Les messages possibles du client sont donc: `led1, led2, allume, ferme, exit`. Les messages du serveur sont `OK, ERR, BYE`.
{{% expand "Solution" %}}
serveur4.py
```python
import socket
import pigpio

LED1 = 17
LED2 = 27
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

# Ajout d'une boucle pour écouter les demandes de connexions entrantes
while True:
    socket_local.listen(3)
    print(f"Écoute au port: {PORT}...")

    socket_desc, client_address = socket_local.accept()
    print(f"Socket distant: {client_address}")

    # Donner une valeur par défaut à la LED active pour éviter les erreurs
    led_active = LED1

    while True:
        data = socket_desc.recv(BUFFER_SIZE)
        if data:
            message = data.decode().strip()
            print(f"< {message}")
            
            reponse = "OK"
            if message == "led1":
                led_active = LED1
            elif message == "led2":
                led_active = LED2
            elif message == "allume":
                pi.write(led_active, 1)
            elif message == "ferme":
                pi.write(led_active, 0)
            elif message == "exit":
                # Tout éteindre et sortir de le boucle WHILE interne
                pi.write(LED1, 0)
                pi.write(LED2, 0) 
                reponse = "BYE"
                socket_desc.send(reponse.encode() + b'\n')
                break
            else:
                reponse = "ERR"
            
            # Envoyer la réponse
            socket_desc.send(reponse.encode() + b'\n')

    # Fermer la connexion
    socket_desc.close()
```
{{% /expand %}}

