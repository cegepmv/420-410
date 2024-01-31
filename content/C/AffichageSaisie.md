+++
title = 'Affichage et saisie'
weight = 22
date = 2024-01-21T19:43:03-05:00
draft = false
+++


## Affichage

### `int printf()`
Cette fonction affiche des informations sur la _sortie standard_ (par défaut, l'écran). Avec une seule chaîne de caractères comme argument, elle l'affiche telle quelle:

```c
printf("Bonjour tout le monde");
```
La fonction retourne un entier qui représente le nombre de caractères affichés; une valeur négative signifie qu'une erreur est survenue lors de l'exécution.

Il est possible d'insérer des valeurs ou des variables dans la chaîne de caractères affichée et de spécifier leur format. Par exemple:
```c
printf("Le nombre %i est affiché\n",25);
```
Dans cet exemple le 2e paramètre de la fonction est la valeur littérale `25`. Le spécifieur `%i` signifie que cette valeur doit être interprétée puis affichée comme un nombre entier. On le met à l'endroit où la valeur doit être insérée dans la chaîne de caractères.

Il existe plusieurs types de spécifieurs selon le type de données à afficher. Par exemple, un nombre sera représenté en hexadécimal si on utilise `%x`, en nombre décimal avec `%f`, etc. Le tableau suivant énumère les principaux:

| SYMBOLE | TYPE | INTERPRÉTATION |
|:---|:---:|:---|
| %d ou %i | int | entier (signé) |
| %u | int | entier naturel (non-signé) |
| %o | int | entier exprimé en octal |
| %x | int | entier exprimé en hexadécimal |
| %c | int | caractère |
| %f | double | rationnel en notation décimale |
| %e | double | rationnel en notation scientifique |
| %s | char* | chaîne de caractères |

Cette valeur peut aussi être définie dans une variable. Dans l'exemple suivant on affiche la valeur d'une variable selon différents formats:
```c
int i = 256;
printf("Le nombre %i est affiché comme entier\n",i);
printf("Le nombre %x est affiché en hexadécimal\n",i);
printf("Le nombre %f est affiché en décimal\n",(float)i/3);
```
> Que se passe-t-il si on omet `(float)` au 3e appel de _printf()_ ?

## Saisie

### `int scanf()`
Cette fonction prend une chaîne de caractères saisie sur _l'entrée standard_ (par défaut, la ligne de commande) et stocke sons contenu dans une ou plusieurs variables.

La valeur retournée désigne le nombre de valeurs qui ont été traitées dans la chaîne saisie.

`scanf()` prend comme premier argument une chaîne de caractères composée des spécifieurs, puis des variables qui correspondent à ceux-ci. Par exemple, dans l'instruction suivante on lit un nombre entier et on le stocke dans la variable _nombre_:

```c
scanf("%d",&nombre);
```
Les variables dans lesquelles les valeurs seront stockées doivent être déclarées au préalable. Le code suivant affichera donc un nombre entré par l'utilisateur:
```c
int main() {
    int i;
    printf("Entrez un nombre: ");
    scanf("%d",&i);
    printf("Vous avez entré le nombre %d\n",i);
    return 0;
}
```

Pour saisir un caractère, il suffit de déclarer et ensuite d'afficher notre variable en conséquence:
```c
int main() {
    char i;
    printf("Entrez un caractère: ");
    scanf("%c",&i);
    printf("Vous avez entré le caractère %c\n",i);
    printf("La valeur numérique de %c est %d\n",i,i);
    return 0;
}
```

Remarquez que puisque _char_ est en réalité représenté comme un entier, il est possible de l'afficher comme tel.

Pour saisir des chaînes de caractères, la variable doit être une liste ("array") de _char_, et on doit lui définir une taille fixe.
```c
int main() {
    char chaine[10];
    printf("Entrez un mot: ");
    scanf("%s",chaine);
    printf("Vous avez entré le mot %s\n",chaine);
    return 0;
}
```
Attention, ici la variable _chaine_ dans la fonction _scanf()_ n'est pas précédée de `&`. 

------------------------------

# Exercices

## 1
Créez un programme nommé `cours2.c` qui permet d'afficher la phrase classique _Hello world_.

<!--
```c
    #include <stdio.h>
    int main() {
        printf("hello world\n");
    }
```
-->
## 2
Modifier le _Makefile_ suivant afin que la commande `make` (sans argument) compile ce fichier vers *cours2* et exécute le programme.

```makefile
all: 

cours2: cours2.c
	# Commande de compilation
	./cours2
```
<!--
```Make
all: cours2

cours2: cours2.c
	gcc cours2.c -o cours2
	./cours2
```
-->
## 3
Faites un programme pour prendre comme entrée le nom d'un utilisateur et vérifier si le nom entré est "Jean". Affichez "Bienvenue" si c'est le cas, sinon affichez "Allez-vous en".

_La fonction **strcmp()** vous sera utile ici..._

<!--
```c
#include <stdio.h>
#include <string.h>

int main() {
    char nom[10];
    printf("Entrez votre nom: ");
    scanf("%s",nom);
    if (strcmp(nom,"Jean") == 0) {
        printf(" Bienvenue\n");
    } else {
        printf(" Allez-vous en\n");
    }
    return 0;
}
```
-->


## 4
 
Voyons maintenant comment utiliser les PINs du RaspberryPi

```bash
    sudo apt update
    sudo apt upgrade
    sudo apt install pigpio

    sudo systemctl start pigpiod
    sudo systemctl enable pigpiod
```

Il faut compiler comme suit (En ajoutant le fichier librairie lpigpio) :

```
    gcc -o cours2 cours2.c -lpigpio
```

```c
#include <stdio.h>
#include <unistd.h>
#include <pigpio.h>

#define PIN_NUMBER 18

int main() {
    // Initialiser
    if (gpioInitialise() < 0) {
        fprintf(stderr, "Failed to initialize pigpio\n");
        return 1;
    }

    // Définir en mode output
    gpioSetMode(PIN_NUMBER, PI_OUTPUT);

    while (1) {
        // Allumer 1 s
        gpioWrite(PIN_NUMBER, 1);
        usleep(1000000);  

        // Éteindre 1 s
        gpioWrite(PIN_NUMBER, 0);
        usleep(1000000);  
    }

    // Terminate the pigpio library
    gpioTerminate();

    return 0;
}

```
_S'il y a des problèmes avec l'initialisation de pigpio, essayer:_

```bash
    sudo killall pigpiod
    sudo rm -f /var/run/pigpio.pid
```

Car il se peut qu'il y ait des processus qui trainent.

## 5

Faites un programme qui allume une LED lorsque vous appuyez sur un bouton (une LED et un bouton du kit KeyStudio).

## 6

Faites un programme qui allume ou éteint une LED selon ce qui est saisi à la ligne de commande:

```c
printf("#########################################\n");
printf("Choisissez l'une des options suivantes:\n");
printf("#########################################\n");
printf("   allume   -   Pour allumer la LED\n");
printf("   ferme    -   Pour fermer la LED\n");
printf("   quitter  -   Pour quitter\n");
```