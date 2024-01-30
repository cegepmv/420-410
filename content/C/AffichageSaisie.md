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

scanf() prend comme premier argument une chaîne de caractères composée desspécifieurs, puis des variables qui correspondent à ceux-ci. Par exemple, dans l'instruction suivante on lit un nombre entier et on le stocke dans la variable nombre:

```c
scanf("%d",&nombre);
```
Les variables dans lesquelles les valeurs seront stockée doivent être déclarées au préalable. Le code suivant affichera donc la somme de deux nombres entrés par l'utilisateur:
```c
int main() {
    int i;
    printf("Entrez un nombre: ");
    scanf("%d",&i);
    printf("Vous avez entré le nombre %d\n",i);
    return 0;
}
```
