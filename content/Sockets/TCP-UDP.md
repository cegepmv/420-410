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

Créez un serveur UDP sur votre PI qui s'attend à recevoir l'une des trois commande suivante: allume, ferme, exit.

- allume :  allume une led
- ferme : ferme cette led
- exit : S'assure de fermer la led et de bien fermer le serveur.

Créez ensuite le client UDP sur votre ordinateur(pas le Pi). Ce client doit pouvoir envoyer ces trois messages mentionné ci-dessus. 