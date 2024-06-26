+++
title = 'Serveur classique'
date = 2024-04-21T20:25:37-04:00
draft = false
weight = 80
+++

Bluetooth a été inventé pour remplacer les câbles qu'on utilisait pour connecter les appareils et leurs périphériques, comme par exemple les écouteurs, les souris ou les manettes de jeu. 

Il est basé sur une architecture client-serveur, mais permet seulement une communication de 1 à 1 entre deux composantes: les multiples clients d'un même serveur ne peuvent pas utiliser bluetooth pour communiquer entre eux.

En plus de la version habituelle du protocole, aussi nommée _classique_, il existe une version allégée nommée _Bluetooth LE_ (aussi _BLE_, pour "Bluetooth low energy"). Celle-ci a été conçue expressément pour les objets connectés et se caractérise par les aspects suivants:
+ Faible consommation d'énergie
+ Taux de transferts plus bas
+ Mieux adaptée aux transferts de petites quantités de données

Même si de nombreuses composantes supportent les deux versions de Bluetooth, les deux sont assez différents pour être considérés comme deux protocoles distincts. Il est donc possible que des composantes supportent l'un mais pas l'autre.

Dans cette section nous allons tester la communication _blutooth classique_ entre un RaspberryPi et un téléphone _Android_ en utilisant la librairie **btferret**.

## Prérequis
### Sur le téléphone Android
+ Installer l'application _Serial Bluetooth Terminal_ 
    + [Play Store](https://play.google.com/store/apps/details?id=de.kai_morich.serial_bluetooth_terminal)
    + [Code source](https://github.com/kai-morich/SimpleBluetoothTerminal?tab=readme-ov-file)
+ Trouver l'adresse MAC de l'interface bluetooth du téléphone
+ Activer bluetooth

 
### Sur le Pi
+ Cloner le projet [btferret](https://github.com/petzval/btferret)
+ Supprimer le contenu du fichier `devices.txt` à la racine du projet

## _btferret_
Le projet contient de nombreux programmes:
+ `btlib`: la librairie bluetooth et BLE
+ `btferret`: le programme linux pour utiliser bluetooth
+ `bluedot`: une application client-serveur pour Android
+ `classic_client`, `classic_server`: des applications client-serveur 
+ `le_client`, `le_server`: des applications client-serveur pour BLE
+ `keyboard`: permet au Pi de se connecter comme un clavier BT
+ `rn4020`: programme pour communiquer avec le module RN4020 (BLE)
+ `sample`: exemple des différents types de connexions supportées par BT

### Première exécution
Pour compiler puis exécuter _btferret_, lancez les commandes suivantes:
```
gcc btferret.c btlib.c -o btferret
./btferret
```
Le programme est un utilitaire en ligne de commande qui utilise la librairie **btlib**. 

À la première exécution du programme, on a le message suivant:
```
It should be added to the devices.txt file as follows:
DEVICE=name (e.g. My Pi) TYPE=MESH  NODE=choose (e.g. 1)  ADDRESS=E4:5F:01:EC:63:4E
```

Donc on ajoute une ligne avec les infos du Pi dans `devices.txt` (remplacez le nom et l'adresse MAC par vos propres valeurs):
```
DEVICE=pi-o TYPE=MESH  NODE=1  ADDRESS=E4:5F:01:EC:63:4E
```

Relancez ensuite le programme pour prendre compte des changements.

Vous devriez voir le contenu de `devices.txt` suivi d'une invite de commandes:
```
Initialising...
Device data from devices.txt file
DEVICE=pi-o TYPE=MESH  NODE=1  ADDRESS=E4:5F:01:EC:63:4E
h = help
>
```

Pour voir les commandes possibles, faites `h`. Parmi elles, voici celles qui sont utiles lors des premières phases de la communication:
- `a` et `b` pour chercher différents type de périphériques (BT et BLE)
- `i` pour des informations sur les périphériques détectés
- `v` pour voir les services disponibles sur un périphérique
- `c` pour se connecter sur un serveur BT
- `s` pour se mettre en mode serveur

### Établir une connexion
Avant que deux appareils puissent s'échanger des données, quelques étapes sont nécessaires:
1. Découverte des périphériques: l'appareil recherche les clients et serveurs à promximité
2. Appariement ("pairing"): les deux appareils s'échangent des informations qui leur permettront de se reconnaître de manière sécurisée plus tard. Cette étape peut nécessiter une confirmation de l'utilisateur sur les appareils.
3. Connexion: les deux appareils établissent un canal de communication pour une application spécifique (écouteurs, clavier, etc.)

La première étape est donc de détecter le téléphone. Cependant lorsqu'on fait `a`, il est très possible qu'il n'aparaisse pas. 

> Pour des raisons de sécurité, la plupart des téléphones d'aujourd'hui limitent sévèrement les connexions bluetooth entrantes. C'est pour cette raison qu'il ne sera peut-être pas détecté.

Ajoutez une ligne dans le fichier `devices.txt` pour l'appareil Android. Attention de bien mettre l'adresse MAC de votre téléphone:
```
DEVICE=Pixel TYPE=CLASSIC NODE=2 ADDRESS=74:74:46:CB:2B:56
```

Après avoir redémarré le programme, vous pourrez faire la commande `i` pour voir les informations:
```
node                        btlib version 14
1  Local (pi-o)   Mesh transmit off
      E4:5F:01:EC:63:4E
2  Pixel-6  Classic  Not connected
      74:74:46:CB:2B:56
```

#### Informations sur les services
Encore pour des raisons de sécurité, les téléphones n'acceptent pas des connexions bluetooth génériques: ils définissent des protocoles (aussi nommés "profils") pour des applications spécifiques.

On peut voir les services disponibles sur le téléphone avec `v`:
```
Connecting to Pixel-6 to read classic serial services...
Unexpected extra data - add time delays
Unexpected extra data - add time delays
Authentication/PIN fail
Trying again with no link key..
Passkey = 524055  Valid for 10 seconds
Unexpected extra data - add time delays
Unexpected extra data - add time delays
Pixel-6 has disconnected

Pixel-6 RFCOMM serial channels
  2  Headset Gateway
    UUID = 1112  Headset Audio Gateway
  3  Handsfree Gateway
    UUID = 111F  HandsfreeAudioGateway
  4  SMS/MMS
    UUID = 1132  Message Access Server
  5  OBEX Phonebook Access Server
    UUID = 112F  Phonebook Access PSE
  6  SIM Access
    UUID = 112D  SIM_Access
  7  OBEX Object Push
    UUID = 1105  OBEXObjectPush
```
Dans cet exemple, on retrouve des profils pour les connexions d'écouteurs, l'accès aux contacts ou à la carte SIM, etc. Cependant, le programme _btferret_ n'implémente pas les profils permettant d'utiliser ces services. 

#### Mode serveur
On configure le Pi en mode serveur afin que la connexion puisse être initiée par le téléphone en mode client.

Au menu de démarrage de _btferret_, faites la commande `s`. Ensuite il faut choisir le mode "Classique", l'identifiant du client puis le type de sécurité (0)
```
> s

  0 = node server
  1 = classic server
  2 = LE server
  3 = mesh server
Input server type 0/1/2/3  (x=cancel)
? 1

Input node of client that will connect

CLASSIC servers + NODE servers
 2 - Pixel-6
 0 - Any device
Input node  (x=cancel)
? 2

Client's security  (0,1,3 to pair or connect Android/Windows.. clients)
  0 = Use link key, print passkey here, remote may ask to confirm
  1 = No link key,  print passkey here (forces re-pair if pairing fails)
  2 = No keys  (connecting client is another mesh Pi)
  3 = Use link key, no passkey
  4 = Use link key, remote prints passkey, enter it here if asked

Client's security requirement  (x=cancel)
? 0

Server will listen on channel 1 and any of the following UUIDs
  Standard serial 2-byte 1101
  Standard serial 16-byte
  Custom serial set via register serial
Listening for Pixel-6 to connect (x=cancel)
```

Du côté du téléphone Android, il sera ensuite possible de se connecter avec l'application _Serial Bluetooth Terminal_ pour envoyer des messages au Pi.


#### Exercice 1
Compiler puis exécutez le programme `classic_server.c` et envoyez-lui des messages à partir de votre téléphone.

#### Exercice 2
Modifiez le programme serveur pour qu'il réponde "pong" lorsqu'il reçoit le message "ping" et qu'il se déconnecte pour n'importe quel autre message.
{{% expand "Solution" %}}
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "btlib.h"

int callback(int node, unsigned char *data, int len);

int main()
{
  int security, keyflag;

  if (init_blue("devices.txt") == 0)
    return (0);

  security = 3;

  keyflag = KEY_ON | PASSKEY_LOCAL;
  if (security == 1)
    keyflag = KEY_OFF | PASSKEY_LOCAL;
  else if (security == 2)
    keyflag = KEY_OFF | PASSKEY_OFF;
  else if (security == 3)
    keyflag = KEY_ON | PASSKEY_OFF;

  classic_server(ANY_DEVICE, callback, 10, keyflag);
  close_all();
}

int callback(int node, unsigned char *data, int len)
{
  static unsigned char *message = {"pong\n"};

  printf("   %s", data);

  if (strcmp(data, "ping\r\n") == 0)
  {
    write_node(node, message, strlen(message));
    printf("%s", message);
    return (SERVER_EXIT);
  }
  return (SERVER_CONTINUE);
}
```
{{% /expand %}}

#### Exercice 3
Connectez le module LED sur votre Pi. Modifiez ensuite le programme serveur pour que "1" allume la LED et "0" l'éteigne. 
{{% expand "Solution" %}}
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "btlib.h"
#include <pigpio.h>

#define LED 26

int callback(int node, unsigned char *data, int len);

int main()
{
    int security, keyflag;

    // Initialiser
    if (gpioInitialise() < 0)
    {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED, PI_OUTPUT);

    if (init_blue("devices.txt") == 0)
        return (0);

    security = 3;

    keyflag = KEY_ON | PASSKEY_LOCAL;
    if (security == 1)
        keyflag = KEY_OFF | PASSKEY_LOCAL;
    else if (security == 2)
        keyflag = KEY_OFF | PASSKEY_OFF;
    else if (security == 3)
        keyflag = KEY_ON | PASSKEY_OFF;

    classic_server(ANY_DEVICE, callback, 10, keyflag);
    close_all();
}

int callback(int node, unsigned char *data, int len)
{
    printf("   %s", data);

    switch (data[0])
    {
    case '1':
        gpioWrite(LED, 1);
        break;
    
    case '0':
        gpioWrite(LED, 0);
        break;

    default:
        break;
    } 
    
    return (SERVER_CONTINUE);
}
```
{{% /expand %}}

#### Exercice 4
Avec une autre équipe, adaptez les programmes`classic_client.c` et `classic_server.c` pour faire un programme de _chat_ simple: une fois connecté, le client et le serveur peuvent s'échanger des messages comme suit:

**Serveur**
```
Connected OK
Waiting for data from Raspberrypi        (x = stop server)
        hey
> salut
        boum
> pouet
Raspberrypi      has disconnected
```

**Client**
```
Connect OK
> hey
        salut
> boum
        pouet
> *
```
Lorsque le client envoit le caractère `*`, le serveur se déconnecte. 

La fonction `read_node_endchar()` ([référence](https://github.com/petzval/btferret?tab=readme-ov-file#4-2-37-read_node_endchar)) sera utile pour le programme client.

{{% expand "Solution" %}}
##### SERVEUR
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "btlib.h"

#define MAX_SIZE 100

int callback(int node, unsigned char *data, int len);

int main()
{
    int security, keyflag;

    if (init_blue("devices.txt") == 0)
        return (0);

    // Essayer 0,1,2,3 si problème
    security = 3;

    keyflag = KEY_ON | PASSKEY_LOCAL;
    if (security == 1)
        keyflag = KEY_OFF | PASSKEY_LOCAL;
    else if (security == 2)
        keyflag = KEY_OFF | PASSKEY_OFF;
    else if (security == 3)
        keyflag = KEY_ON | PASSKEY_OFF;

    classic_server(ANY_DEVICE, callback, 10, keyflag);
    close_all();
}

int callback(int node, unsigned char *data, int len)
{

    char *message;
    size_t length = 0;

    printf("        %s", data);
    if (data[0] == '*')
    {
        printf("Déconnexion\n");
        return (SERVER_EXIT);
    }
    printf("> ");
    getline(&message, &length, stdin);
    message[length] = '\n';
    write_node(node, message, strlen(message));
    free(message);
    
    return (SERVER_CONTINUE);
}
```
##### CLIENT
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "btlib.h"

#define NODE_ID 1
#define MAX_SIZE 100
#define END_CHAR 10 // \n

int main()
{
    int channel, ret;
    char *message;
    size_t len = 0;
    unsigned char buf[MAX_SIZE];

    if (init_blue("devices.txt") == 0)
        return (0);

    channel = find_channel(NODE_ID, UUID_16, strtohex("00001101-0000-1000-8000-00805F9B34FB", NULL));
    connect_node(NODE_ID, CHANNEL_NEW, channel);

    while (1)
    {
        printf("> ");
        getline(&message, &len, stdin);
        write_node(NODE_ID, message, strlen(message));
        free(message);

        ret = read_node_endchar(NODE_ID, buf, MAX_SIZE, END_CHAR, EXIT_TIMEOUT, 300000); 
        printf("        %s",buf);
    }

    disconnect_node(NODE_ID);
    close_all();
}
```
{{% /expand %}}

#### Exercice 5
Dans cet exercice, un message doit circuler à travers toutes les équipes: chaque équipe reçoit un message d'une autre équipe et doit immédiatement le retransmettre à une 3e équipe. Votre programme doit donc:
+ Démarrer en mode serveur
+ Recevoir un message
+ Arrêter le serveur  
+ Se connecter sur un autre serveur et envoyer le message.
