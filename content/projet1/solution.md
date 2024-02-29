+++
title = 'Solution'
date = 2024-02-26T20:18:15-05:00
draft = true
+++

# Exercice 1
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>
#include <stdlib.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include "../include/ads1115_rpi.h"

#define LED1 17
#define ADS1115_ADDRESS 0x48 // Adresse I2C du ADS1115

int main() {

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }
    
    // ADC
    if(openI2CBus("/dev/i2c-1") == -1) {
		return EXIT_FAILURE;
	}
	setI2CSlave(ADS1115_ADDRESS);

	while(1) {
        // Convertir valeur 0-3.3v à 0-255
        int dc = readVoltage(0) / 3.3 * 256;
		printf("%.2f\n", readVoltage(0));
        gpioPWM(LED1, dc); 
	} 

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```

# Exercice 2
#### projet1_cl.c
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>
#include <stdlib.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>
#include "../include/ads1115_rpi.h"

#define ADS1115_ADDRESS 0x48 // Adresse I2C du ADS1115

int main() {

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }
    
    // ADC
    if(openI2CBus("/dev/i2c-1") == -1) {
		return EXIT_FAILURE;
	}
	setI2CSlave(ADS1115_ADDRESS);

    // Connexion
    int sock = 0;
    struct sockaddr_in dest_addr;
    
    // Créer le socket
    sock = socket(AF_INET, SOCK_STREAM, 0);
    
    // Initialiser la struct de l'adresse IP 
    memset(&dest_addr, '0', sizeof(dest_addr));
    dest_addr.sin_family = AF_INET;
    dest_addr.sin_addr.s_addr = inet_addr("192.168.0.106");
    dest_addr.sin_port = htons(9090);

    // Créer la connexion
    connect(sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
    
    // Envoyer le message 
    int dc;
	while(1) {
        // Convertir valeur 0-3.3v à 0-255
        dc = readVoltage(0) / 3.3 * 256;
		printf("%d\n", dc);
        send(sock, &dc, sizeof(dc), 0);
	} 

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```

#### projet1_sv.c
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>
#include <stdlib.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>
#include "../include/ads1115_rpi.h"

#define LED1 17

int main() {

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Test LED
    gpioSetMode(LED1, PI_OUTPUT);
    gpioWrite(LED1, 1);
    usleep(1000000);
    gpioWrite(LED1, 0);
    usleep(1000000);

    int socket_local, socket_dist;
    struct sockaddr_in address;
    int addrlen = sizeof(address);

    // Créer le socket et initialiser l'adresse
    socket_local = socket(AF_INET, SOCK_STREAM, 0);
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(9090);

    // Associer le socket à l'adresse de l'interface
    bind(socket_local, (struct sockaddr *)&address, sizeof(address));
    
    // Attendre une connexion entrante
    listen(socket_local, 3);
    socket_dist = accept(socket_local, (struct sockaddr *)&address, (socklen_t*)&addrlen);

    // Réception des messages
    int dc;
    while(1) {
        int resultat;
        // Stocker le message
        resultat = recv(socket_dist, &dc, sizeof(dc),0);
      
        if (resultat != -1) {
            // Afficher le message
            printf("Reçu: %d\n", dc);
            gpioPWM(LED1, dc);
            fflush(stdout); 
        }        
    }

    close(socket_dist);
    close(socket_local);
	

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
abc```
