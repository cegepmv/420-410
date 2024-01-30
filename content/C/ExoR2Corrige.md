+++
title = 'ExoR2Corrige'
date = 2024-01-29T21:33:41-05:00
draft = true
+++
## 1
Créez un fichier nommé cours2.c qui permet d'afficher la phrase classic : Hello world.



```c
    #include <stdio.h>
    int main() {
        printf("hello world\n");
    }
```

## 2
Modifier le Makefile pour pouvoir utiliser make cours2 pour rapidement compiler ce fichier vers *cours2* et exécuter cet éxécutable.

```Make
all:

cours2: cours2.c
	gcc cours2.c -o cours2
	./cours2
```

## 3
Faites un code pour prendre comme entrée le nom d'un utilisateur et vérifier si c'est Jean

```c
#include <stdio.h>
#include <string.h>

int main() {
    char nom[30];
    fgets(nom, sizeof(nom), stdin);
    printf("%s", nom);

    if (strcmp(nom, "Jean") == 0) {
        printf(" Identique\n");
    } else {
        printf(" Différent\n");
    }

    return 0;
}
```

Ce code indique qu'il sont différent, c'est parce qu'il faut considérer le "\n". Donc il faudrait écrire le code ainsi.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char nom[30];
    fgets(nom, sizeof(nom), stdin);
    printf("%s", nom);

    if (strcmp(nom, "Jean\n") == 0) {
        printf(" Identique\n");
    } else {
        printf(" Différent\n");
    }

    return 0;
}
```
## 4
Sinon nous pouvons retirer le \n du string avant la comparaison:

```c
#include <stdio.h>
#include <string.h>

int main() {
    char nom[30];
    fgets(nom, sizeof(nom), stdin);
    printf("%s", nom);

    nom[strcspn(nom, "\n")] = 0; // Remove newline character
    if (strcmp(nom, "Jean") == 0) {
        printf(" Identique\n");
    } else {
        printf(" Différent\n");
    }

    return 0;
}
```

## 5 
Voyons maintenant comment utiliser les PINs de notre raspberryPi

```c
#include <stdio.h>
#include <unistd.h>
#include <pigpio.h>

#define PIN_NUMBER 18

int main() {
    // Initialize the pigpio library
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Failed to initialize pigpio\n");
        return 1;
    }

    // Set the specified GPIO pin to output mode
    gpioSetMode(PIN_NUMBER, PI_OUTPUT);

    while (1) {
        // Turn on the GPIO pin
        gpioWrite(PIN_NUMBER, 1);
        usleep(1000000);  // 1 second delay (in microseconds)

        // Turn off the GPIO pin
        gpioWrite(PIN_NUMBER, 0);
        usleep(1000000);  // 1 second delay (in microseconds)
    }

    // Terminate the pigpio library
    gpioTerminate();

    return 0;
}

```
## 6
Si vous avez des problèmes avec l'initialisation de pigpio, essayer:

```bash
    sudo killall pigpiod
```

Car il se peut qu'il y est des processus qui trainent.
## 7

Faites un programme qui lorsque vous appuyez sur le touch module, allume la LED

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pigpio.h>
//sudo killall pigpiod

int main() {
    if (gpioInitialise() < 0) {
        fprintf(stderr, "pigpio initialization failed.\n");
        return 1;
    }

    while (1) {
        usleep(500000); // Sleep for 0.5 seconds

        int value = gpioRead(17);

        if (value == 1) {
            gpioWrite(18, 1);
        } else {
            gpioWrite(18, 0);
        }
    }

    gpioTerminate();

    return 0;
}
```
## 8

Maintenant, faites un programme qui rendra interactif ce menu:


```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pigpio.h>

int main() {
    if (gpioInitialise() < 0) {
        fprintf(stderr, "GPIO initialization failed\n");
        return 1;
    }

    int pin = 18;
    gpioSetMode(pin, PI_OUTPUT);

    while (1) {
        
        printf("#########################################\n");
        printf("Choisissez l'une des options suivantes:\n");
        printf("#########################################\n");
        printf("   allume   -   Pour allumer la LED\n");
        printf("   ferme    -   Pour fermer la LED\n");
        printf("   quitter  -   Pour quitter\n");

        char cmd[20];
        fgets(cmd, sizeof(cmd), stdin);
        cmd[strcspn(cmd, "\n")] = 0; // Remove newline character

        if (strcmp(cmd, "allume") == 0) {
            gpioWrite(pin, 1);
        } else if (strcmp(cmd, "ferme") == 0) {
            gpioWrite(pin, 0);
        } else if (strcmp(cmd, "quitter") == 0) {
            gpioWrite(pin, 0);
            gpioTerminate();
            exit(0);
        } else {
            const char *data = "Commande introuvable";
            printf("%s\n", data);
        }
    }

    return 0;
}

```