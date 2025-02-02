+++
title = 'Exercices'
date = 2025-01-26T15:07:06-05:00
draft = false
weight = 21
+++


### UDP

Créez un **serveur** UDP sur votre PI qui s'attend à recevoir l'une des trois commandes suivante: `allume`, `ferme`, `exit`.

- `allume` : Allume une led
- `ferme` : Éteint la led
- `exit` : S'assure de fermer la led et de bien fermer le serveur.

Créez ensuite le **client** UDP sur votre ordinateur (pas le Pi). Le client doit pouvoir envoyer ces trois messages. 


### TCP

1. Faites deux programmes (`serveur1.c` et `client1.c`) ayant les mêmes fonctionnalités que celui de l'exercice précédent, mais en utilisant une connexion TCP. Au message _exit_, la connexion TCP est fermée des deux côtés.
{{% expand "Solution" %}}
client1.py
```python
import socket

PORT = 9090
DEST_IP = "10.10.20.245"

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
dest_addr = (DEST_IP, PORT)
sock.connect(dest_addr)

while True:
    message = input("> ")
    sock.send(message.encode() + b'\n')
    
    if message == "exit":
        break

sock.close()
```
serveur1.py
```python
import socket
import pigpio

LED_PIN = 17
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

socket_local.listen(3)
print(f"Écoute au port: {PORT}...")

socket_desc, client_address = socket_local.accept()
print(f"Socket distant: {client_address}")

while True:
    data = socket_desc.recv(BUFFER_SIZE)
    if data:
        message = data.decode().strip()
        print(f"< {message}")
        
        if message == "allume":
            pi.write(LED_PIN, 1)
        elif message == "ferme":
            pi.write(LED_PIN, 0)
        elif message == "exit":
            pi.write(LED_PIN, 0)
            pi.stop()
            break

socket_desc.close()
socket_local.close()
```
{{% /expand %}}

2. Ajoutez le fonctionnalité suivante: le serveur répond "OK" au client si la commande est _allume_, _ferme_ ou _exit_ ou "ERR" autrement (`serveur2.c` et `client2.c`).
<!--
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
-->

3. Modifiez votre programme: lorsque le client envoit _exit_, la connexion TCP est terminée, le client se termine, mais le serveur continue à attendre d'autres connexions TCP (`serveur3.c` et `client3.c`).

<!--
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
-->

4. Ajoutez une 2e LED sur votre Pi. Modifiez le programme serveur pour que les commandes envoyées permettent de spécifier laquelle des 2 LED allumer (les messages "led1" et "led2" doivent être envoyés au serveur pour qu'il sache quelle LED allumer ou éteindre). Le programme client n'a pas besoin d'être modifié. Les messages possibles du client sont donc: `led1, led2, allume, ferme, exit`. Les messages du serveur sont `OK, ERR, BYE`.

<!--
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
-->
