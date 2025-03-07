+++
title = 'Exercice synthèse'
date = 2025-03-03T00:56:26-05:00
draft = false
weight = 42
+++

Vous devez faire un programme qui reçoit par UDP des messages pour contrôler une LED et un moteur _stepper_. 

Les messages se composent de 3 parties séparées par `:`
- Première partie: `LED` ou `STEP`. Désigne ce que le programme doit activer.
- Deuxième partie: 
  - Si `LED`: Nombre entier. Correspond au nombre de fois que la LED doit clignoter.
  - Si `STEP`: `H` ou `A`. Sens de rotation Horaire ou Antihoraire.
- Troisième partie: 
  - SI `LED`: Nombre entier. La vitesse de clignotement en nombre de fois par seconde.
  - Si `STEP`: Nombre entier [0-360]. L'angle de rotation.
  
#### Exemples de messages
- `LED:10:4`  -   La LED clignote 10 fois à vitesse de 4 fois par seconde
- `STEP:H:90` -   Le moteur fait une rotation de 90 degrés dans le sens horaire

Le programme doit pouvoir faire fonctionner le moteur et la LED en même temps, comme par exemple si le serveur reçoit `LED:60:1` et `STEP:A:720` immédiatement après. Vous devrez donc utiliser les _Threads_.
{{% expand "Solution" %}}
```python
import pigpio
import time
import socket
import threading

M1,M2,M3,M4 = 26,13,19,6
LED = 4
PORT = 8888
BUFFER_SIZE = 1024
step_pause = 0.002
steps_par_tour = 2048
steps_par_degre = 5.68

# Full-step
seq_full = [
    [1,0,0,1],
    [1,1,0,0],
    [0,1,1,0],
    [0,0,1,1]
]

def stop_moteur():
    pi.write(M1, 0)
    pi.write(M2, 0)
    pi.write(M3, 0)
    pi.write(M4, 0)

def clignoter(nombre,fois_par_seconde):
    n = 0
    while n < nombre:
        pi.write(LED,1)
        time.sleep(0.5 / fois_par_seconde)
        pi.write(LED,0)
        time.sleep(0.5 / fois_par_seconde)
        n+=1

def tourner(sens_horaire,degres):
    sequence = seq_full if sens_horaire else list(reversed(seq_full))
    n = 0 
    while n < degres * steps_par_degre:
        for step in sequence:
            # Activer chacune des 4 bobines
            pi.write(M1, step[0])
            pi.write(M2, step[1])
            pi.write(M3, step[2])
            pi.write(M4, step[3])

            n += 1
            time.sleep(step_pause)

# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
adr_local = ('', PORT)

# Lier le socket à l'adresse
socket_local.bind(adr_local)

# Initialiser pi 
pi = pigpio.pi()
pi.set_mode(LED,pigpio.OUTPUT)
pi.set_mode(M1,pigpio.OUTPUT)
pi.set_mode(M2,pigpio.OUTPUT)
pi.set_mode(M3,pigpio.OUTPUT)
pi.set_mode(M4,pigpio.OUTPUT)

try:
    while True:
        # Réception du message UDP
        buffer, adr_dist = socket_local.recvfrom(BUFFER_SIZE)
        message = buffer.decode().strip()
        donnees = message.split(":")

        # Lancer le thread de la fonction "clignoter"
        if donnees[0] == "LED":
            th_led = threading.Thread(target=clignoter, daemon=True, args=(int(donnees[1]),int(donnees[2])))
            th_led.start()

        # Lancer le thread de la fonction "tourner"
        elif donnees[0] == "STEP":
            if donnees[1] == "HOR":
                horaire = True 
            elif donnees[1] == "AHR":
                horaire = False
            else: print("ERREUR")
            th_step = threading.Thread(target=tourner, daemon=True, args=(horaire,int(donnees[2])))
            th_step.start()

        else: 
            print("ERREUR")
            continue
        
        # On n'appelle pas join() car on ne veut pas attendre la fin 
        # de l'éxécution des fonctions pour continuer la boucle

except KeyboardInterrupt:
    stop_moteur()
    socket_local.close()
```
{{% /expand %}}
