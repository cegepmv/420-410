+++
title = 'Compilation'
weight = 21
date = 2024-01-21T19:36:31-05:00
draft = false
+++

## Compilation
Hello world compilé avec gcc sur la ligne de commande
- a.out, changé avec l'option -o

Option -v: afficher messages reltaifs à la compilation

Supposons qu'on a 2 versions du programme à compiler: uneversion debug avec tous les messages de compilation, une version sans messages avec l'exec nommé. Ça donne 2 commandes:
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

all: Ce que make fait si on ne spécifie pas de cible
```

## Fichiers de programmes
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
   saluerEn();
   return 0;
}
```

Compilation: `gcc hello.c salutations.c -o hello`

Ajouter les fichier d'entêtes