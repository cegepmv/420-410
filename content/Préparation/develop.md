+++
title = 'Développement en C'
weight = 12
date = 2024-01-21T19:36:31-05:00
draft = true
+++

Dans cette section nous verrons les outils courants pour développer des programmes en C sur RaspberryPi.

## Compilation
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
La commande _gcc_ comprend de nombreuses options qui peuvent générer des exécutables différents, et parfois la compilation d'un programme se fait en plusieurs étapes où chacune de ses parties sera compilées séparément. 

Supposons par exemple qu'on a 2 versions du programme à compiler: une version de débogage avec tous les messages de compilation, et une version sans messages avec l'exécutable renommé. Il y aurait donc 2 commandes de compilation:
```bash
gcc hello.c -g -v
gcc hello.c -o hello
```
Plutôt que de lancer chacune séparément, on peut avoir un Makefile avec deux cibles (nommées par exemple _release_ et _debug_)
```makefile
all: release

debug: hello.c
	gcc -g -v hello.c

release: hello.c
	gcc -o hello hello.c

clean:
	rm -f hello a.out

CIBLE: DÉPENDANCES
    COMMANDE
    COMMANDE
    ...
```
Ainsi `make debug` compilera la version de débogage, `make release` compilera la version finale, etc.
> La cible **all** définit la cible par défaut si on lance _make_ sans arguments.

Il est recommandé de mettre chaque projet dans un répertoire différent afin que chacun d'eux puisse avoir son propre Makefile.

## Fichiers de programme
Afin de faciliter l'orgaisation des fichiers de code source et de favoriser leur réutilisation, il est utile de séparer un programme dans plusieurs fichiers:

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

Dans les exemples ci-dessus, les fonctions `saluerFr()` et `saluerEn()` ne sont pas définies dans le fichier **hello.c**; cependant au moment de la compilation un lien sera fait avec le fichier qui les contient (**salutations.c**). Mais il faut quand même les déclarer dans **hello.c** pour pouvoir les utiliser: c'est ce que l'on fait aux lignes 3-4 du programme. 

Pour compiler le programme, il s'agit d'ajouter le nouveau fichier, qui contient les deux fonctions, aux arguments de `gcc` lorsqu'on lance la compilation: 
```bash
gcc hello.c salutations.c -o hello
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

## Débogage
Un programme compilé contient des instructions en langage machine: ces instructions ne sont pas identifées de la même manière dans un fichier exécutable qu'elles le sont dans le code source - si votre code contient une variable nommée `luminosite`, on ne retrouvera pas cette chaîne de caractères dans l'exécutable. 

Lorsqu'on débogue un programme compilé, le débogueur doit être en mesure de faire le lien entre les éléments dans l'exécutable et leurs correspondants dans le code source. Il faut lui donner la liste des noms de variables, de fonctions, etc. qu'il doit associer au programme compilé. On nomme cette liste la "table des symboles", et on l'inclut au programme exécutable en ajoutant l'option `-g` lors de la compilation.

La première chose à faire pour déboguer des programmes en C est donc de les compiler en incluant les informations de débogage, comme suit:

```
gcc -g -o programme programme.c
```

#### VSCode
Pour déboguer un programme en C dans VSCode, il faut commencer par installer l'extension _C/C++_ au moment où VSCode est connecté sur le Pi:

![cext](/420-410/images/cext.png)

Sélectionnez ensuite le fichier source à déboguer puis cliquez sur le bouton `Run and Debug`. 

À cette étape il faut définir une configuration de débogage. Dans VSCode, elles sont définies dans des fichiers JSON. Vous pouvez en créer une vous-même mais il est plus simple d'en choisir une parmi les modèles prédéfinis: cliquez sur `Show all automatic debug configurations`. Ceci fait apparaître le menu des débogueurs disponibles.

Sélectionnez `C++ (GDB/LLDB)`, puis `Default Configuration`, et ensuite cliquez sur les paramètres de `(gdb) Launch` pour modifier sa configuration:

![gdbconf](/420-410/images/gdbconf.png)

![gdbconf2](/420-410/images/gdbconf2.png)

![gdbconf3](/420-410/images/gdbconf3.png)

Nous allons indiquer à VSCode que le programme à déboguer est nommé `a.out` dans le répertoire courant:

![launchjson](/420-410/images/launchjson.png)

Il faut donc s'assurer que la commande pour la version `debug` du programme soit conforme dans le Makefile (c'est-à-dire qu'elle ait l'option `-g` et qu'elle ne renomme pas le fichier compilé), comme suit:
```Makefile
debug: test.c
	gcc -g test.c
```

Lancez ensuite `make debug`, puis enfin vous pourrez lancer le débogage dans VSCode:

![debugdone](/420-410/images/debugdone.png)

{{% notice primary "pigpio" %}}
L'appel de la fonction `gpioInitialise()` nécessite les permissions **root**, donc un programme qui appelle cette fonction doit être précédé de _sudo_. Comme on ne peut pas spécifier à `launch.json` d'utiliser _sudo_, la solution consiste à se connecter sur le Pi avec VSCode en tant que **root**.

Pour ce faire, il faut modifier le fichier `/etc/ssh/sshd_config` pour qu'il contienne la ligne `PermitRootLogin yes`. Redémarrez ensuite le service ssh.
{{% /notice %}}
