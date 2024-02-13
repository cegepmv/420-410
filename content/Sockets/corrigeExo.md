+++
title = 'CorrigeExo'
date = 2024-02-07T22:01:17-05:00
draft = true
+++
## UDP
#### Client
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
#### Serveur
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

## TCP
#### Numéro 1
client1.c
```c
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

        fgets(message,sizeof(message),stdin);
        send(sock, message, strlen(message), 0);

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

#### Numéro 2
client2c
```c
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
}```

serveur2.c
```c
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


#### Numéro 3
client3.c
```c
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


#### Numéro 4
client4.c (PAS MODIFIÉ)
```c
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

        // Afficher réponse
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

serveur4.c
```c
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
                    gpioWrite(LED_0, 0);                // ** AJOUTE ** //
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
