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

Prenons un programme simple qui fait clignoter indéfiniment une LED toutes les 250ms:
```python
import pigpio
import time

try:
    pi = pigpio.pi()
    pi.set_mode(20,pigpio.OUTPUT)
    
    while True:
        pi.write(20,1)
        time.sleep(0.25)
        pi.write(20,0)
        time.sleep(0.25)

except KeyboardInterrupt:
    print("Arrêt du programme")
    pi.write(20,0)
    pi.stop()
```


Comment peut-on modifier ce programme pour faire clignoter 2 LEDs, 1 chaque 250ms et l'autre chaque 500ms?
```python
import pigpio
import time

LED1 = 20
LED2 = 21

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
    pi.write(LED1,0)
    pi.write(LED2,0)
    pi.stop()

```

On voit immédiatement que cette solution n'est pas idéale car la structure de notre programme dépend des données. Si en effet on veut changer les durées, par exemple 1 LED qui clignote chaque 300ms et l'autre chaque 700ms, il faudra revoir tout ce que contient la boucle `while`.

Intuitivement on voit qu'il serait préférable d'avoir deux boucles et que chacune définisse sa propre durée. Utiliser une fonction serait utile dans ce but. Le programme suivant est donc un peu mieux:

```python
import pigpio
import time

LED1 = 20
LED2 = 21

def clignoter1():
    # Clignote 20 fois avec un délai de 0.3 secondes
    n = 0
    limite = 20
    delai = 0.3
    while n < limite:
        pi.write(LED1, 1)
        time.sleep(delai)
        pi.write(LED1, 0)
        time.sleep(delai)
        n += 1

def clignoter2():
    # Clignote 10 fois avec un délai de 0.7 secondes
    n = 0
    limite = 10
    delai = 0.7
    while n < limite:
        pi.write(LED2, 1)
        time.sleep(delai)
        pi.write(LED2, 0)
        time.sleep(delai)
        n += 1

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
    pi.write(LED1, 0)
    pi.write(LED2, 0)
    pi.stop()
```

Chacune de ces deux fonctions gère sa propre vitesse et sa durée de clignotement. Le programme est mieux structuré, mais il ne donne pas le résultat escompté car les deux fonctions vont s'éxécuter une après l'autre.

## _Threads_
En programmation, il est possible de créer des nouveaux processus plus ou moins indépendants du flot normal d'exécution du programme principal. On nomme ces processus "threads". 

Lorsqu'on crée un _thread_, on lui associe une fonction. Cette fonction sera exécutée dans son propre processus et ainsi ne bloquera pas le processus principal. 

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
        time.sleep(0.7)
        pi.write(LED2, 0)
        time.sleep(0.7)

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
    thread_led1.join()
    print("Fin du thread 1")
    thread_led2.join()
    print("Fin du thread 2")

except KeyboardInterrupt:
    print("Fin du programme")
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
<!--
+ `daemon` : Spéficie si le thread créée est en mode "démon". Si cet argument est à `True`, le thread créée se terminera lorsque le thread parent (i.e. le programme principal) se termine.
-->

##### `.start()`
Sert à lancer un _thread_.

##### `.join()`
Attend la fin de l'exécution du processus et revient au programme principal. Si `join()` n'est pas appelée, cette attente aura lieu à la fin de l'exécution du programme.

#### Passage de paramètres
On aurait un meilleur programme si on avait une seule fonction `clignoter()` à laquelle on passerait toutes les variables, par exemple:

```python
def clignoter(gpio,delai,duree):
    # La durée divisée par 2 fois le delai donne le nombre
    # d'itérations de la boucle. On multiplie par 2 car time.sleep() 
    # est appelée 2 fois.
    # ex: 10 secondes avec un délai de 1 secondes = 5 itérations
    n = 0
    while n < duree / (2 * delai):
        pi.write(gpio, 1)
        time.sleep(delai)
        pi.write(gpio, 0)
        time.sleep(delai)
        n += 1
```

Il faut donc modifier la partie du programme où on crée les threads pour passer les paramètres à la variable `args`:
```python

    thread_led1 = threading.Thread(target=clignoter, args=(0.3, LED1))
    thread_led2 = threading.Thread(target=clignoter, args=(0.7, LED2))

    thread_led1.start()
    thread_led2.start()
```


## Exercice
Connectez votre RaspberryPi à un bouton et une LED. 

Faites un programme qui, chaque 10 secondes, fait clignoter en une seconde la LED le nombre de fois que le bouton a été appuyé durant les 10 secondes précédentes. Par exemple, si durant 10 secondes le bouton a été appuyé 5 fois, alors la LED clignote 5 fois en une seconde.

Vous devez utiliser deux fonctions:
- `compter_clics()`: Compte le nombre de clics sur le bouton
- `clignoter()`: Clignote _n_ fois durant 1 seconde (n = nombre de clics)
  
Vous pouvez utiliser une variable globale pour stocker le nombre de clics. Il s'agit de la re-déclarer dans chaque fonction qui l'utilise avec le mot-clé `global`, par exemple:
```python

# Initialisation de la variable
maVariable = 0

def uneFonction():
    global maVariable
    ...

def uneAutreFonction():
    global maVariable
    ...

```


