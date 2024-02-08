+++
title = 'CorrigeExo'
date = 2024-02-07T22:01:17-05:00
draft = true
+++
# Client_udp
Changer l'ip!
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

    while(1){

    
        //Demander un message
        printf("######################################\n");
        printf("Entrez votre message : ");
        char message[100];
        fgets(message,sizeof(message),stdin);
        printf("\n");
        printf(message);
       
        printf("######################################\n");
        // Envoyer le message
        sendto(sock, (const char *)message, strlen(message), MSG_CONFIRM, (const struct sockaddr *)&dest_addr, sizeof(dest_addr));
         if(strcmp(message,"exit\n") == 0){
            break;
         }
    }
    // Fermer le socket
    close(sock);
    return 0;
}
```
# Serveur_udp
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define PIN_NUMBER 18
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

    //Preparation de pigpio
    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Failed to initialize pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(PIN_NUMBER, PI_OUTPUT);

    // Réception des messages
    while (1) {
        int len, n;

        len = sizeof(adr_dist); 
        // Stocker le message reçu dans n
        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        // Afficher
        printf("Reçu: %s", buffer);
        if(strcmp(buffer,"allume\n") == 0){
             gpioWrite(PIN_NUMBER, 1);
        }
        else if(strcmp(buffer,"ferme\n") == 0){
             gpioWrite(PIN_NUMBER, 0);
        }
        else if(strcmp(buffer,"exit\n") == 0){
             gpioWrite(PIN_NUMBER, 0);
             // Terminate the pigpio library
             gpioTerminate();
             break;
        } 
        
        fflush(stdout); 
    }

    close(socket_local);
    return 0;
}

```