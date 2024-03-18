+++
title = 'Utilisation'
date = 2024-03-14T11:51:44-04:00
draft = true
+++

<!-- 
Dans ce cours le broker MQTT sera fourni par le prof, les étudiants n'ont pas besoin d'en configurer un.

Mais s'il faut en configurer un la procédure est la suivante (pour un hôte Debian 11):

apt install mosquitto mosquitto-clients

Dans /etc/mosquitto/mosquitto.conf, ajouter:

listener 1883 0.0.0.0
allow_anonymous true
sys_interval 5
-->

> Pour le cours, SVP utilisez l'agent MQTT disponible sur le réseau interne au nom `mqttbroker.lan`.

## Client MQTT _Mosquitto_
Dans ce cours nous utiliserons [Mosquitto](https://mosquitto.org/). Cette implémentation de MQTT, développée par la fondation _Eclipse_, fournit des utilitaires de ligne de commande pour envoyer et recevoir des messages (`mosquitto_pub` et `mosquitto_sub`), et aussi des librairies permettant d'implémenter des clients MQTT en C/C++.

Pour installer les logiciels requis, lancez la commande suivante:

```
apt update
apt install mosquitto-clients libmosquitto-dev
```

Pour tester l'installation, lancez le client "abonné" à l'aide de la commande suivante:
```
mosquitto_sub -h mqttbroker.lan -t 'test'
```

(Vous pouvez choisir un autre nom de rubrique que 'test')

Ensuite publiez un message (à partir du même hôte ou d'un autre hôte sur le même réseau) comme suit:
```
mosquitto_pub -h mqttbroker.lan -t 'test' -m 'Ceci est un test'
```

## "Subscriber" en C

mqtt_sub.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <mosquitto.h>

#define MQTT_BROKER_HOST "mqttbroker.lan"
#define MQTT_PORT 1883
#define MQTT_TOPIC "rubrique"
#define MQTT_QOS 1

void on_connect(struct mosquitto *mosq, void *userdata, int result) {
    if (result == 0) {
        mosquitto_subscribe(mosq, NULL, MQTT_TOPIC, MQTT_QOS);
    } else {
        fprintf(stderr, "Erreur: connexion broker MQTT.\n");
    }
}

void on_message(struct mosquitto *mosq, void *userdata, const struct mosquitto_message *message) {
    printf("Message: (%s) %s\n",message->topic, (char *)message->payload);
}

int main() {
    struct mosquitto *mosq = NULL;
    int rc;

    mosquitto_lib_init();

    mosq = mosquitto_new(NULL, true, NULL);
    if (!mosq) {
        fprintf(stderr, "Erreur: création de l'instance mosquitto.\n");
        return 1;
    }

    mosquitto_connect_callback_set(mosq, on_connect);
    mosquitto_message_callback_set(mosq, on_message);

    rc = mosquitto_connect(mosq, MQTT_BROKER_HOST, MQTT_PORT, 60);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Connexion impossible au broker: %s\n", mosquitto_strerror(rc));
        return 1;
    }

    mosquitto_loop_forever(mosq, -1, 1);

    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();

    return 0;
}
```

## "Publisher" en C

mqtt_pub.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <mosquitto.h>

#define MQTT_BROKER_HOST "mqttbroker.lan"
#define MQTT_PORT 1883
#define MQTT_TOPIC "rubrique"
#define MQTT_QOS 0
#define MQTT_MESSAGE "Allo le monde"

void on_connect(struct mosquitto *mosq, void *userdata, int result) {
    if (result != 0) {
        fprintf(stderr, "Erreur: connexion broker MQTT.\n");
    }
}

void on_publish(struct mosquitto *mosq, void *userdata, int mid) {
    printf("Message publié.\n");
}

int main() {
    struct mosquitto *mosq = NULL;
    int rc;

    mosquitto_lib_init();

    mosq = mosquitto_new(NULL, true, NULL);
    if (!mosq) {
        fprintf(stderr, "Erreur: création de l'instance mosquitto.\n");
        return 1;
    }

    mosquitto_connect_callback_set(mosq, on_connect);
    mosquitto_publish_callback_set(mosq, on_publish);

    rc = mosquitto_connect(mosq, MQTT_BROKER_HOST, MQTT_PORT, 60);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Connexion impossible au broker: %s\n", mosquitto_strerror(rc));
        return 1;
    }

    rc = mosquitto_publish(mosq, NULL, MQTT_TOPIC, strlen(MQTT_MESSAGE), MQTT_MESSAGE, MQTT_QOS, false);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Failed to publish message: %s\n", mosquitto_strerror(rc));
    }

    mosquitto_loop_forever(mosq, -1, 1);

    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();

    return 0;
}
```

## Exercice 1
L'agent MQTT _mqttbroker.lan_ diffuse des messages dans 2 rubriques: `ex1_temp` et `ex1_hum`. La première donne une température en Celsius et la 2e un pourcentage d'humidité.

Faites un programme qui lit les valeurs des 2 rubriques et affiche les données sur une même ligne à chaque 10 secondes, formatées comme suit:
```
root@pi:~# ./ex1
T: 23C | Hum: 45%
T: 23C | Hum: 45%
T: 22C | Hum: 45%
T: 22C | Hum: 45%
T: 22C | Hum: 46%
```

## Exercice 2
L'agent _mqttbroker.lan_ diffuse chaque 5 secondes des données de température (Celsius) et de bruit (décibels) sur les rubriques suivantes:
+ ex2/salle1/temp
+ ex2/salle1/db
+ ex2/salle2/temp
+ ex2/salle2/db
+ ex2/salle3/temp
+ ex2/salle3/db
+ ex2/salle4/temp
+ ex2/salle4/db

Faites un programme qui détecte quelles sont les salles où les valeurs de température et de bruit sont les plus élevées, et écrit dans un fichier CSV le nom de la salle et la valeur chaque fois que ce maximum change. Le fichier devrait contenir des valeurs similaires à celles-ci:
```
root@pi:~# cat données.csv
2024-03-15 15:41:35;salle2;78dB
2024-03-15 15:45:22;salle1;81dB
2024-03-15 15:46:15;salle2;28C
2024-03-15 15:54:12;salle2;29C
2024-03-15 15:58:35;salle1;19C
```

## Projet 2
Connectez un senseur de luminosité et l'écran LCD à votre Pi.

Vous devez faire 2 programmes:
+ **p2_pub**: Publie chaque 10 secondes dans la rubrique `p2` la valeur de luminosité lue sur le senseur;
+ **p2_sub**: À partir des données lues sur tous les Pi, affiche le nom du Pi où est la valeur maximale et la moyenne.

#### Spécifications:
+ Le message publié doit avoir le format `HOTE-VALEUR` (HOTE est le _hostname_ du Pi et VALEUR est la luminosité)
+ Le nom de l'agent et de la rubrique ne doivent pas être directement dans le code mais être écrits dans un fichier sur 2 lignes. La première ligne est le nom de l'agent et la deuxième est le nom de la rubrique. Le fichier doit se nommer `mqttcl.conf` et être dans le même répertoire que l'exécutable.
+ Les informations doivent être affichées comme suit sur l'écran LCD:
```
equipe2
21093
```


