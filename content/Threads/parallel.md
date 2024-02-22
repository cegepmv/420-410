+++
title = 'Processus en parallèle'
date = 2024-02-11T19:56:53-05:00
draft = false
+++

Dans les applications connectées, il est fréquent qu'un même programme doive gérer plusieurs processus simultanément. Cela peut poser des problèmes lorsqu'une tâche monopolise le processeur et que pendant ce temps, d'autres processus attendent leur tour. 

Dans un programme ordinaire, les instructions sont exécutées en séquence:
```c
long double pi;

pi = calculerPiALaMillioniemeDecimale()
printf("SVP Patientez");
```

Dans cet exemple, le message "SVP Patientez" est assez inutile car l'instruction `printf()` ne s'exécutera pas tant que la fonction de la ligne précédente ne sera pas terminée. 

Prenons un programme simple qui fait clignoter indéfiniment un module LED toutes les 250ms:
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>

#define LED 17

int main() {
    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED, PI_OUTPUT);

    while (1) {
        // Allumer 1 s
        gpioWrite(LED, 1);
        usleep(250000);  

        // Éteindre 1 s
        gpioWrite(LED, 0);
        usleep(250000);  
    }
    
    // Libérer les ressources
    gpioTerminate();
    return 0;
}
```

Comment peut-on modifier ce programme pour faire clignoter 2 modules LED, 1 chaque 250ms et l'autre chaque 500ms?
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>

int main() {
    // Faire clignoter LED 1 chaque 250ms et LED 2 chaque 500ms
    int LED1 = 17;
    int LED2 = 24;

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED1, PI_OUTPUT);
    gpioSetMode(LED2, PI_OUTPUT);

    while (1) {
        gpioWrite(LED1, 1);
        gpioWrite(LED2, 1);

        usleep(250000);
        gpioWrite(LED1, 0);

        usleep(250000);
        gpioWrite(LED1, 1);
        gpioWrite(LED2, 0);

        usleep(250000);
        gpioWrite(LED1, 0);

        usleep(250000);
    }

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```

On voit immédiatement que cette solution n'est pas idéale car la structure de notre programme dépend des données. Si en effet on veut changer les durées, par exemple 1 LED qui clignote chaque 300ms et l'autre chaque 700ms, il faudra revoir tout ce que contient la boucle `while`.

Intuitivement on voit qu'il serait préférable d'avoir deux boucles et que chacune définit sa propre durée. Le programme suivant est mieux construit:

```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>
#define LED1 17
#define LED2 24

void clignoter1() {
    for (;;) {
        gpioWrite(LED1, 1);
        usleep(250000);
        gpioWrite(LED1, 0);
        usleep(250000);
    }
}

void clignoter2() {
    for (;;) {
        gpioWrite(LED2, 1);
        usleep(500000);
        gpioWrite(LED2, 0);
        usleep(500000);
    }
}

int main() {
    // Faire clignoter LED 1 chaque 250ms et LED 2 chaque 500ms

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED1, PI_OUTPUT);
    gpioSetMode(LED2, PI_OUTPUT);

    clignoter1();
    clignoter2();

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```
Chacune des deux boucles est définie dans sa propre fonction, et même si il y a plusieurs répétitions, le programme est mieux structuré. Par contre il ne donne pas le résultat escompté car les deux fonctions vont s'éxécuter une après l'autre.

## _Threads_
En programmation, il est possible de créer des nouveaux processus plus ou moins indépendants du flot normal d'exécution du programme principal. On nomme ces processus "threads". 

Lorsqu'on crée un thread, on lui associe une fonction. Cette fonction sera exécutée dans son propre processus et ainsi ne bloquera pas le processus principal. 

Dans l'exemple suivant, on utilise la fonction `pthread_create()` pour lancer les fonctions "clignoter" dans leurs propres processus:
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>
#include <pthread.h>
#define LED1 17
#define LED2 24

void *clignoter1() {
    for (;;) {
        gpioWrite(LED1, 1);
        usleep(250000);
        gpioWrite(LED1, 0);
        usleep(250000);
    }
}

void *clignoter2() {
    for (;;) {
        gpioWrite(LED2, 1);
        usleep(500000);
        gpioWrite(LED2, 0);
        usleep(500000);
    }
}

int main() {
    // Faire clignoter LED 1 chaque 250ms et LED 2 chaque 500ms

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED1, PI_OUTPUT);
    gpioSetMode(LED2, PI_OUTPUT);

    // Créer les threads
    pthread_t t_led1,t_led2;

    if (pthread_create(&t_led1,NULL,clignoter1,NULL) != 0) {
        printf("Erreur à la création du thread pour LED 1.\n");
        return 1;
    }
    if (pthread_create(&t_led2,NULL,clignoter2,NULL) != 0) {
        printf("Erreur à la création du thread pour LED 2.\n");
        return 1;
    }

    // Attendre la fin de l'exécution de chaque thread
    pthread_join(t_led1, NULL);
    printf("Fin du thread 1.\n");
    pthread_join(t_led2, NULL);
    printf("Fin du thread 2.\n");

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```
Les modifications apportées au code:
+ Ajouté `#include <pthread.h>`
+ Les fonctions sont déclarées comme des pointeurs
+ Des variables de type `pthread_t` réfèrent aux threads créés
+ La fonction `pthread_create()` crée les threads
+ La fonction `pthread_join()` attend que les threads se terminent

> ATTENTION: Au moment de la compilation, il faut ajouter l'option `-lpthread` à la commande **gcc**.
> 
##### `pthread_create()`
Sert à créer un _thread_, le mettre dans une variable et l'associer à une fonction donnée. Dans le programme cette fonction a 4 paramètres:
+ `&t_led1` : La référence au thread créé
+ `NULL` : Les attributs qu'on veut lui donner (par exemple sa priorité, la taille de la mémoire allouée, etc.). Si NULL, le thread aura des valeurs par défaut. 
+ `clignoter` : Le nom de la fonction que le processus doit appeler. Attention, il n'y a pas les parenthèses: on donne le nom seulement.
+ `NULL` : Un pointeur vers les arguments de la fonction s'il y en a.

##### `pthread_join()`
Attend la fin de l'exécution du processus et revient au programme principal. Dans l'exemple, comme on a des boucles infinies on n'y arrivera jamais.
+ `t_led1` : La référence au thread créé
+ `NULL` : Une variable qui contiendra la valeur retournée par la fonction.

{{% notice primary "Pause pour le prof" %}}
1. Modifier les fonctions pour que les LED clignotent 10 fois seulement.
2. Comment changer la boucle FOR des fonctions pour que le thread 2 termine avant le premier?
{{% /notice %}}

#### Passage de paramètres
On aurait un meilleur programme si on avait une seule fonction `clignoter()` à laquelle on passerait toutes les variables:
```c
// 'ms' est le nombre de millisecondes
// 'gpio' est le numéro de broche GPIO
void *clignoter(int ms, int gpio) {
    for (int i=0;i<15;i++) {
        gpioWrite(gpio, 1);
        usleep(ms*000);
        gpioWrite(gpio, 0);
        usleep(ms*000);
    }
}
```

Mais la fonction passée à `pthread_create()` ne peut pas avoir plusieurs arguments: elle n'a droit qu'à un seul. Celui-ci peut cependant être un pointeur, donc on peut passer à la fonction un tableau (ou une [struct](https://www.w3schools.com/c/c_structs.php)) qui contient autant d'éléments qu'on veut, simplement en passant un pointeur sur ce tableau.

On réécrira donc la fonction pour que la variable passée soit un tableau de deux éléments; le premier est le nombre de millisecondes, et le deuxième est le numéro de GPIO:
```c
void *clignoter(int *args) {
    for (int i=0;i<15;i++) {
        gpioWrite(args[1], 1);
        usleep(args[0]*000);
        gpioWrite(args[1], 0);
        usleep(args[0]*000);
    }
}
```
Il faut aussi modifier la partie du programme où on crée les threads:
```c
    pthread_t t_led1,t_led2;

    int arg1[] = {250,LED1};
    int arg2[] = {500,LED2};

    if (pthread_create(&t_led1,NULL,clignoter,&arg1) != 0) {
        printf("Erreur à la création du thread pour LED 1.\n");
        return 1;
    }
    if (pthread_create(&t_led2,NULL,clignoter,&arg2) != 0) {
        printf("Erreur à la création du thread pour LED 2.\n");
        return 1;
    }
```
Si on compile ce programme on aura une erreur. Pourquoi?

La raison est la suivante: afin que la fonction passée à `pthread_create()` ne soit pas limitée dans les types d'arguments qu'elle peut avoir, le 4e paramètre (`&arg1` et `&arg2` dans l'exemple) sera passé à la fonction comme un pointeur de type `void *`. Puisque dans notre cas elle est définie avec un argument `int *`, on a une erreur.

La version qui fonctionne:
```c
void *clignoter(void *args) {
    int *valeurs = (int *)args;
    int ms = valeurs[0];
    int gpio = valeurs[1];

    for (int i=0;i<10;i++) {
        gpioWrite(gpio, 1);
        usleep(ms*1000);
        gpioWrite(gpio, 0);
        usleep(ms*1000);
    }
} 
```
On crée un tableau d'entiers, par exemple `arg1[]`, mais ce tableau "perd" en quelque sorte son type lorsqu'il est passé à la fonction `pthread_create()`. Il est ensuite passé à `clignoter()` comme un pointeur _void_, et on lui redonne le type `int *` pour accéder aux valeurs qu'il contient.

> `pthread_create()` travaille uniquement avec des **adresses**: celles de la fonction qu'elle doit appeler (c'est pourquoi `*clignoter()` est déclarée avec `*`) et celle d'un pointeur _void_ qui indique où se trouvent les arguments à passer à cette fonction. C'est à nous de s'assurer qu'on redonne ensuite le bon type à ce pointeur: si un pointeur a un type _void_, le compilateur ne sait pas quelle taille la variable occupe en mémoire et donc est incapable de lire sa valeur, de se déplacer à la prochaine valeur, etc.


#### Exercice 1
Pour l'instant dans notre fonction les LED clignotent 10 fois. Modifier le programme pour que le nombre de clignotements soit le 3e argument passé à la fonction.


#### Exercice 2
Changez la fonction pour que les LED clignotent toujours 10 fois à 100 ms d'intervalle, avec ces valeurs codées directement dans la fonction. La seule chose qu'il faut passer à `clignoter()` est donc un `int` qui désigne le GPIO. Refaites le programme pour que les variables `arg1` et `arg2` soient des nombres entiers.


#### Exercice 3
On se base sur l'exercice du chapitre précédent:
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define BUFFER_SIZE 10
#define LED_PIN 17
#define PORT 8888

// Modifier cette fonction pour qu'elle puisse être passée à pthread_create()
void clignoter(int gpio) {
    for (;;) {
        gpioWrite(gpio, 1);
        usleep(100000);
        gpioWrite(gpio, 0);
        usleep(100000);
    }
} 

int main() {
    int socket_local;
    struct sockaddr_in adr_local, adr_dist;
    char buffer[BUFFER_SIZE];

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_DGRAM, 0);
    memset(&adr_local, 0, sizeof(adr_local));
    adr_local.sin_family = AF_INET; 
    adr_local.sin_addr.s_addr = INADDR_ANY;
    adr_local.sin_port = htons(PORT);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (const struct sockaddr *)&adr_local, sizeof(adr_local));

    //Preparation de pigpio
    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Failed to initialize pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED_PIN, PI_OUTPUT);

    // Réception des messages  
    char input[BUFFER_SIZE];

    while (1) {
        int len,n;
        len = sizeof(adr_dist); 
        // Stocker le message reçu dans n
        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        // Voir quel mot a été reçu
        if (strcmp(buffer, "allume\n") == 0) {
            gpioWrite(LED_PIN, 1);
        }
        else if(strcmp(buffer,"ferme\n") == 0){
            gpioWrite(LED_PIN, 0);
        }
        else if(strcmp(buffer,"exit\n") == 0){
            gpioWrite(LED_PIN, 0);
            gpioTerminate();
            break;
        }
        else if(strcmp(buffer,"flash\n") == 0){
            // Créer ici le thread qui lance la fonction clignoter
        }
    }

    return 0;
}
```

Ajoutez à ce programme une commande "flash" qui aura pour effet de faire clignoter la LED en créant un thread.
