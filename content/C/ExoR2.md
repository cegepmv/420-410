+++
title = 'Exo R2'
date = 2024-01-29T21:33:41-05:00
draft = true
+++
## 1
Créez un programme nommé `cours2.c` qui permet d'afficher la phrase classique _Hello world_.

<!--
```c
    #include <stdio.h>
    int main() {
        printf("hello world\n");
    }
```
-->
## 2
Modifier le _Makefile_ suivant afin que la commande `make` (sans argument) compile ce fichier vers *cours2* et exécute le programme.

```makefile
all: 

cours2: cours2.c
	# Commande de compilation
	./cours2
```
<!--
```Make
all: cours2

cours2: cours2.c
	gcc cours2.c -o cours2
	./cours2
```
-->
## 3
Faites un programme pour prendre comme entrée le nom d'un utilisateur et vérifier si le nom entré est "Jean". Affichez "Bienvenue" si c'est le cas, sinon affichez "Allez-vous en".

_La fonction **strcmp()** vous sera utile ici..._

<!--
```c
#include <stdio.h>
#include <string.h>

int main() {
    char nom[10];
    printf("Entrez votre nom: ");
    scanf("%s",nom);
    if (strcmp(nom,"Jean") == 0) {
        printf(" Bienvenue\n");
    } else {
        printf(" Allez-vous en\n");
    }
    return 0;
}
```
-->


## 4
 
Voyons maintenant comment utiliser les PINs du RaspberryPi

```c
#include <stdio.h>
#include <unistd.h>
#include <pigpio.h>

#define PIN_NUMBER 18

int main() {
    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Failed to initialize pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(PIN_NUMBER, PI_OUTPUT);

    while (1) {
        // Allumer 1 s
        gpioWrite(PIN_NUMBER, 1);
        usleep(1000000);  

        // Éteindre 1 s
        gpioWrite(PIN_NUMBER, 0);
        usleep(1000000);  
    }

    // Terminate the pigpio library
    gpioTerminate();

    return 0;
}

```
_S'il y a des problèmes avec l'initialisation de pigpio, essayer:_

```bash
    sudo killall pigpiod
```

Car il se peut qu'il y ait des processus qui trainent.

## 5

Faites un programme qui allume une LED lorsque vous appuyez sur un bouton (une LED et un bouton du kit KeyStudio).

## 6

Faites un programme qui allume ou éteint une LED selon ce qui est saisi à la ligne de commande:

```c
printf("#########################################\n");
printf("Choisissez l'une des options suivantes:\n");
printf("#########################################\n");
printf("   allume   -   Pour allumer la LED\n");
printf("   ferme    -   Pour fermer la LED\n");
printf("   quitter  -   Pour quitter\n");
```