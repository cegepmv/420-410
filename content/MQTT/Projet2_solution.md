+++
title = 'Projet2_solution'
date = 2024-03-26T00:18:01-04:00
draft = true
+++

# Dépendances:
ads1115_rpi (https://github.com/giobauermeister/ads1115-linux-rpi)

# Flags de compilation
-lpigpio -lmosquitto -lpthread

# p2_pub.c:
```c
#include <stdio.h>
#include <mosquitto.h>
#include <pigpio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include "../include/ads1115_rpi.h"

#define ADS1115_ADDRESS 0x48 // Adresse I2C du ADS1115
#define MQTT_BROKER "mqttbroker.lan"
#define MQTT_TOPIC "p2" 
#define MQTT_PORT 1883
#define MQTT_QOS 0
#define BUFFER_LENGTH 255
#define HOSTNAME_MAX_LENGTH 8


void on_connect(struct mosquitto *mosq, void *userdata, int result) {
    if (result != 0) {
        fprintf(stderr, "Erreur: connexion broker MQTT.\n");
    }
}

char* r_trim(char* str) {
    int size = strlen(str);
    char* trimmed;
    if (strchr(str, '\n') != NULL) {
        size--;
    }
    trimmed = (char *)malloc(size * sizeof(char));
    strncpy(trimmed,str,size * sizeof(char));

    return trimmed;
}

int main() {

    char hostname[HOSTNAME_MAX_LENGTH];
    char message[BUFFER_LENGTH];

    // Trouver hostname
    if (gethostname(hostname, 8) != 0) {
        fprintf(stderr, "Erreur pour déterminer hostname\n");
        return 1;
    }

    // Initialiser GPIO
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }
    
    // Initialiser ADC
    if(openI2CBus("/dev/i2c-1") == -1) {
		return EXIT_FAILURE;
	}
	setI2CSlave(ADS1115_ADDRESS);

    // Initialiser mqtt
    struct mosquitto *mosq = NULL;
    mosquitto_lib_init();
    mosq = mosquitto_new(NULL, true, NULL);
    if (!mosq) {
        fprintf(stderr, "Erreur: création de l'instance mosquitto.\n");
        return 1;
    }

    // Connexion broker
    int rc;

    mosquitto_connect_callback_set(mosq, on_connect);
    rc = mosquitto_connect(mosq, MQTT_BROKER, MQTT_PORT, 60);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Connexion impossible au broker: %s\n", mosquitto_strerror(rc));
        return 1;
    }

	while(1) {

        sprintf(message,"%s|%.0f",hostname,readVoltage(0)/4.096*100);
        printf("%s\n",message);

        rc = mosquitto_publish(mosq, NULL, MQTT_TOPIC, strlen(message), message, MQTT_QOS, false);
        if (rc != MOSQ_ERR_SUCCESS) {
            fprintf(stderr, "Failed to publish message: %s\n", mosquitto_strerror(rc));
        }
        sleep(10);

;	} 

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```

# p2_sub.c
```c
#include <stdio.h>
#include <mosquitto.h>
#include <pigpio.h>
#include <unistd.h>
#include <pthread.h>
#include <stdlib.h>
#include <string.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include "../include/ads1115_rpi.h"

#define ADS1115_ADDRESS 0x48 // Adresse I2C du ADS1115
#define MQTT_BROKER "192.168.50.242"
#define MQTT_TOPIC "p2" 
#define MQTT_PORT 1883
#define MQTT_QOS 0
#define DELIMITEUR '|'
#define BUFFER_LENGTH 255
#define ARR_SIZE 5

int data_lumi[ARR_SIZE];
char* data_hosts[ARR_SIZE];
int array_index = 0;


char** split(char* message, char delim) {
    // Retourne un array de 2 éléments: le 1er est le string qui précède le delim,
    // le 2e est tout ce qui suit le delim. S'il y a plusieurs delim dans message
    // on ne tient compte que du premier
    char** data = (char**)malloc(2 * sizeof(char*)); // Allouer l'esapce dynamiquement
    char* delim_pos = strchr(message,delim);
    size_t taille = delim_pos - message; // Adr delim_pos - adr message = taille 1ere partie

    if (delim_pos != NULL) {
        data[0] = (char *)malloc((taille + 1) * sizeof(char));
        strncpy(data[0], message, taille);
        data[0][taille] = '\0';

        data[1] = strdup(delim_pos + 1);
    }
    return data;
}

void print_info() {
    
    int max_lumi = data_lumi[0];
    char* host = data_hosts[0];
    int sum = data_lumi[0];
    float avg;

    // Attendre 10s
    sleep(10);

    // Trouver valeur max, chercher le host au même index
    for(int i=1;i<ARR_SIZE;i++) {
        sum += data_lumi[i];
        if (data_lumi[i] > max_lumi) {
            max_lumi = data_lumi[i];
            host = data_hosts[i];
        }
    }
    avg = (float)sum / ARR_SIZE;

    printf("Max: %s (%i)\nMoy: %.2f\n-------------\n",host,max_lumi,avg);
    fflush(stdout);

    // Reinitialiser les variables
    array_index = 0;
    memset(data_lumi, 0, ARR_SIZE * sizeof(int));
}

void add_data(char* message){

    // Split message
    char** data = split(message,DELIMITEUR);

    // Ajouter données aux arrays
    int lumi = atoi(data[1]);
    data_lumi[array_index] = lumi;
    data_hosts[array_index] = data[0];

    array_index++;

    if (array_index == 5) {
        print_info();
    }

}

void on_connect(struct mosquitto *mosq, void *userdata, int result) {
    if (result == 0) {
        printf("Abonnement à %s...",MQTT_TOPIC);
        mosquitto_subscribe(mosq, NULL, "p2", MQTT_QOS);
        printf("ok.\n");
    } else {
        fprintf(stderr, "Erreur: connexion broker MQTT.\n");
    }
}

void on_message(struct mosquitto *mosq, void *userdata, const struct mosquitto_message *message) {
    //printf("Message: (%s) %s\n",message->topic, (char *)message->payload);
    add_data((char *)message->payload);
}


char* r_trim(char* str) {
    int size = strlen(str);
    char* trimmed;
    if (strchr(str, '\n') != NULL) {
        size--;
    }
    trimmed = (char *)malloc(size * sizeof(char));
    strncpy(trimmed,str,size * sizeof(char));

    return trimmed;
}



int main() {

    // Initialiser mqtt
    struct mosquitto *mosq = NULL;
    mosquitto_lib_init();
    mosq = mosquitto_new(NULL, true, NULL);
    if (!mosq) {
        fprintf(stderr, "Erreur: création de l'instance mosquitto.\n");
        return 1;
    }


    // Connexion broker
    int rc;
    mosquitto_connect_callback_set(mosq, on_connect);
    rc = mosquitto_connect(mosq, MQTT_BROKER, MQTT_PORT, 60);
    printf("Connexion à %s...",MQTT_BROKER);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Connexion impossible au broker: %s\n", mosquitto_strerror(rc));
        return 1;
    }
    printf("ok.\n");

    mosquitto_message_callback_set(mosq, on_message);

    mosquitto_loop_forever(mosq, 0, 1);

    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```