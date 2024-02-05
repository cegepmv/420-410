+++
title = 'Recettes'
weight = 24
date = 2024-01-21T19:49:14-05:00
draft = false
+++

Dans ce module nous verrons comment réaliser des opérations courantes (et utiles) en C:
+ Manipulation de fichiers
+ Passage d'arguments à un exécutable
+ Gestion de la mémoire

## Lire et écrire dans des fichiers
On utilise un *pointeur* de type FILE pour parcourir le contenu d'un fichier. Ce pointeur est retourné par la fonction `fopen()` qui permet de spécifier le nom du fichier et le mode d'accès.

```c
FILE *p fopen(const char nom, const char mode)
```
`mode` peut avoir les valeurs suivantes:
| Mode | Signification |
|:---|:---|
| `r` | Lecture |
| `w` | Écriture |
| `a` | Ajout |
| `r+` ou `a+` ou `w+` | Lecture / écriture |

##### Afficher ligne par ligne le contenu d'un fichier
```c
#include <stdio.h>

int main() {
    FILE *p;
    char ligne[50];

    // Ouvrir le fichier en mode lecture
    p = fopen("test.txt", "r");

    // Tant que la ligne lue n'est pas nulle
    while( fgets(ligne, sizeof(ligne), p) != NULL) {
        printf("%s",ligne);
    }    
    fclose(p);
    return 0;
}
```
Dans cet exemple, **fgets()** retourne NULL lorsqu'il atteint la fin du fichier. La variable `ligne` sert à stocker le contenu lu, `sizeof(ligne)` désigne la taille maximale lue et `p` est le pointeur de fichier.

##### Afficher chaque caractère d'un fichier


##### Écrire des lignes dans un fichier


##### Écrire dans un fichier et le lire ensuite

## Passer des arguments à un programme

## Gérer la mémoire

