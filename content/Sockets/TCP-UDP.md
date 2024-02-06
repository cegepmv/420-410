+++
title = 'TCP et UDP'
date = 2024-02-06T12:05:07-05:00
weight = 20
draft = true
+++

serveur_tcp.c
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

    socket_local = socket(AF_INET, SOCK_STREAM, 0);
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    listen(socket_local, 3);
    socket_dist = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

    while(1) {
        int data;
        data = read(socket_dist, buffer, BUFFER_SIZE);
        if (data == 0) {
            printf("Déconnexion\n");
            break;
        } else {
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

client_tcp.c
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

    connect(sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
    
    send(sock, MESSAGE, strlen(MESSAGE), 0);
    close(sock);
    return 0;
}
```

serv_udp.c
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

    socket_local = socket(AF_INET, SOCK_DGRAM, 0);
    memset(&adr_local, 0, sizeof(adr_local));
    adr_local.sin_family = AF_INET; 
    adr_local.sin_addr.s_addr = INADDR_ANY;
    adr_local.sin_port = htons(PORT);

    bind(socket_local, (const struct sockaddr *)&adr_local, sizeof(adr_local));

    while (1) {
        int len, n;

        len = sizeof(adr_dist); 

        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        printf("Reçu: %s", buffer);
        fflush(stdout); 
    }

    close(socket_local);
    return 0;
}
```

client_udp.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

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

    sendto(sock, (const char *)MESSAGE, strlen(MESSAGE), MSG_CONFIRM, (const struct sockaddr *)&dest_addr, sizeof(dest_addr));

    close(sock);
    return 0;
}
```