+++
title = 'Compilation'
weight = 21
date = 2024-01-21T19:36:31-05:00
draft = false
+++

La compilation d'un programme est la traduction du code source en code machine. Le code machine se compose de différentes instructions qui peuvent varier selon le CPU ou le système d'exploitation; c'est pourquoi le fichier exécutable d'un programme pour Windows ne pourra pas être exécuté sur un RaspberryPi (par exemple). 

Lorsqu'on compile un programme, le code source est analysé, optimisé et les instructions (qui peuvent se trouver dans différents fichiers) sont regroupées en un exécutable unique.

Le programme suivant affiche "Bonjour le monde" à l'écran:

_hello.c_
```c
#include <stdio.h>

int main() {
   printf("Bonjour le monde!\n");
}
```
La fonction `printf()` sert à afficher une chaîne de caractère.

L'instruction `#include` importe les définitions de fonctions de la librairie _stdio_ (contenues dans le fichier **stdio.h**), dont la fonction `printf()` fait partie.

Pour compiler ce programme, on utilisera le compilateur `gcc` (_GNU C Compiler_) cpmme suit:
```bash
gcc hello.c
```
L'exécutable créé par cette compilation sera sauvegardé dans un fichier nommé `a.out`. Pour changer le nom de ce fichier, on utilise l'option `-o`, comme dans l'exemple suivant où l'exécutable portera le nom `hello`:
```bash
gcc -o hello hello.c
```
{{% notice primary "Info" %}}
Les options du programme gcc sont nombreuses: `-o` spécifie le nom de l'exécutable, `-l` permet de lier le programme avec une librairie externe, `-v` affiche des messages, etc. La documentation officielle est ici: [https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html](https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html)
{{% /notice %}}



## _Makefile_
Supposons qu'on a 2 versions du programme à compiler: une version debug avec tous les messages de compilation, une version sans messages avec l'exec nommé. Ça donne 2 commandes:
```bash
gcc hello.c -v
gcc hello.c -o hello
```
Plutôt que de lancer chacune séparément, on peut avoir un Makefile avec deux cibles (release et debug)
```makefile
all: release

debug: hello.c
	gcc -v hello.c

release: hello.c
	gcc -o hello hello.c

clean:
	rm -f hello a.out

CIBLE: DÉPENDANCES
    COMMANDE
    COMMANDE
    ...
```
**all**: Ce que _make_ fait si on ne spécifie pas de cible.


## Fichiers de programme
Il est possible de séparer un programme dans plusieurs fichiers:

_salutation.c_
```c
#include <stdio.h>

void saluerFr() {
    printf("Bonjour le monde!\n");
}

void saluerEn() {
    printf("Hello World!\n");
}
```

_hello.c_
```c
#include <stdio.h>

void saluerFr();
void saluerEn();

int main() {
   saluerFr();
   saluerEn();
   return 0;
}
```

Les fonctions `saluerFr()` et `saluerEn()` ne sont pas définies dans le fichier **hello.c**, mais au moment de la compilation un lien sera fait avec le fichier qui les contient (**salutations.c**). Mais il faut quand même les déclarer dans **hello.c** pour pouvoir les utiliser: c'est ce que l'on fait aux lignes 3-4 du programme. 

Pour compiler le programme, il s'agit d'ajouter le nouveau fichier aux arguments de `gcc` lorsqu'on lance la compilation: 
```bash
gcc hello.c salutations.c -o hello`
```

Le _Makefile_ doit aussi être modifié en conséquence:

```Makefile
all: release

debug: hello.c
	gcc -v hello.c salutations.c

release: hello.c
	gcc -o hello hello.c salutations.c

clean:
	rm -f hello a.out
```

## Entêtes (_headers_)
Lorsqu'on construit un programme il est facile de le faire de façon désorganisée. Ceci rend le code et la structure du programme difficiles à comprendre, et donc difficiles à déboguer et améliorer.

Utiliser une nomenclature pour les fonctions et les variables, indenter le code clairement et insérer des commentaires sont des exmples de mesures qui rendent le code plus facile à maintenir. Une autre bonne pratique en C consiste à séparer les déclarations du code principal et les mettre dans des fichiers séparés avec l'extension **.h** (pour "headers").

Le programme _hello_ serait donc constitué des fichiers suivants:

_hello.c_: Ne contient plus les déclarations de fonctions; mais contient une instruction `#include` qui permet au compilateur de lier les déclarations des fonctions au programme principal.
```c
#include <stdio.h>
#include "salutations.h"

int main() {
   saluerFr();
   saluerEn();
   return 0;
}
```

_salutations.c_: Inchangé
```c
#include <stdio.h>

void saluerFr() {
    printf("Bonjour le monde!\n");
}

void saluerEn() {
    printf("Hello World!\n");
}
```

_salutations.h_: Déclarations de fonctions
```c
void saluerFr();
void saluerEn();
```

Le _Makefile_ n'a pas besoin d'être modifié.

