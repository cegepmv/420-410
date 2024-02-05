+++
title = 'Recettes'
weight = 24
date = 2024-01-21T19:49:14-05:00
draft = false
+++


## Passer des arguments à un programme

## Gérer la mémoire

# Lecture et écriture dans des fichiers
[Lien vers w3school pour les fichiers en C, dont les notes sont très fortement inspirée](https://www.w3schools.com/c/c_files_read.php)
## Écriture
### Inclusion de la bibliothèque standard C :
Assurez-vous d'inclure la bibliothèque standard C en ajoutant la directive #include <stdio.h> au début de votre programme.

```c
#include <stdio.h>
```

### Ouverture et/ou création d'un fichier :
Utilisez la fonction fopen pour ouvrir un fichier. La fonction prend deux paramètres : le nom du fichier et le mode d'ouverture (lecture, écriture, etc.).

```C
FILE *fichier;
fichier = fopen("nom_du_fichier.txt", "w"); // "w" pour écriture, "r" pour lecture
```

***

Assurez-vous que fichier n'est pas NULL, ce qui indiquerait une ouverture de fichier infructueuse.

```c
if (fichier == NULL) {
        printf("Erreur lors de l'ouverture du fichier.\n");
        return 1;
    }
```
***

Écriture dans un fichier :
Utilisez la fonction fprintf ou fputc pour écrire dans le fichier.

```c
fprintf(fichier, "Bonjour, monde !\n");
// ou
fputc('A', fichier);
```

***

Fermeture du fichier :
Assurez-vous de fermer le fichier après avoir terminé les opérations. Utilisez la fonction fclose.

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

### Lecture

##### fget() :
- Le premier paramètre de cette fonction permet de spécifié la varibale qui stockera ce que retourne cette fonction.
- Le deuxième paramètre spécifie la taille maximale des données à lire. 
- Le troisième paramètre nécessite un pointeur de fichier qui est utilisé pour lire le fichier. Donc la cible de la lecture.

```c

FILE *fptr;

// Open a file in read mode
fptr = fopen("filename.txt", "r");

// Store the content of the file
char myString[100];

// Read the content and store it inside myString
fgets(myString, 100, fptr);

// Print the file content
printf("%s", myString);

// Close the file
fclose(fptr);
```


*** 
##### Bonne pratique

C'est toujours une bonne idée de vérifier si le fichier à bien été ouvert. Ceci évite des comportements inatendu du code.


```c
FILE *fptr;

// Open a file in read mode
fptr = fopen("filename.txt", "r");

// Store the content of the file
char myString[100];

// If the file exist
if(fptr != NULL) {

  // Read the content and print it
  while(fgets(myString, 100, fptr)) {
    printf("%s", myString);
  }

// If the file does not exist
} else {
  printf("Not able to open the file.");
}

// Close the file
fclose(fptr);
```
