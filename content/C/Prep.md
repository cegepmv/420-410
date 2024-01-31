+++
title = 'Préparation du Pi'
weight = 20
date = 2024-01-31T08:30:37-05:00
draft = true
+++

Pour programmer en langage C sur un _RaspberryPi_, il est essentiel d'installer quelques logiciels.

## Outils de développement
Les programmes requis sont les suivants:
+ Le compilateur C _gcc_;
+ Les librairies C standard;
+ L'utilitaire _make_

Tous sont inclus dans le paquet _build-essential_ qu'on installe avec la commande suivante:
```c
sudo apt update
sudo apt install build-essential
```

## Librairies GPIO
Le _RaspberryPi_ est équipé de 40 broches GPIO utilisées pour communiquer avec différents périphériques: senseurs, actuateurs, moteurs, LED, etc.

Pour utiliser cette interface à partir d'un programme en C, deux librairies sont disponibles: **WiringPi** et **pigpio**. 

#### WiringPi
Cette librairie semble très populaire et est utilisée dans de nombreux projets; cependant elle n'est plus activement maintenue par son développeur d'origine (https://projects.drogon.net/raspberry-pi/wiringpi/). Il n'est donc pas recommandé de l'utiliser.

Pour installer _WiringPi_, téléchargez le paquet **.deb** puis installez-le à l'aide des commandes suivantes:
```bash
wget https://project-downloads.drogon.net/wiringpi-latest.deb
sudo dpkg -i wiringpi-latest.deb
```

Le programme suivant envoit un courant de 3.3V durant 1 seconde sur la broche 11 du Pi (qui correspond à l'identifiant GPIO `0` dans _WiringPi_):

```c
// testWiring.c
#include <wiringPi.h>
int main (void) 
{
    wiringPiSetup();
    pinMode(0,OUTPUT);
    
    digitalWrite(0,HIGH);
    delay(1000);
    digitalWrite(0,LOW);
    
    return 0 ;
}
```

Au moment de la compilation il faut lier l'exécutable à la librairie _wiringpi_. Pour ce faire on utilise  l'option `-l`. La commande pour compiler le programme `testWiring.c` et générer l'exécutable `test` est donc la suivante:
```bash
gcc testWiring.c -o test -lwiringPi
```
La documentation de _WiringPi_ est ici: https://projects.drogon.net/raspberry-pi/wiringpi/functions/

#### pigpio
Cette librairie est réputée plus performante que _WiringPi_, et semble être mieux maintenue que cette dernière. Elle inclut également un "wrapper" qui permet de l'importer en python comme un module. Elle est accessible via un service linux nommé _pigpiod_.

On peut l'installer à partir des dépôts de _Raspberry Pi OS_ comme suit:
```bash
sudo apt update
sudo apt install pigpio
```

Pour exécuter le service et s'assurer qu'il le sera à chaque démarrage du Pi, vous devez faire les commandes suivantes:
```bash
sudo systemctl start pigpiod
sudo systemctl enable pigpiod
```

Le programme suivant est similaire à celui de l'exemple précédent: un courant de 3.3V  est envoyé durant 1 seconde sur la broche 11 du Pi (qui correspond à l'identifiant GPIO `17` dans _pigpio_):

```c
// testPig.c
#include <stdio.h>
#include <pigpio.h>

int main() {
    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(17, PI_OUTPUT);

    // Allumer 1 s
    gpioWrite(17, 1);
    time_sleep(1);  

    // Éteindre 1 s
    gpioWrite(17, 0);
    time_sleep(1);  
    
    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```

Au moment de la compilation il faut lier l'exécutable à la librairie _wiringpi_. Pour ce faire on utilise  l'option `-l`. La commande pour compiler le programme `testPig.c` et générer l'exécutable `test` est donc la suivante:
```bash
gcc testPig.c -o test -lpigpio
```
La documentation de l'API C de _pigpio_ est ici: http://abyz.me.uk/rpi/pigpio/cif.html



