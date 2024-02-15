+++
title = 'TCP et UDP'
date = 2024-02-06T12:05:07-05:00
weight = 20
draft = false
+++

Pour établir une communication entre un Raspberry Pi et un autre hôte sur le réseau, on peut adopter l'approche client-serveur: 
+ Le _serveur_ est celui qui reçoit les messages
+ Le _client_ est celui qui envoit les messages

Tout dépendant des applications, le Pi peut être n'importe lequel des deux. 

Deux protocoles réseau peuvent être utilisés pour envoyer des messages entre un client et un serveur, soit _TCP_ et _UDP_.
+ TCP: une connexion est établie entre client et serveur. Cette connexion est ensuite utilisée pour que les deux s'échangent des messages. Lorsqu'un des deux termine la connexion, celle-ci est fermée.
+ UDP: le client envoit des messages au serveur sans établir de connexion préalable. Il n'est donc pas possible de savoir si le serveur est en ligne et prêt à recevoir les messages.

Pour communiquer en utilisant TCP ou UDP, il faut utiliser les _sockets_.

Un socket représente le point terminal d'une communication entre deux hôtes sur un réseau. Il peut être la source ou la destination d'un message. Les sockets sont définis par une adresse IP et un port et sont utilisés par les programmes pour envoyer et recevoir des messages.

## Client
#### UDP
Le programme suivant envoit un message par UDP sur un réseau. L'adresse de destination est `127.0.0.1` et le port est `8888`:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8888
#define DEST_IP "127.0.0.1"
#define MESSAGE "hello!\n"

int main() {
    int sock = 0;
    struct sockaddr_in dest_addr;

    // Créer le socket
    sock = socket(AF_INET, SOCK_DGRAM, 0);
    
    // Initialiser la struct de l'adresse IP 
    memset(&dest_addr, 0, sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_addr.s_addr = inet_addr(DEST_IP);
    dest_addr.sin_port = htons(PORT);

    // Envoyer le message
    sendto(sock, (const char *)MESSAGE, strlen(MESSAGE), MSG_CONFIRM, (const struct sockaddr *)&dest_addr, sizeof(dest_addr));

    // Fermer le socket
    close(sock);
    return 0;
}
```

Vous pouvez tester ce programme comme suit: à partir d'un hôte sur le même réseau que le Pi, ouvrez un port UDP en mode "listen" avec la commande `nc`. Cet hôte joue donc le rôle de serveur. Par exemple, pour ouvrir le port UDP 8888 la commande est la suivante :
```bash
nc -ulp 8888
``` 
> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`.

#### TCP
Le programme suivant envoit un message par TCP sur un réseau. L'adresse de destination est `127.0.0.1` et le port est `8888`:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8888
#define DEST_IP "127.0.0.1"
#define MESSAGE "hello!\n"

int main() {
    int sock = 0;
    struct sockaddr_in dest_addr;
    
    // Créer le socket
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    // Initialiser la struct de l'adresse IP 
    memset(&dest_addr, '0', sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_addr.s_addr = inet_addr(DEST_IP);
    dest_addr.sin_port = htons(PORT);

    // Créer la connexion
    connect(sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
    
    // Envoyer le message et fermer la connexion
    send(sock, MESSAGE, strlen(MESSAGE), 0);
    close(sock);
    return 0;
}
```
Vous pouvez tester ce programme avec la commande `nc`. À partir d'un hôte sur le même réseau que le Pi (qui aura le rôle de serveur), ouvrez un port TCP en mode "listen" au port UDP 8888:
```bash
nc -lp 8888
``` 
> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`; ensuite exécutez-le.


## Serveur
#### UDP
Le programme suivant ouvre un socket au port UDP 8888 et affiche les messages entrants:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8888
#define BUFFER_SIZE 1024

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

    // Réception des messages
    while (1) {
        int len, n;

        len = sizeof(adr_dist); 
        // Stocker le message reçu dans n
        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        // Afficher
        printf("Reçu: %s", buffer);
        fflush(stdout); 
    }

    close(socket_local);
    return 0;
}
```
Pour tester ce programme à partir d'un autre hôte (qui agit comme client), lancez la commande `nc` (remplacez 1.2.3.4 par l'adresse IP du serveur) et tapez le message que vous voulez envoyer:
```bash
nc -u 1.2.3.4 8888
Bonjour!
```

#### TCP
Le programme suivant ouvre un socket au port TCP 8888 et attend les connexions entrantes. Lorsque la connexion est établie il affiche les messages et termine la connexion lorsque le message reçu est vide (taille 0):

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8888
#define BUFFER_SIZE 1024

int main() {
    int socket_local, socket_dist;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_STREAM, 0);
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    // Attendre une connexion entrante
    listen(socket_local, 3);
    socket_dist = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

    // Réception des messages
    while(1) {
        int datalen;
        // Stocker le message
        datalen = read(socket_dist, buffer, BUFFER_SIZE);
        if (datalen == 0) {
            printf("Déconnexion\n");
            break;
        } else { // Afficher le message
            printf("Reçu: %s", buffer);
            fflush(stdout); 
        }
        memset(buffer, 0, BUFFER_SIZE); 
    }

    close(socket_dist);
    close(socket_local);
    return 0;
}
```
Pour tester ce programme à partir d'un autre hôte (qui agit comme client), lancez la commande `nc` (remplacez 1.2.3.4 par l'adresse IP du serveur) et tapez le message que vous voulez envoyer:
```bash
nc 1.2.3.4 8888
Bonjour!
```

# Exercices

## UDP

Créez un **serveur** UDP sur votre PI qui s'attend à recevoir l'une des trois commande suivante: allume, ferme, exit.

- allume :  allume une led
- ferme : ferme cette led
- exit : S'assure de fermer la led et de bien fermer le serveur.

Créez ensuite le **client** UDP sur votre ordinateur (pas le Pi). Ce client doit pouvoir envoyer ces trois messages mentionné ci-dessus. 

## TCP
 > [Source](https://github.com/cegepmv/410-code.git)

1. Faites deux programmes (`serveur1.c` et `client1.c`) ayant les mêmes fonctionnalités que celui de l'exercice précédent, mais en utilisant une connexion TCP. Au message _exit_, la connexion TCP est fermée des deux côtés.
{{% expand "Solution" %}}
client1.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 9090
#define DEST_IP "10.10.20.245"

int main() {
    int sock = 9090;
    struct sockaddr_in dest_addr;
    
    // Créer le socket
    sock = socket(AF_INET, SOCK_STREAM, 0);            // ** SOCK_STREAM ** //
    
    // Initialiser la struct de l'adresse IP 
    memset(&dest_addr, '0', sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_addr.s_addr = inet_addr(DEST_IP);
    dest_addr.sin_port = htons(PORT);

    // Créer la connexion
    connect(sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
    
    while (1) {
        printf("> ");
        char message[100];

        fgets(message,sizeof(message),stdin);
        send(sock, message, strlen(message), 0);       // ** MODIFIÉ **//

        if(strcmp(message,"exit\n") == 0){
            break;
        } 
    }
    
    close(sock);
    return 0;
}
```
serveur1.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define LED_PIN 17
#define PORT 9090
#define BUFFER_SIZE 1024

int main() {
    int socket_local, socket_desc;
    struct sockaddr_in address;
    int addrlen = sizeof(address);      // ** AJOUTE ** //
    char buffer[BUFFER_SIZE] = {0};

    // Initialiser GPIO
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_STREAM, 0);         // ** SOCK_STREAM ** //
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    // Attendre une connexion entrante
    listen(socket_local, 3);
    socket_desc = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

    // Réception des messages
    while(1) {
        int datalen;

        // Stocker le message
        datalen = read(socket_desc, buffer, BUFFER_SIZE);       // ** read ** //
        if (datalen != 0) {
            printf("< %s",buffer);
            if(strcmp(buffer,"allume\n") == 0){
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
        }
        memset(buffer, 0, BUFFER_SIZE); 
    }

    close(socket_desc);              // ** AJOUTE ** //
    close(socket_local);
    return 0;
}
```
{{% /expand %}}

2. Ajoutez le fonctionnalité suivante: le serveur répond "OK" au client si la commande est _allume_, _ferme_ ou _exit_ ou "ERR" autrement (`serveur2.c` et `client2.c`).
{{% expand "Solution" %}}
client2.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 9090
#define DEST_IP "10.10.20.245"
#define ANSWER_LEN 4                // ** AJOUTE ** //

int main() {
    int sock = 0;
    struct sockaddr_in dest_addr;
    
    // Créer le socket
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    // Initialiser la struct de l'adresse IP 
    memset(&dest_addr, '0', sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_addr.s_addr = inet_addr(DEST_IP);
    dest_addr.sin_port = htons(PORT);

    // Créer la connexion
    connect(sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
    
    while (1) {
        printf("> ");
        char message[100];
        char answer[4];

        fgets(message,sizeof(message),stdin);
        send(sock, message, strlen(message), 0);

        if(strcmp(message,"exit\n") == 0){
            break;
        } else {                                // ** ELSE AJOUTE ** //
            // Afficher réponse
            recv(sock, answer, ANSWER_LEN, 0);
            printf("< %s\n",answer);
        }
    }
    
    close(sock);
    return 0;
}
```
serveur2.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define LED_PIN 17
#define PORT 9090
#define BUFFER_SIZE 1024
#define ANSWER_LEN 4                            // ** AJOUTE ** //

int main() {
    int socket_local, socket_desc;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    char answer[4];

    // Initialiser GPIO
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_STREAM, 0);
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    // Attendre une connexion entrante
    listen(socket_local, 3);
    socket_desc = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

    // Réception des messages
    while(1) {
        int datalen;
        strcpy(answer,"OK");                        // ** AJOUTE ** //

        // Stocker le message
        datalen = read(socket_desc, buffer, BUFFER_SIZE);
        if (datalen != 0) {
            printf("< %s",buffer);
            if(strcmp(buffer,"allume\n") == 0){
                gpioWrite(LED_PIN, 1);
            }
            else if(strcmp(buffer,"ferme\n") == 0){
                gpioWrite(LED_PIN, 0);
            }
            else if(strcmp(buffer,"exit\n") == 0){
                gpioWrite(LED_PIN, 0);
                gpioTerminate();
                break;
            } else {                                // ** AJOUTE ** //
                strcpy(answer,"ERR");
            }
            send(socket_desc, answer, ANSWER_LEN, 0);
        }
        memset(buffer, 0, BUFFER_SIZE); 
    }

    close(socket_desc);
    close(socket_local);
    return 0;
}
```
{{% /expand %}}

3. Modifiez votre programme: lorsque le client envoit _exit_, la connexion TCP est terminée, le client se termine, mais le serveur continue à attendre d'autres connexions TCP (`serveur3.c` et `client3.c`).
{{% expand "Solution" %}}
client3.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 9090
#define DEST_IP "10.10.20.245"
#define ANSWER_LEN 4                

int main() {
    int sock = 0;
    struct sockaddr_in dest_addr;
    
    // Créer le socket
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    // Initialiser la struct de l'adresse IP 
    memset(&dest_addr, '0', sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_addr.s_addr = inet_addr(DEST_IP);
    dest_addr.sin_port = htons(PORT);

    // Créer la connexion
    connect(sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
    
    while (1) {
        printf("> ");
        char message[100];
        char answer[4];

        fgets(message,sizeof(message),stdin);
        send(sock, message, strlen(message), 0);

        // Afficher réponse                     // ** MODIFIE ** //
        recv(sock, answer, ANSWER_LEN, 0);
        printf("< %s\n",answer);
        if(strcmp(answer,"BYE") == 0) {
            break;
        }
    }
    
    close(sock);
    return 0;
}
```
serveur3.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define LED_PIN 17
#define PORT 9090
#define BUFFER_SIZE 1024
#define ANSWER_LEN 4                            

int main() {
    int socket_local, socket_desc;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    char answer[4];

    // Initialiser GPIO
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_STREAM, 0);
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    while (1) {                                     // ** AJOUTE WHILE ** //
        // Attendre une connexion entrante
        listen(socket_local, 3);
        socket_desc = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

    
        // Réception des messages
        while(1) {
            int datalen;
            strcpy(answer,"OK");

            // Stocker le message
            datalen = read(socket_desc, buffer, BUFFER_SIZE);
            if (datalen != 0) {
                printf("< %s",buffer);
                if(strcmp(buffer,"allume\n") == 0){
                    gpioWrite(LED_PIN, 1);
                }
                else if(strcmp(buffer,"ferme\n") == 0){
                    gpioWrite(LED_PIN, 0);
                }
                else if(strcmp(buffer,"exit\n") == 0){
                    gpioWrite(LED_PIN, 0);
                    strcpy(answer,"BYE");                       // ** AJOUTE ** //
                    send(socket_desc, answer, ANSWER_LEN, 0);   // ** AJOUTE ** //
                    break;
                } else {
                    strcpy(answer,"ERR");
                }
                send(socket_desc, answer, ANSWER_LEN, 0);
            }
            memset(buffer, 0, BUFFER_SIZE); 
        }

        close(socket_desc);                     // ** MODIFIÉ ** //
    }
    close(socket_local);
    gpioTerminate();
    return 0;
}
```
{{% /expand %}}

4. Ajoutez un 2e module LED sur votre Pi. Modifiez le programme (`serveur4.c` et `client4.c`) pour que les commandes envoyées permettent de spécifier laquelle des 2 LED allumer (les messages "led1" et "led2" doivent être envoyés au serveur pour qu'il sache quelle LED allumer ou éteindre). Les messages possibles du client sont donc: `led1, led2, allume, ferme, exit`. Les messages du serveur sont `OK, ERR, BYE`.
{{% expand "Solution" %}}
client4.c

_Pas de changements_

serveur4.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define LED_PIN 17
#define PORT 9090
#define BUFFER_SIZE 1024
#define ANSWER_LEN 4                
#define LED_1 17                    // ** AJOUTE ** //
#define LED_2 24                    // ** AJOUTE ** //

int main() {
    int socket_local, socket_desc;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    char answer[4];
    int led_pin = LED_1;            // ** AJOUTE ** //

    // Initialiser GPIO
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_STREAM, 0);
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    while (1) {
        // Attendre une connexion entrante
        listen(socket_local, 3);
        socket_desc = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

        // Réception des messages
        while(1) {
            int datalen;
            strcpy(answer,"OK");

            // Stocker le message
            datalen = read(socket_desc, buffer, BUFFER_SIZE);
            if (datalen != 0) {
                printf("< %s",buffer);
                if(strcmp(buffer,"allume\n") == 0){
                    gpioWrite(led_pin, 1);
                }
                else if(strcmp(buffer,"ferme\n") == 0){
                    gpioWrite(led_pin, 0);
                }
                else if(strcmp(buffer,"exit\n") == 0){
                    gpioWrite(LED_1, 0);                // ** AJOUTE ** //
                    gpioWrite(LED_2, 0);                // ** AJOUTE ** //
                    strcpy(answer,"BYE");
                    send(socket_desc, answer, ANSWER_LEN, 0);
                    break;
                }
                else if(strcmp(buffer,"led1\n") == 0){  // ** AJOUTE ** //
                    led_pin = LED_1;
                }
                else if(strcmp(buffer,"led2\n") == 0){  // ** AJOUTE ** //
                    led_pin = LED_2;
                }
                else {
                    strcpy(answer,"ERR");
                }
                send(socket_desc, answer, ANSWER_LEN, 0);
            }
            memset(buffer, 0, BUFFER_SIZE); 
        }

        close(socket_desc);
    }
    close(socket_local);
    gpioTerminate();
    return 0;
}
```
{{% /expand %}}

