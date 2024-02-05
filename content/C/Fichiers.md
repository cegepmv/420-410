+++
title = 'Recettes'
weight = 24
date = 2024-01-21T19:49:14-05:00
draft = false
+++

Dans cette section on verra comment lire et écrire dans les fichiers, passer des arguments à un programme et gérer la mémoire avec _malloc()_, _realloc()_ et _free()_.

## Lecture et écriture dans des fichiers
[Lien vers w3school pour les fichiers en C, dont les notes sont très fortement inspirée](https://www.w3schools.com/c/c_files_read.php)
#### Écriture
###### Inclusion de la bibliothèque standard C :
Assurez-vous d'inclure la bibliothèque standard C en ajoutant la directive #include <stdio.h> au début de votre programme.

```c
#include <stdio.h>
```

###### Ouverture et/ou création d'un fichier :
Utilisez la fonction `fopen()` pour ouvrir un fichier. La fonction prend deux paramètres : le nom du fichier et le mode d'ouverture (lecture, écriture, etc.).

```C
FILE *fichier;
fichier = fopen("nom_du_fichier.txt", "w"); // "w" pour écriture, "r" pour lecture
```

***

Assurez-vous que _fichier_ n'est pas NULL, ce qui indiquerait une ouverture de fichier infructueuse.

```c
if (fichier == NULL) {
        printf("Erreur lors de l'ouverture du fichier.\n");
        return 1;
}
```
***

###### Écriture dans un fichier 
Utilisez la fonction `fprintf()` ou `fputc()` pour écrire dans le fichier.

```c
fprintf(fichier, "Bonjour, monde !\n");
// ou
fputc('A', fichier);
```

***

###### Fermeture du fichier
Assurez-vous de fermer le fichier après avoir terminé les opérations. Utilisez la fonction `fclose()`.

```c
fclose(fichier);
```
***
Voici un exemple complet :
```c
#include <stdio.h>

int main() {
    FILE *fichier;

    fichier = fopen("mon_fichier.txt", "w");

    if (fichier == NULL) {
        printf("Erreur lors de l'ouverture du fichier.\n");
        return 1;
    }

    fprintf(fichier, "Bonjour, monde !\n");

    fclose(fichier);

    return 0;
}
```
Cet exemple crée un fichier nommé "mon_fichier.txt" et y écrit la chaîne de caractères "Bonjour, monde !". Assurez-vous que votre programme a les autorisations nécessaires pour écrire dans le répertoire spécifié.

#### Lecture

###### fget() :
- Le premier paramètre de cette fonction permet de spécifier la variable qui stockera ce que retourne cette fonction.
- Le deuxième paramètre spécifie la taille maximale des données à lire. 
- Le troisième paramètre nécessite un pointeur de fichier qui est utilisé pour lire le fichier. Donc la cible de la lecture.

```c

FILE *fptr;

// Ouvrir en lecture
fptr = fopen("filename.txt", "r");

// Créer une variable pour le contenu
char myString[100];

// Mettre le contenu du fichier dans la variable
fgets(myString, 100, fptr);

// Afficher la variable
printf("%s", myString);

// Fermer
fclose(fptr);
```


*** 
###### Bonne pratique

C'est toujours une bonne idée de vérifier si le fichier à bien été ouvert. Ceci évite des comportements inattendus du code.


```c
FILE *fptr;

// Ouvrir en lecture
fptr = fopen("filename.txt", "r");

// Créer une variable pour le contenu
char myString[100];

// Si le fichier existe
if(fptr != NULL) {

  // Lire le fichier et afficher
  while(fgets(myString, 100, fptr)) {
    printf("%s", myString);
  }

// Fichier inexistant
} else {
  printf("Not able to open the file.");
}

// Fermer
fclose(fptr);
```


## Passer des arguments à un programme
Souvent il est utile de passer des arguments à un exécutable au moment où on l'appelle, par exemple le nom d'un fichier dans lequel on doit lire des données, etc. L'exemple suivant affiche "Bonjour" suivi de la chaîne de caractères passée au programme:
```c
#include <stdio.h>

int main(int argc, char *argv[]) {

    printf("Bonjour %s\n", argv[1]);

    return 0;
}
```
Pour passer des arguments à un programme il faut définir des arguments à la fonction _main()_. Le premier est un entier qui contient le nombre d'arguments passés. Le deuxième est le tableau qui les contient.

Le programme suivant fait la somme des deux nombres passés au programme:
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {

    int num1 = atoi(argv[1]);
    int num2 = atoi(argv[2]);
    int sum = num1 + num2;

    printf("%d\n",sum);

    return 0;
}
```
Dans ce programme, on remarque les choses suivantes:
+ `**argv` est une autre manière de représenter `*argv[]`, qui est un tableau de pointeurs
+ `atoi()` est une fonction qui convertit une chaîne de caractères en entier.

Il est recommandé de prévenir les erreurs en vérifiant le nombre d'arguments passé, comme suit:
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {

    if (argc != 3) {
        printf("Le programme a besoin de deux arguments!\n");
        return 1;
    }
    int num1 = atoi(argv[1]);
    int num2 = atoi(argv[2]);
    int sum = num1 + num2;

    printf("%d\n",sum);

    return 0;
}
```

## Gérer la mémoire
Jusqu'ici nous avons vu comment allouer la mémoire de manière statique, au moment de la compilation. Dans le programme suivant par exemple, on stocke dans un tableau de taille 10 les caractères entrés sur la ligne de commande:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char mot[10];
    char c;
    int i = 0;

    printf("Ecrivez un mot: ");

    // Lire chaque caractère jusqu'à avoir '\n'
    while ( (c = getchar()) != EOF && c != '\n') {
        mot[i] = c;
        i++;
    }

    // Afficher le contenu de la mémoire
    char *p = mot;
    for (int j=0;j<10;j++) {
        printf("Adresse: %p\n",p);
        printf("Valeur: %c\n\n",*p);
        p++;
    }
    printf("Nombre de caracteres: %d\n", i);

    return 0;
}
```

Le problème avec ce code est que la chaîne de caractères entrée par l'utilisateur ne peut pas avoir une taille supérieure à 10. 

La fonction `malloc()` permet de changer l'espace alloué à une variable pendant l'exécution du programme. Le code suivant demande à l'utilisateur la taille du mot avant de la définir:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char *mot;
    char c;
    int i;

    printf("Entrez le nombre de caracteres: ");
    scanf("%d",&i);
    mot = (char *)malloc(i * sizeof(char));

    printf("Ecrivez un mot: ");
    
    getchar(); // Ignorer le '\n' de l'input précédent
    i = 0; // Remettre à 0
    while ( (c = getchar()) != EOF && c != '\n') {
        mot[i] = c;
        i++;
    }

    // Afficher le contenu de a mémoire
    char *p = mot;
    for (int j=0;j<i;j++) {
        printf("Adresse: %p\n",p);
        printf("Valeur: %c\n\n",*p);
        p++;
    }
    free(mot);

    return 0;
}
```
La fonction `malloc()` prend la taille réelle qu'on veut allouer au pointeur `mot`, soit le nombre de caractères souhaité multiplié par la taille d'un caractère en mémoire. Elle retourne un pointeur au début de cet espace.

La fonction `free()` libère l'espace mémoire alloué au pointeur.

Pour changer la taille d'un espace préalablement défini avec `malloc()`, on peut utiliser `malloc()` une nouvelle fois mais il faudra alors copier le contenu de la mémoire dans le nouveau bloc alloué. Dans cette situation il est donc préférable d'utiliser `realloc()`.

Dans l'exemple suivant, on utilise `realloc()` pour ajouter incrémenter la taille d'un espace mémoire à mesure que l'utilisateur entre des caractères:
```c
int main() {
    char *mot;
    char c;
    int i;

    // Donner une taille de 1 caractère
    mot = (char *)malloc(sizeof(char));

    printf("Ecrivez un mot: ");
    
    while ( (c = getchar()) != EOF && c != '\n') {
        // Augmenter de 1 la taille de mot à chaque nouveau caractère
        mot = (char *)realloc(mot, (i + 1) * sizeof(char));
        mot[i] = c;
        i++;
    }

    char *p = mot;
    for (int j=0;j<i;j++) {
        printf("Adresse: %p\n",p);
        printf("Valeur: %c\n\n",*p);
        p++;
    }
    free(mot);

    return 0;
}
```
La fonction `realloc(*p,int)` prend le pointeur dont on veut changer la taille et la nouvelle taille qu'on veut lui donner, puis retourne un pointeur au début de cet espace mémoire.
