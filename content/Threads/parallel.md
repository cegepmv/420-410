+++
title = 'Processus en parallèle'
date = 2024-02-11T19:56:53-05:00
draft = false
weight = 41
+++

Dans les applications connectées, il est fréquent qu'un même programme doive gérer plusieurs processus simultanément. Cela peut poser des problèmes lorsqu'une tâche monopolise le processeur et que pendant ce temps, d'autres processus attendent leur tour. 

Dans un programme ordinaire, les instructions sont exécutées en séquence:
```python
pi = calculerPiALaMillioniemeDecimale()
print("SVP Patientez")
```

Dans cet exemple, le message "SVP Patientez" est assez inutile car l'instruction `print()` ne s'exécutera pas tant que la fonction de la ligne précédente ne sera pas terminée. 

Prenons un programme simple qui fait clignoter indéfiniment un module LED toutes les 250ms:
```python
import pigpio
import time

try:
    pi = pigpio.pi()
    pi.set_mode(16,pigpio.OUTPUT)
    
    while True:
        pi.write(16,1)
        time.sleep(0.25)
        pi.write(16,0)

except KeyboardInterrupt:
    print("Arrêt du programme")

finally:
    pi.write(16,0)
    pi.stop()
```


Comment peut-on modifier ce programme pour faire clignoter 2 modules LED, 1 chaque 250ms et l'autre chaque 500ms?
```python
import pigpio
import time

LED1 = 17
LED2 = 24

while True:
    # Initialiser
    pi = pigpio.pi()

    # Définir en mode output
    pi.set_mode(LED1,pigpio.OUTPUT)
    pi.set_mode(LED2,pigpio.OUTPUT)

    pi.write(LED1, 1)
    pi.write(LED2, 1)

    time.sleep(0.25)
    pi.write(LED1, 0)

    time.sleep(0.25)
    pi.write(LED1, 1)
    pi.write(LED2, 0)

    time.sleep(0.25)
    pi.write(LED1, 0)

    time.sleep(0.25)
except KeyboardInterrupt:
    print("Arrêt du programme")

finally:
    pi.write(LED1,0)
    pi.write(LED2,0)
    pi.stop()

```
<!-- 
```c
#include <stdio.h>
#include <pigpio.h>
#include <unistd.h>

int main() {
    // Faire clignoter LED 1 chaque 250ms et LED 2 chaque 500ms
    int LED1 = 17;
    int LED2 = 24;

    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Erreur d'initialisation pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(LED1, PI_OUTPUT);
    gpioSetMode(LED2, PI_OUTPUT);

    while (1) {
        gpioWrite(LED1, 1);
        gpioWrite(LED2, 1);

        usleep(250000);
        gpioWrite(LED1, 0);

        usleep(250000);
        gpioWrite(LED1, 1);
        gpioWrite(LED2, 0);

        usleep(250000);
        gpioWrite(LED1, 0);

        usleep(250000);
    }

    // Libérer les ressources
    gpioTerminate();

    return 0;
}
```
-->


On voit immédiatement que cette solution n'est pas idéale car la structure de notre programme dépend des données. Si en effet on veut changer les durées, par exemple 1 LED qui clignote chaque 300ms et l'autre chaque 700ms, il faudra revoir tout ce que contient la boucle `while`.

Intuitivement on voit qu'il serait préférable d'avoir deux boucles et que chacune définit sa propre durée. Le programme suivant est mieux construit:

```python
import pigpio
import time

LED1 = 17
LED2 = 24

def clignoter1():
    while True:
        pi.write(LED1, 1)
        time.sleep(0.3)
        pi.write(LED1, 0)
        time.sleep(0.3)

def clignoter2():
    while True:
        pi.write(LED2, 1)
        time.sleep(0.7)
        pi.write(LED2, 0)
        time.sleep(0.7)

try:
    # Initialiser
    pi = pigpio.pi()

    # Définir en mode output
    pi.set_mode(LED1,pigpio.OUTPUT)
    pi.set_mode(LED2,pigpio.OUTPUT)

    clignoter1()
    clignoter2()

except KeyboardInterrupt:
    print("Arrêt du programme")

finally:
    pi.write(LED1, 0)
    pi.write(LED2, 0)
    pi.stop()

```

Chacune des deux boucles `while` est définie dans sa propre fonction, et même si il y a plusieurs répétitions, le programme est mieux structuré. Par contre il ne donne pas le résultat escompté car les deux fonctions vont s'éxécuter une après l'autre.

## _Threads_
En programmation, il est possible de créer des nouveaux processus plus ou moins indépendants du flot normal d'exécution du programme principal. On nomme ces processus "threads". 

Lorsqu'on crée un thread, on lui associe une fonction. Cette fonction sera exécutée dans son propre processus et ainsi ne bloquera pas le processus principal. 

Dans l'exemple suivant, on crée un thread avec `threading.Thread()` puis on lance les fonctions "clignoter" dans leurs propres processus avec la fonction `.start()`:

```python
import pigpio
import time
import threading

LED1 = 17
LED2 = 24

def clignoter1():
    while True:
        pi.write(LED1, 1)
        time.sleep(0.25)
        pi.write(LED1, 0)
        time.sleep(0.25)

def clignoter2():
    while True:
        pi.write(LED2, 1)
        time.sleep(0.5)
        pi.write(LED2, 0)
        time.sleep(0.5)

try: 
    # Initialiser
    pi = pigpio.pi()

    # Définir en mode output
    pi.set_mode(LED1,pigpio.OUTPUT)
    pi.set_mode(LED2,pigpio.OUTPUT)

    # Déclaration des threads
    thread_led1 = threading.Thread(target=clignoter1, daemon=True)
    thread_led2 = threading.Thread(target=clignoter2, daemon=True)

    # Lancement des threads
    thread_led1.start()
    thread_led2.start()

    # Attendre que les threads se terminent
    # Dans ce programme les threads ne se terminent jamais
    thread_led1.join()
    print("Fin du thread 1")
    thread_led2.join()
    print("Fin du thread 2")

except KeyboardInterrupt:
    print("Fin du programme")

finally:
    pi.write(LED2, 0)
    pi.write(LED1, 0)
    pi.stop()




```
Les modifications apportées au code:
+ Ajouté le module `threading` qui permet de créer et manipuler des threads ;
+ Deux variables de type `threading.Thread()` (`thread_led1` et `thread_led2`) qui permet de déclarer/créer des threads ;
+ La fonction `.start()` commence les threads
+ La fonction `.join()` attend que les threads se terminent

##### `threading.Thread()`
Sert à déclarer un _thread_, le mettre dans une variable et l'associer à une fonction donnée. Cette fonction prend 2 paramètres:
+ `target` : La fonction que le processus doit appeler. Attention, il n'y a pas les parenthèses: on donne le nom seulement. 
+ `args` : Les arguments de la fonction (s'il y en a). Peut être un tuple ou une liste.
+ `daemon` : Spéficie si le thread créée est en mode "démon". Si cet argument est à `True`, le thread créée se terminera lorsque le thread parent (i.e. le programme principal) se termine.

##### `.start()`
Sert à lancer un _thread_.

##### `.join()`
Attend la fin de l'exécution du processus et revient au programme principal. Dans l'exemple, comme on a des boucles infinies on n'y arrivera jamais.


{{% notice primary "Défis" %}}
1. Modifier les fonctions pour que les LED clignotent 10 fois seulement.
2. Comment changer la boucle `while` des fonctions pour que le thread 2 termine avant le premier?
{{% /notice %}}

#### Passage de paramètres
On aurait un meilleur programme si on avait une seule fonction `clignoter()` à laquelle on passerait toutes les variables:

```python
# 'ms' est le nombre de millisecondes
# 'gpio' est le numéro de broche GPIO
def clignoter(ms, gpio) {
    rep = 10
    compteur = 0
    
    while counter <= 10:
        pi.write(gpio, 1)
        time.sleep(ms)
        pi.write(gpio, 0)
        compteur += 1
}
```

Il faut aussi modifier la partie du programme où on crée les threads:
```python

    thread_led1 = threading.Thread(target=clignoter, args=(0.25, LED1))
    thread_led2 = threading.Thread(target=clignoter, args=(0.50, LED2))

    thread_led1.start()
    thread_led2.start()
```

<!-- 

Si on compile ce programme on aura une erreur. Pourquoi?

La raison est la suivante: afin que la fonction passée à `pthread_create()` ne soit pas limitée dans les types d'arguments qu'elle peut avoir, le 4e paramètre (`&arg1` et `&arg2` dans l'exemple) sera passé à la fonction comme un pointeur de type `void *`. Puisque dans notre cas elle est définie avec un argument `int *`, on a une erreur.

La version qui fonctionne:
```c
void *clignoter(void *args) {
    int *valeurs = (int *)args;
    int ms = valeurs[0];
    int gpio = valeurs[1];

    for (int i=0;i<10;i++) {
        gpioWrite(gpio, 1);
        usleep(ms*1000);
        gpioWrite(gpio, 0);
        usleep(ms*1000);
    }
} 
``` 

On crée un tableau d'entiers, par exemple `arg1[]`, mais ce tableau "perd" en quelque sorte son type lorsqu'il est passé à la fonction `pthread_create()`. Il est ensuite passé à `clignoter()` comme un pointeur _void_, et on lui redonne le type `int *` pour accéder aux valeurs qu'il contient.

> `pthread_create()` travaille uniquement avec des **adresses**: celles de la fonction qu'elle doit appeler (c'est pourquoi `*clignoter()` est déclarée avec `*`) et celle d'un pointeur _void_ qui indique où se trouvent les arguments à passer à cette fonction. C'est à nous de s'assurer qu'on redonne ensuite le bon type à ce pointeur: si un pointeur a un type _void_, le compilateur ne sait pas quelle taille la variable occupe en mémoire et donc est incapable de lire sa valeur, de se déplacer à la prochaine valeur, etc.
-->

#### Exercice 1
Pour l'instant dans notre fonction les LED clignotent 10 fois. Modifier le programme pour que le nombre de clignotements soit le 3e argument passé à la fonction.


#### Exercice 2
Changez la fonction pour que les LED clignotent 10 fois à 100 ms d'intervalle, avec ces valeurs codées directement dans la fonction. La seule chose qu'il faut passer à `clignoter()` est donc un entier qui désigne le GPIO.


#### Exercice 3
<!-- 
On se base sur l'exercice du chapitre précédent:
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pigpio.h>

#define BUFFER_SIZE 10
#define LED_PIN 17
#define PORT 8888

// Modifier cette fonction pour qu'elle puisse être passée à pthread_create()
void clignoter(int gpio) {
    
} 

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
    gpioSetMode(LED_PIN, PI_OUTPUT);

    // Réception des messages  
    char input[BUFFER_SIZE];

    while (1) {
        int len,n;
        len = sizeof(adr_dist); 
        // Stocker le message reçu dans n
        n = recvfrom(socket_local, (char *)buffer, BUFFER_SIZE, MSG_WAITALL, (struct sockaddr *)&adr_dist, &len);
        buffer[n] = '\0';

        // Voir quel mot a été reçu
        if (strcmp(buffer, "allume\n") == 0) {
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
        else if(strcmp(buffer,"flash\n") == 0){
            // Créer ici le thread qui lance la fonction clignoter
        }
    }

    return 0;
}
```
-->


```python
import socket
import pigpio

LED_PIN = 17
PORT = 9090
BUFFER_SIZE = 1024

# Définissez cette fonction
def clignoter(gpio):
    pass

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

Ajoutez à ce programme une commande "flash" qui aura pour effet de faire clignoter la LED en créant un thread.
