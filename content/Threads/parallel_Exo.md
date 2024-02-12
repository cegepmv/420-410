+++
title = 'Parallel_Exo'
date = 2024-02-11T22:40:42-05:00
draft = true
+++

#### Exercice 1
Pour l'instant dans notre fonction les LED clignotent 10 fois. Modifier le programme pour que le nombre de clignotements soit le 3e argument passé à la fonction.

Solution
```c
void *clignoter(void *args) {
    int *valeurs = (int *)args;
    int ms = valeurs[0];
    int gpio = valeurs[1];
    int n = valeurs[2];  // Ajouté cette ligne

    for (int i=0;i<n;i++) { // Remplacé 10 par n
        gpioWrite(gpio, 1);
        usleep(ms*1000);
        gpioWrite(gpio, 0);
        usleep(ms*1000);
    }
} 
(...)
    int arg1[] = {250,LED1,5}; // Ajouter un élément
    int arg2[] = {500,LED2,6};
(...)
```


#### Exercice 2
Changez la fonction pour que les LED clignotent toujours 10 fois à 100 ms d'intervalle, avec ces valeurs codées directement dans la fonction. La seule chose qu'il faut passer à `clignoter()` est donc un `int` qui désigne le GPIO. Refaites le programme pour que les variables `arg1` et `arg2` soient des nombres entiers.

Solution
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>
#include <pthread.h>
#define LED1 17
#define LED2 24

void *clignoter(void *args) {
    int gpio = *(int *)args; // args est un pointeur d'entiers; on le déréférence pour avoir la valeur du gpio

    for (int i=0;i<10;i++) {
        gpioWrite(gpio, 1);
        usleep(100000);
        gpioWrite(gpio, 0);
        usleep(100000);
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
    int arg1 = LED1;  // Valeurs des GPIO
    int arg2 = LED2;

    if (pthread_create(&t_led1,NULL,clignoter,&arg1) != 0) {
        printf("Erreur à la création du thread pour LED 1.\n");
        return 1;
    }
    if (pthread_create(&t_led2,NULL,clignoter,&arg2) != 0) {
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

Solution
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

// ******* Modification
//
void *clignoter(void *args) {               
    int gpio = *(int *)args; 

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

    // ******* Modification
    //
    pthread_t thread;               
    char input[BUFFER_SIZE];

    while (1) {
        int len,n;
        len = sizeof(adr_dist); 
        // Stocker le message reçu dans n
        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        // Voir quel mot a été reçu
        if (strcmp(buffer, "allume\n") == 0) {
            //pthread_cancel(thread);
            gpioWrite(LED_PIN, 1);
        }
        else if(strcmp(buffer,"ferme\n") == 0){
            //pthread_cancel(thread);
            gpioWrite(LED_PIN, 0);
        }
        else if(strcmp(buffer,"exit\n") == 0){
            //pthread_cancel(thread);
            gpioWrite(LED_PIN, 0);
            gpioTerminate();
            break;
        }
        else if(strcmp(buffer,"flash\n") == 0){       

            // ******* Modification
            //
            int arg = LED_PIN;
            if (pthread_create(&thread, NULL, clignoter, &arg) != 0) {
                printf("Erreur de création du thread.\n");
                return 1;
            }
        }
    }

    return 0;
}
```

Problème: le thread n'arrête jamais. On veut qu'il s'arrête à **allume**, **ferme** et **exit**. `pthread_cancel()` le supprime sans attendre qu'il se termine (comme avec `pthread_join()`).

Solution 2
```c
        // Voir quel mot a été reçu
        if (strcmp(buffer, "allume\n") == 0) {
            pthread_cancel(thread);
            gpioWrite(LED_PIN, 1);
        }
        else if(strcmp(buffer,"ferme\n") == 0){
            pthread_cancel(thread);
            gpioWrite(LED_PIN, 0);
        }
        else if(strcmp(buffer,"exit\n") == 0){
            pthread_cancel(thread);
            gpioWrite(LED_PIN, 0);
            gpioTerminate();
            break;
        }
        else if(strcmp(buffer,"flash\n") == 0){
            int arg = LED_PIN;
            if (pthread_create(&thread, NULL, clignoter, &arg) != 0) {
                printf("Erreur de création du thread.\n");
                return 1;
            }
        }
```

Problème: si on commence par **allume** ou **ferme**, on appelle `pthread_cancel(thread)` avant que le thread existe. Il faut donc:
+ L'initialiser : pthread_t thread = pthread_self()
+ Vérifier que c'est le bon thread : `if(pthread_equal(thread, pthread_self()) == 0) { ... }`

Solution 3
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

void *clignoter(void *args) {
    int gpio = *(int *)args; 

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
    // ******* Modification
    //
    pthread_t thread = pthread_self();
    char input[BUFFER_SIZE];

    while (1) {
        int len,n;
        len = sizeof(adr_dist); 
        // Stocker le message reçu dans n
        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        // Voir quel mot a été reçu
        if (strcmp(buffer, "allume\n") == 0) {
            
            // ******* Modification
            //
            if(pthread_equal(thread, pthread_self()) == 0){
                 pthread_cancel(thread);
            }
            gpioWrite(LED_PIN, 1);
        }
        else if(strcmp(buffer,"ferme\n") == 0){
            if(pthread_equal(thread, pthread_self()) == 0){
                 pthread_cancel(thread);
            }
            gpioWrite(LED_PIN, 0);
        }
        else if(strcmp(buffer,"exit\n") == 0){
            if(pthread_equal(thread, pthread_self()) == 0){
                 pthread_cancel(thread);
            }
            gpioWrite(LED_PIN, 0);
            gpioTerminate();
            break;
        }
        else if(strcmp(buffer,"flash\n") == 0){
            int arg = LED_PIN;
            if (pthread_create(&thread, NULL, clignoter, &arg) != 0) {
                printf("Erreur de création du thread.\n");
                return 1;
            }
        }
    }

    return 0;
}
```

