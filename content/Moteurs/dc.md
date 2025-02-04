+++
title = 'Moteurs DC'
date = 2025-01-27T16:59:50-05:00
draft = false
weight = 31
+++

![dcmotor3v](/420-410/images/dcmotor3v.png?width=300px)

## Fonctionnement
Les moteurs DC ("courant direct") sont les plus simples des moteurs électriques. Ils ont deux pôles (positif et négatif), et le sens de leur rotation change lorsqu'on inverse la polarité du courant sur ces pôles. 


## Contrôleur 
Un moteur DC ne peut pas être connecté directement sur les broches GPIO d'un Pi: l'intensité du courant sur celles-ci peut atteindre un maximum de 50mA, mais un moteur de 3V-6V comme ceux fournis nécessite au moins 500mA. Il faut donc utiliser une alimentation électrique continue (comme les broches 3.3V ou 5V du Pi, ou encore une source de courant externe comme une pile), et contrôler ce courant électrique à l'aide du GPIO.

Plusieurs composantes électroniques peuvent être utilisées à cette fin, mais une des plus populaires est le contrôleur L293D. 

![l293d](/420-410/images/l293d.jpg)

Le courant électrique destiné au moteur traverse ce circuit intégré. Lorsqu'on y connecte aussi le GPIO du Pi, il est possible de le contrôler pour inverser la rotation, servir d'interrupteur, etc.

Utiliser un contrôleur a d'autres avantages:
- Le Pi fournit un courant continu de 3.3V ou 5V, mais certains moteurs ont besoin d'une tension plus élevée. Le contrôleur L293D supporte une tension pouvant aller jsuqu'à 36V.
- On peut connecter deux moteurs DC au contrôleur L293D.
- Il inclut des composantes qui diminuent le risque de dommages électriques.

### Broches
![l293dpins](/420-410/images/l293dpinout.png)



## Exemple
Comme on le voit dans le programme suivant, on contrôle le sens de la rotation à partir des deux broches `INPUT`: inverser les valeurs inversera la rotation. Si les deux valeurs sont identiques, le moteur ne tournera pas (on recommande quand même de mettre les deux valeurs à 0 pour arrêter le moteur).  

La broche `ENABLE` agit comme un interrupteur: elle bloque ou laisser passer le courant VSS (alimentation du moteur).

```python
import pigpio
import time

# Broches GPIO
EN1 = 16
IN1 = 20
IN2 = 21

# Initialisation
pi = pigpio.pi()
pi.set_mode(EN1,pigpio.OUTPUT)
pi.set_mode(IN1,pigpio.OUTPUT)
pi.set_mode(IN2,pigpio.OUTPUT)

try:
    # Sens de rotation
    pi.write(IN1, 0)
    pi.write(IN2, 1)

    # Envoyer le courant
    pi.write(EN1, 1)
    time.sleep(5)

finally:
    pi.write(IN1,0)
    pi.write(IN2,0)
    pi.write(EN1,0)
    pi.stop()
```

## Exercices
1. Faites un programme qui fait tourner le moteur 1 seonde dans un sens et 1 seconde dans l'autre sens, et qui se termine en arrêtant le moteur.

<!--
{{% expand "Solution" %}}
```python
import pigpio
import time

# Broches GPIO
EN1 = 16
IN1 = 20
IN2 = 21

# Initialisation
pi = pigpio.pi()
pi.set_mode(EN1,pigpio.OUTPUT)
pi.set_mode(IN1,pigpio.OUTPUT)
pi.set_mode(IN2,pigpio.OUTPUT)

try:
    # Sens de rotation
    pi.write(IN1, 0)
    pi.write(IN2, 1)

    # Envoyer le courant
    pi.write(EN1, 1)
    time.sleep(1)

    pi.write(IN1, 1)
    pi.write(IN2, 0)
    time.sleep(1)
    
finally:
    pi.write(IN1,0)
    pi.write(IN2,0)
    pi.write(EN1,0)
    pi.stop()
```
{{% /expand %}}
-->

2. En utilisant la fonction ***set_PWM_dutycycle()***, démarrez le moteur à sa vitesse maximale puis diminuez-la de 10% à chaque seconde jusqu'à l'arrêt. 
<!--
{{% expand "Solution" %}}
```python
import pigpio
import time

# Broches GPIO
EN1 = 16
IN1 = 20
IN2 = 21

# Initialisation
pi = pigpio.pi()
pi.set_mode(EN1,pigpio.OUTPUT)
pi.set_mode(IN1,pigpio.OUTPUT)
pi.set_mode(IN2,pigpio.OUTPUT)

try:
    dc = 255

    pi.write(IN1, 0)
    pi.write(IN2, 1)

    while dc > 0:
        pi.set_PWM_dutycycle(EN1,dc)
        dc -= 25
        time.sleep(1)

finally:
    pi.write(IN1,0)
    pi.write(IN2,0)
    pi.write(EN1,0)
    pi.stop()
```
{{% /expand %}}
-->
3. Changez le programme du numéro précédent pour qu'il augmente sa vitesse de rotation de 10% à toutes les secondes. Que se passe-t-il?
4. Faites un programme basé sur le [serveur UDP](/420-410/sockets/tcp-udp/#serveur) qui reçoit des messages UDP au port 2112 pour contrôler le moteur. Les messages doivent contenir 2 informations séparées par `:`: 
- `fwd` ou `rwd`: Tourner en sens horaire ou antihoraire.
- Un nombre qui représente le nombre de secondes avant que le moteur s'arrête.
- Par exemple, `rwd:4` fera tourner le moteur 4 secondes en sens antihoraire. 

<!--
{{% expand "Solution" %}}
import socket
import pigpio
import time

# Constantes
EN1 = 16
IN1 = 20
IN2 = 21
PORT = 2112
BUFFER_SIZE = 1024

# Fonctions
def off():
    pi.write(EN1,0)
    pi.write(IN1,0)
    pi.write(IN2,0)

def on():
    pi.write(EN1,1)
    pi.write(IN1,1)
    pi.write(IN2,0) 

def fwd():
    pi.write(IN1,1)
    pi.write(IN2,0)    

def rwd():
    pi.write(IN1,0)
    pi.write(IN2,1) 

# Init
pi = pigpio.pi()
pi.set_mode(EN1,pigpio.OUTPUT)
pi.set_mode(IN1,pigpio.OUTPUT)
pi.set_mode(IN2,pigpio.OUTPUT)
socket_local = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
adr_local = ('', PORT)
socket_local.bind(adr_local)

print(f"On attend les messages UDP entrants au port {PORT}...")

try:
    while True:
        buffer, adr_dist = socket_local.recvfrom(BUFFER_SIZE)
        message = buffer.decode()

        print(f"Message reçu: {message}", end='')

        sens, sec = message.split(":")
        
        # Traitement des données reçues
        on()
        if sens == "fwd": fwd()
        elif sens == "rwd": rwd()
        time.sleep(int(sec))
        off()

except ValueError:
    print("Erreur dans le message")
    off()

finally:
    off()
    socket_local.close()
{{% /expand %}}
-->

<!-- POUR PLUS TARD, 
Faire une version ou les message est en bits sur un octet
bit 1: on/off
bit 2: fwd/rwd
bits 3-8: nombre de secondes (0-64)

avec fonction asynchrone pour arrêter le moteur si le bit 1 est à 0
-->