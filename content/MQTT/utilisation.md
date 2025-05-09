+++
title = 'Utilisation'
date = 2024-03-14T11:51:44-04:00
draft = true
weight = 71
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


Dans ce cours nous utiliserons [Mosquitto](https://mosquitto.org/). Cette implémentation de MQTT, développée par la fondation _Eclipse_, fournit des utilitaires de ligne de commande pour envoyer et recevoir des messages (`mosquitto_pub` et `mosquitto_sub`), et aussi des librairies permettant d'implémenter des clients MQTT en python.

## Agent MQTT _Mosquitto_
Pour installer le service _mosquitto_, lancez la commande suivante:
```
sudo apt update
sudo apt install mosquitto
```
#### Configuration de base
Les paramètres de configuration de _mosquitto_ sont définis dans le fichier `/etc/mosquitto/mosquitto.conf`. 

Pour utiliser mosquitto sans authentification, assurez-vous d'avoir la valeur **true** pour la directive `allow_anonymous`. Aussi, vérifiez que `listener` est bien au port **1883** et que mosquitto écoute sur toutes les interfaces (valeur **0.0.0.0**). Ceci correspond aux lignes suivantes dans le fichier:
```
listener 1883 0.0.0.0
allow_anonymous true
```
Laissez les autres valeurs inchangées.

Lorsque vous avez modifié la configuration, lancez les commandes suivantes pour démarrer le service puis pour l'activer au démarrage du serveur:
```
systemctl start mosquitto
systemctl enable mosquitto
```
{{% notice primary "Note" %}}
Nous verrons dans la section [Sécurité](/420-410/mqtt/securite/) comment activer l'authentification et le chiffrement dans mosquitto.
{{% /notice %}}


## Client MQTT _Mosquitto_
Pour installer les logiciels et librairies pour la partie client, lancez la commande suivante:

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

```bash
    gcc -o mqtt_sub mqtt_sub.c -lmosquitto
```
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
```bash
    gcc -o mqtt_pub mqtt_pub.c -lmosquitto
```
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
<!-- Pour que les étudiants fassent les exercices 1 et 2 il faut avoir lancé le programme suivant avec l'adresse ou le nom du broker
en argument:
```python
import paho.mqtt.publish as publish
import sys, random, time
from threading import Thread

host = sys.argv[1]

def genNombre(compte,minVal,maxVal):
    # Générer le premier nombre dans la plage [minVal,maxVal]
    random_number = random.randint(minVal, maxVal)
    random_list = [random_number]

    # Générer les 99 nombres suivants
    for _ in range(compte):
        # Choisir un nombre aléatoire entre -1 et 1 pour ajouter ou soustraire au nombre précédent
        change = random.randint(-1, 1)
        # Si le nombre précédent est déjà au bord de la plage, ajuster la direction en conséquence
        if random_number == minVal:
            change = 1
        elif random_number == maxVal:
            change = -1
        # Ajouter ou soustraire le changement pour obtenir le prochain nombre
        random_number += change
        random_list.append(random_number)

    return random_list


def publishData(broker,top,minVal,maxVal,intervalle,prefixe):
    # Publier des nombres vers un broker MQTT
    while True:
        liste = genNombre(100,minVal,maxVal)
        for nombre in liste:
            publish.single(topic=top, payload=prefixe+str(nombre), hostname=broker)        
            time.sleep(intervalle)

    
tt = Thread(target=publishData,args=(host,'ex1_tmp',18,24,9,'t|'))
th = Thread(target=publishData,args=(host,'ex1_hum',34,45,11,'h|'))

allt = [Thread(target=publishData,args=(host,'ex2/salle1/temp',16,22,5,'1|t|')),
        Thread(target=publishData,args=(host,'ex2/salle1/db',50,75,5,'1|db|')),
        Thread(target=publishData,args=(host,'ex2/salle2/temp',18,24,5,'2|t|')),
        Thread(target=publishData,args=(host,'ex2/salle2/db',65,85,5,'2|db|')),
        Thread(target=publishData,args=(host,'ex2/salle3/temp',19,25,5,'3|t|')),
        Thread(target=publishData,args=(host,'ex2/salle3/db',70,90,5,'3|db|')),
        Thread(target=publishData,args=(host,'ex2/salle4/temp',30,75,5,'4|t|')),
        Thread(target=publishData,args=(host,'ex2/salle4/db',18,24,5,'4|db|')) 
    ]

tt.start()
th.start()
for t in allt:
    t.start()
    time.sleep(5/6)
```
-->
## Exercice 1
L'agent MQTT _mqttbroker.lan_ diffuse des messages dans 2 rubriques: `ex1_tmp` et `ex1_hum`. La première donne une température en Celsius et la 2e un pourcentage d'humidité.

Faites un programme qui lit les valeurs des 2 rubriques et affiche les données sur une même ligne à chaque 10 secondes, formatées comme suit:
```
root@pi:~# ./ex1
T: 23C | Hum: 45%
T: 23C | Hum: 45%
T: 22C | Hum: 45%
T: 22C | Hum: 45%
T: 22C | Hum: 46%
```
Attention, l'agent MQTT n'envoit pas lui-même les données aux 10 secondes...

{{% expand "Solution" %}}
```c
/*
Les données sont écrites dans des fichiers à mesure qu'elles arrivent. 
Une fonction lit le contenu de ce fichiers et l'affiche à chaque 10 sec.
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <mosquitto.h>
#include <pthread.h>

#define MQTT_BROKER_HOST "mqttbroker.lan"
#define MQTT_PORT 1883
#define MQTT_TOP_TMP "ex1_tmp"
#define MQTT_TOP_HUM "ex1_hum"
#define FILE_T "temp.txt"
#define FILE_H "humi.txt"

#define MQTT_QOS 1

/*
Écrit les données 'data'' dans le fichier 'filename' 
*/
int write_data(char* filename,char* data) {
    FILE *fichier;
    fichier = fopen(filename, "w");
    if (fichier == NULL) {
        printf("Erreur lors de l'ouverture du fichier.\n");
        return 1;
    }
    fprintf(fichier,data);
    fclose(fichier);
    return 0;
}

/*
Retourne un pointeur sur la chaine de caractères après le délimiteur.
extract_data("abcd*1234", '*'') retourne "1234"
*/
char* extract_data(char* data, char delim) {
    char* substr = strchr(data,delim);
    if (substr != NULL) {
        return substr+1;
    }
    return NULL;
}

/*
Prend les données dans les deux fichiers et affiche la chaine
chaque 10sec
*/
void print_data() {

    while(1) {
        sleep(10);

        FILE *fptr;
        fptr = fopen(FILE_T, "r");
        char temp[3];
        fgets(temp,3,fptr);
        fclose(fptr);

        fptr = fopen(FILE_H, "r");
        char humi[3];
        fgets(humi,3,fptr);
        fclose(fptr);

        fprintf(stdout,"T: %sC | Hum: %s\%\n",temp,humi);
        fflush(stdout);
    }
}

void on_connect(struct mosquitto *mosq, void *userdata, int result) {
    if (result == 0) {
        // S'abonner à 2 rubriques
        mosquitto_subscribe(mosq, NULL, MQTT_TOP_TMP, MQTT_QOS);
        mosquitto_subscribe(mosq, NULL, MQTT_TOP_HUM, MQTT_QOS);
    } else {
        fprintf(stderr, "Erreur: connexion broker MQTT.\n");
    }
}

void on_message(struct mosquitto *mosq, void *userdata, const struct mosquitto_message *message) {
    char* data = extract_data((char *)message->payload,'|');
    // message->topic détermine dans quel fichier on écrit
    if (strcmp(message->topic,MQTT_TOP_TMP) == 0) {
        write_data(FILE_T,data);
    } else {
        write_data(FILE_H,data);
    }
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

    // Lancer print_data dans un thread 
    pthread_t t_mosq;
    if (pthread_create(&t_mosq,NULL,print_data,NULL) != 0) {
        printf("Erreur à la création du thread pour publication.\n");
        return 1;
    }

    mosquitto_loop_forever(mosq, -1, 1);

    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();

    return 0;
}
```
{{% /expand %}}

## Exercice 2
Quels sont les sous-rubriques de _$SYS_ qui permettent d'obtenir les informations suivantes sur l'agent :
1.  Nombre d'octets de données reçues
2.  Nombre total de clients connectés
3.  Nombre de messages reçus durant les 15 dernières minutes
4.  Le nombre d'abonnements en cours

Cherchez la réponse dans la documentation ( https://mosquitto.org/man/mosquitto-8.html ), puis modifiez le programme client [mqtt_sub.c](https://github.com/cegepmv/410-code/blob/main/MQTT/mqtt_sub.c) pour afficher à la console ces 4 rubriques.


{{% expand "Solution" %}}
Ajouter les lignes suivantes dans la fonction `on_connect()`:
```c
void on_connect(struct mosquitto *mosq, void *userdata, int result) {
    if (result == 0) {
        mosquitto_subscribe(mosq, NULL, "$SYS/broker/bytes/received", MQTT_QOS);
        mosquitto_subscribe(mosq, NULL, "$SYS/broker/clients/connected", MQTT_QOS);
        mosquitto_subscribe(mosq, NULL, "$SYS/broker/load/messages/received/15min", MQTT_QOS);
        mosquitto_subscribe(mosq, NULL, "$SYS/broker/subscriptions/count", MQTT_QOS);
    } else {
        fprintf(stderr, "Erreur: connexion broker MQTT.\n");
    }
}
```
{{% /expand %}}



## Exercice 3
Créez le fichier `/etc/mosquitto/mosquittocl.conf`. Celui-ci contiendra 4 paramètres de configuration pour votre client: 
+ Le nom ou l'adresse IP de l'agent
+ Le nom du _topic_
+ Le nom d'utilisateur
+ Le mot de passe

Chaque élément devra être sur sa propre ligne, par exemple:
```
192.168.0.110
appli_mqtt
bob
abc-123
```
Modifiez le code du projet 2 pour que vos 2 programmes client utilisent les données dans ce fichier de configuration.

{{% expand "Solution" %}}
Ajouter une référence au fichier de configuration:
```C
#define MQTT_CLIENT_CONFIG "/root/410-projet2/mosquittocl.conf"
```
Ensuite, supprimer les variables MQTT_BROKER, MQTT_TOPIC.

Ajouter les globales suivantes:
```C
char* mqtt_broker;
char* mqtt_topic;
char* mqtt_user;
char* mqtt_psw;
```
Ajouter la fonction pour lire les variables dans le fichier:
```C
int get_config(char* filename) {
    FILE* f;
    char* ligne = NULL;
    size_t taille = 0;
    ssize_t read;

    f = fopen(filename,"r");
    if (f == NULL) {
        return 1;
    }
    read = getline(&ligne,&taille,f);
    mqtt_broker = (char *)malloc((strlen(ligne) + 1) * sizeof(char));
    strcpy(mqtt_broker, r_trim(ligne));

    read = getline(&ligne,&taille,f);
    mqtt_topic = (char *)malloc((strlen(ligne) + 1) * sizeof(char));
    strcpy(mqtt_topic, r_trim(ligne));

    read = getline(&ligne,&taille,f);
    mqtt_user= (char *)malloc((strlen(ligne) + 1) * sizeof(char));
    strcpy(mqtt_user, r_trim(ligne));

    read = getline(&ligne,&taille,f);
    mqtt_psw = (char *)malloc((strlen(ligne) + 1) * sizeof(char));
    strcpy(mqtt_psw, r_trim(ligne));

    fclose(f);
}
```
Ajouter le code suivant dans main():
```C
if (get_config(MQTT_CLIENT_CONFIG) != 0){
    fprintf(stderr, "Erreur: Lecture du fichier de configuration.\n");
    return 1;
}
mosquitto_username_pw_set(mosq,mqtt_user,mqtt_psw);
```
{{% /expand %}}

<!--
## Exercice 4
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
-->



