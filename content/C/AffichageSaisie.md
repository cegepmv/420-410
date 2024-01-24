+++
title = 'Affichage et saisie'
weight = 22
date = 2024-01-21T19:43:03-05:00
draft = true
+++


## Affichage

### `int printf()`
Cette fonction affiche des informations sur la _sortie standard_ (par défaut, l'écran). Avec une seule chaîne de caractères comme argument, elle l'affiche telle quelle:

```c
printf("Bonjour tout le monde");
```

Il est possible d'insérer des valeurs ou des variables dans la chaîne de caractères affichée et de spécifier leur format. Par exemple:
```c
printf("Le nombre %i est affiché\n",25);
```
Dans cet exemple le 2e paramètre de la fonction est la valeur littérale `25`. Le spécifieur `%i` signifie que cette valeur doit être interprétée puis affichée comme un nombre entier. On le met à l'endroit où la valeur doit être insérée dans la chaîne de caractères.

Il existe plusieurs types de spécifieurs selon le type de données à afficher. Par exemple, un nombre sera représenté en hexadécimal si on utilise `%x`, en nombre décimal avec `%f`, etc. Le tableau suivant énumère les principaux:

| SYMBOLE | TYPE | INTERPRÉTATION |
|:---|:---:|:---|
| %d ou %i | int | entier relatif |
| %u | int | entier naturel (unsigned) |
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

## `scanf()`
+ scanf
+ fscanf
+ sscanf
+ vsscanf
+ vfscanf