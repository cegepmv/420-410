+++
title = 'Logiciels requis'
weight = 11
date = 2024-01-31T08:30:37-05:00
draft = false
+++

Pour programmer en Python sur un _RaspberryPi_, il est essentiel d'installer quelques logiciels et modules.

## Outils de développement
Les programmes et libraires requis sont les suivants:
+ *python3* ;
+ Les modules `venv` et `pigpio` de *pip*.

Normalement, *python3* devrait déjà être installé dans _RaspberryPi OS_. Toutefois, si ce n'est pas le cas, il est possible de le faire manuellement en installant le paquet *python3* avec la commande suivante:

```bash
sudo apt update
sudo apt install python3
```

### Environnement virtuel Python (*venv*)

Le module Python *venv* permet de créer des "environnements virtuels" légers, portables et isolés du système global. Chaque environnement virtuel a son propre binaire Python (qui correspond à la version du binaire qui a été utilisée pour créer cet environnement) et peut avoir sa propre liste de paquets Python installée. 

Les environnements virtuels permettent d'éviter d'avoir à installer les paquets Python sur tout le système. Il permettent aussi de créer un fichier de configuration contenant la liste complète des dépendances/modules de notre programme, ce qui facilite grandement son exportation sur une autre machine.

#### Création d'un environnement virtuel

La création d'un environnement virtuel se fait en exécutant la commande `venv`, suivie du chemin du répertoire qui contiendra la configuration de l'environnement :

```bash
python3 -m venv /chemin/vers/le/dossier/de/lenvironment/virtuel
```

Pour *rentrer* dans l'environnement virtuel, il suffit de lancer la commande suivante : 

```bash
source /chemin/vers/le/dossier/de/lenvironment/virtuel/bin/activate
```

{{% notice style="info" title="Conseils"%}}
Créez le répertoire de l'environnement virtuel à l'intérieur du répertoire de votre programme.
{{% /notice %}}

Pour vérifier que nous sommes bel et bien *à l'intérieur* de l'environnement virtuel, son nom devrait apparaître sur la ligne de commande (entre parenthèses) :

![venv01](/420-410/images/venv01.png)

Lorsque l'environnement virtuel est actif, `pip` installe les paquets **dans** l'environnement, ce qui n'affecte en rien l'installation de base de Python (celle du système global).

![venv02](/420-410/images/venv02.png)


Pour "sauvegarder" la liste des modules installés, il faut utiliser la commande `pip freeze` : 

```bash
pip freeze > requirements.txt
```

Cette commande redirige la liste des modules et les dépendances installées dans un fichier `requirements.txt` :

![venv04](/420-410/images/venv04.png)

{{% notice style="warning" title="Attention"%}}
À chaque fois que vous installez un nouveau module dans votre environnement virtuel, n'oubliez pas de relancer la commande `pip freeze > requirements.txt` pour mettre à jour la liste !
{{% /notice %}}

Pour sortir de l'environnement, il suffit de lancer la commande `deactivate` (vous remarquerez que le nom de l'environnement virtuel disparait de la ligne de commande) : 

![venv03](/420-410/images/venv03.png)


Si vous désirez exporter votre code sur une autre machine (par exemple, sur un autre *Raspberry PI*), il vous suffira de copier votre code (et le fichier `requirements.txt`), créer un nouvel environnement virtuel, entrer dans l'environnement, puis lancer la commande suivante : 

```bash
pip install -r requirements.txt
```

Cette commande installera la liste des paquets et et les dépendances du fichier `requirements.txt`. 

## Module GPIO
Le _RaspberryPi_ est équipé de 40 broches GPIO utilisées pour communiquer avec différents périphériques: senseurs, actuateurs, moteurs, LED, etc.

![pinoutpi](/420-410/images/pinoutpi.png?height=300px)
(source: https://pinout.xyz/)

Pour communiquer avec cette interface à partir d'un programme en Python on utilise le module **pigpio**. 

> La documentation de _pigpio_ est ici: https://abyz.me.uk/rpi/pigpio/python.html 

Normalement il est déjà disponible sur _Raspberry Pi OS_, mais au besoin on peut aussi l'installer à partir des dépôts de comme suit:

```bash
sudo apt update
sudo apt install pigpio
```

Pour exécuter le service et s'assurer qu'il le sera à chaque démarrage du Pi, vous devez entrer les commandes suivantes:
```bash
sudo systemctl start pigpiod
sudo systemctl enable pigpiod
```

Dans le programme suivant un courant de 3.3V  est envoyé durant 1 seconde sur la broche 11 du Pi (qui correspond à l'identifiant GPIO `17` dans _pigpio_):

```python
import pigpio
import time

# Initialisation
pi = pigpio.pi()

# Définir le mode du GPIO 17
LED = 17
pi.set_mode(LED, pigpio.OUTPUT)

try:
    # Allumer 1 seconde
    pi.write(LED, 1)
    time.sleep(1)
    
    # Éteindre
    pi.write(LED, 0)

finally:
    pi.stop()
```





