+++
title = 'Exercices'
date = 2024-02-14T18:26:55-05:00
weight = 15
draft = false
+++


#### Exercice 1
Soit les variables suivantes:
```c
int liste[] = {23,66,1,21} // supposez que '23' est à l'adresse 0x77770210
int* p = liste;
```
Quelles sont les valeurs des expressions suivantes?
1. liste[2]+3
2. *p+liste[0]
3. p+3
4. *p+3
5. *(p+3)

#### Exercice 2
Faites un programme qui prend deux arguments, un mot et un caractère, et affiche le nombre de fois que le caractère apparaît dans la chaîne. Un exemple d'appel du programme:
```bash
pi@raspberry:~$ ./exo2 bonjour o
2
```

#### Exercice 3
Faites un programme qui prend un mot en argument, met la première moitié dans un tableau de `char` et la deuxième moitié dans un autre tableau de `char`. Utilisez `malloc()` pour définir leurs tailles. Par exemple, le mot `bonjour` sera retranscrit comme `bonj` et `our`.

#### Exercice 4
Faites un programme qui prend un nombre arbitraire d'arguments et les concatène dans un tableau de `char`. Par exemple:
```bash
pi@raspberry:~$ ./exo4 abc xyz qwerty
```
créera un tableau qui contient la chaîne `abcxyzqwerty`.

#### Exercice 5
Faites un programme qui lit un fichier et affiche le nombre de lignes vides qu'il contient.

#### Exercice 6
Modifiez la fonction suivante afin d'échanger les valeurs des deux variables
en utilisant des pointeurs. 
```c
void changerVal(int x, int y) {
    int temp;

    temp = x;
    x = y;
    y = temp;
}
```

#### Exercice 7
Déclarer un nombre entier **int** et utiliser un pointeur de type `char` pour afficher la valeur en hexadécimal de chacun des octets (`int` est stocké sur 4 octets).

#### Exercice 8
Faire la fonction `lire_mem(char*, int)` à partir du code précédent. Le premier paramètre est un pointeur sur `char` et le deuxième est le nombre d'itérations de la boucle. Utilisez ensuite cette fonction pour montrer le contenu de trois variables: 
```c
int v1 = 25; 
float v2 = 34.908; 
long int v3 = 908445
```

