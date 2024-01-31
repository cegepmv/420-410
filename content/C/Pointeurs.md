+++
title = 'Pointeurs'
weight = 23
date = 2024-01-21T19:49:58-05:00
draft = false
+++

## Variables et mémoire
Lorsqu'on déclare une variable dans un programme, elle est stockée à un endroit précis de la mémoire. Cet endroit a une adresse, notée en hexadécimal, et c'est l'adresse qui se trouve au début des données contenues dans la variable qui est utilisée. 

Dans l'exemple suivant, la variable nommée **mot** contient la chaîne de caractères "abcd" et est à l'adresse `0x77fe00`:

![abcd](/420-410/images/abcd.png)

L'espace occupé par une variable dans la mémoire dépend de ce qu'elle contient: dans l'exemple, plus la chaîne de caractères contenue dans **mot** est longue, plus l'espace occupé par la variable sera grand. Mais le _type_ d'une variable a aussi un effet sur l'espace qu'elle occupe.

#### Types

Dans le langage C, les variables peuvent avoir les types _char_, _int_, _long_, _float_ et _double_. Ces types permettent de stocker des valeurs numériques de tailles différentes car le nombre d'octets qu'ils utilisent pour stocker les variables sont différents. Le tableau suivant compare les types pour les nombres entiers:

| Type | Nombre d'octets | Valeurs possibles | 
|:---:|:---:|:---:|
| _char_ | 1 | 256 (2⁸) |
| _int_ | 4 | 4,3 milliards (2³²) | 
| _long_ | 8 | plein (2⁶⁴) |

Les types peuvent être _signés_, c'est-à-dire qu'ils acceptent les valeurs négatives, ou _non-signés_ (soit 0 ou une valeur positive). Dans les deux cas, le nombre de valeurs différentes possibles demeure le même: par exemple, une variable de type *int* non-signé peut avoir une valeur de 0 à 4 294 967 295; un *int* signé peut avoir une valeur de -2 147 483 648 à 2 147 483 647.

Par défaut, les variables sont toujours signées à moins d'ajouter le mot-clé _unsigned_ devant leur type au moment de leur déclaration.

La mémoire peut ainsi contenir des variables de divers types occupant des blocs de taille différentes:

![memblock](/420-410/images/memblock.png)

Dans cet exemple:
+ Une variable de type _long_ est à l'adresse `0x77fe00` et a la valeur 5655434;
+ Une variable de type _char_ est à l'adresse `0x77fe08` et a la valeur 119, ce qui correspond à "w" selon le standard ASCII;
+ Une variable de type _int_ est à l'adresse `0x77fe09` et a la valeur 23112.


#### Exercices
1. Quelle est la valeur numérique maximale qu'on peut donner à une variable de type _char_ signée?
2. Quelle est la valeur minimale qu'on peut donner à une variable _long_ non signée?
3. Quels types peut-on donner à une variable si on veut qu'elle contienne le nombre `0xb2d05e00`?

{{% expand "Réponses" %}}
1. 127
2. 0
3. unsigned int, long
{{% /expand %}}



## Pointeurs

Un pointeur est une variable dont le comportement est différent des variables ordinaires.

Normalement lorsqu'on utilise une variable, ce qui nous intéresse est sa valeur:
```c
int main() {
    int a = 10;
    printf("%d\n",a);
    printf("%d\n",a*3);
    for (int i=0;i<a;i++){
        // Ce code sera répété 'a' fois
    }
    return 0;
}
```
Dans cet exemple, on donne à la variable _a_ la valeur 10, et par la suite chaque fois qu'on réfère à _a_ dans le programme c'est en fait sa valeur (10) qui est utilisée.

Dans certains cas par contre, ce qui serait utile au programme serait de connaitre *l'adresse* d'une variable. Dans cette situation, il faut déclarer la variable différemment:
```c
int main() {
    int a = 10;
    int *p = &a;  
    printf("%d\n",*p); //Affiche 10 (ce qu'il y a à l'adresse de 'a')
    printf("%p\n",p);  //Affiche 0x7fff206ab37c (l'adresse de 'a')
    printf("%p\n",&a);  //Affiche 0x7fff206ab37c (l'adresse de 'a')
    return 0;
}
```

Dans cet exemple, `&a` désigne l'adresse de _a_ (pas sa valeur). Si on veut conserver cette adresse dans une autre variable, il faut que cette autre variable soit déclarée comme un _pointeur_. Pour ce faire, on la précède par `*` à sa déclaration; et attention, un pointeur doit avoir le même type que la variable qu'il pointe. 

> Un pointeur est simplement une variable dont la valeur est une adresse mémoire, ce qui lui permet de **pointer** sur une valeur sans vraiment la contenir.

Dans l'exemple, on utilise le pointeur de deux façons:
+ `p`, qui affcihe l'_adresse_ de la variable _a_;
+ `*p`, qui affiche la _valeur_ à l'adresse de la variable 'a'

L'exemple suivant utilise les pointeurs pour montrer que l'adresse d'une variable ne change pas lorsque sa valeur change:
```c
int main() {
    int a = 10;
    int *p = &a;
    printf("valeur %d\n",*p);
    printf("adresse %p\n",p);
    a = 30;
    printf("valeur %d\n",*p);
    printf("adresse %p\n",p);
    return 0;
}
```

#### Exercices
> Attention, pour afficher l'adresse d'un pointeur avec `printf()`on doit utiliser le spécifieur `%p`.

1. Faites un programme qui déclare la variable **n** de type _long_ dont la valeur est 1000, puis affichez l'adresse de la variable et sa valeur à l'aide d'un pointeur.
2. Faites un programme qui déclare (sans lui donner de valeur) une variable _int_ nommée **i**. Affichez ensuite son adresse et sa valeur à l'aide d'un pointeur. 
3. Faites un programme qui déclare (sans lui donner de valeur) un pointeur _int_ nommé **p**. Affichez ensuite son adresse et sa valeur. int *p; print p (nil)

<!--
{{% expand "Réponses" %}}
```c
// 1.
int main() {
    long n = 1000;
    long *p = &n;  
    printf("%p\n",p); 
    printf("%ld\n",*p); 
    return 0;
}
```
```c
// 2. 
int main() {
    int i;
    int *p = &i;
    printf("%p\n",p); 
    printf("%d\n",*p);  
    return 0;
}
```
```c
int main() {
    int *p;
    printf("%p\n",p); 
    printf("%d\n",*p);  
    return 0;
}
```
{{% /expand %}}
-->


## Déréférencement
Comme on l'a vu plus haut, le pointeur donne accès à deux choses:
+ L'adresse d'une information en mémoire
+ L'information elle-même

Comme un pointeur est à la base une façon de stocker des adresses, lorsqu'on affiche un pointeur on verra une adresse:
```c
int *pointeur = &a;
printf("valeur %p\n",pointeur);
```

En utilisant l'opérateur `*`, le pointeur va à l'adresse qu'il contient lire la valeur:
```c
int *pointeur = &a;
printf("valeur %p\n",*pointeur);
```

On appelle ce `*` "opérateur de déréférencement".

Puisqu'un pointeur déréférencé donne accès à la valeur, il est possible de changer celle-ci sans passer par la variable d'origine:
```c
int main() {
    int a = 10;
    int *p = &a;
    printf("valeur %d\n",a);
    *p = 30;
    printf("valeur %d\n",a);
    return 0;
}
```

#### Exercices
1. Dans le programme suivant:
```c
int main() {
    int a = 10;
    int b = a;
    return 0;
}
```
Les variables _a_ et _b_ ont-elles la meme adresse? Prouvez-votre réponse en affichant ces adresses avec des pointeurs.

2. Compléter le code suivant pour multiplier la valeur de _a_ par 10 sans utiliser la variable _a_ elle-même.
```c
int main() {
    int a = 10;
    int *p = &a;
    //Votre code ici
    printf("valeur %d\n",a);
    return 0;
}
```

<!--
{{% expand "Réponses" %}}
```c
// 1. 
int main() {
    int a = 10;
    int b = a;
	int *p1 = &a;
	int *p2 = &b;
    printf("valeur a: %d\n",*p1);
    printf("adresse a: %p\n",p1);
    printf("valeur b: %d\n",*p2);
    printf("adresse b: %p\n",p2);
    return 0;
}
```
```c
// 2.
int main() {
    int a = 10;
    int *p = &a;
    *p = *p * 10; 
    printf("valeur %d\n",a);
    return 0;
}
```
{{% /expand %}}
-->

## Déplacement des pointeurs
Les pointeurs sont souvent utilisés comme des marque-pages, c'est-à-dire qu'ils permettent de conserver à quelle position le programme se trouve dans une structure de données qu'il utilise.

Puisqu'un pointeur contient une adresse mémoire, changer sa valeur équivaut à le déplacer dans l'espace-mémoire. Par exemple, avec une variable de type _int_, l'opérateur `++` incrémente sa valeur de 1. Dans le cas d'un pointeur, le même opérateur déplace le pointeur de 1:
```c
int main() {
    char a;
    char *p = &a;
    for (int i=0;i<4;i++) {
        p++;
        printf("adresse %p\n",p);
    }
    return 0;
}
```
Comme le type _char_ a une taille de 1 octet, la boucle dans ce programme incrémente le pointeur de 1 octet à chaque itération; c'est pourquoi les valeurs affichées sont successives. Mais attention, **l'incrémentation dépend du type du pointeur**. En effet pour un pointeur de type _int_, le déplacement sera de 4 octets à la fois; pour _long_, de 8 octets.

> Lancez le programme en changeant le type de **a** et ***p** en _int_, vous verrez que les adresses affichées ont des intervalles de 4 octets.

Il faut être prudent lorsqu'on déplace un pointeur de cette manière: le langage C ne nous prévient pas lorsque le pointeur est dans une zone de mémoire non-initialisée. En effet, nous avons vu plus haut qu'on peut déréférencer un pointeur pour changer la valeur à l'adresse qu'il contient, mais ceci n'est possible que lorsque l'adresse contient effectivemnt une valeur.

Dans l'exemple suivant, on déréférence un pointeur pour changer la valeur à l'adresse qu'il contient, mais on voit que cela cause une erreur lorsque le pointeur ne pointe sur aucune valeur: 
```c
int main() {
    int a;
    int *p = &a;
    
    printf("valeur %d\n",*p);
    printf("adresse %p\n",p);
        
    *p = 433;
    printf("nouvelle valeur %d\n",*p);
    
    p++;
    printf("nouvelle adresse %p\n",p);
    
    *p = 261;
    printf("nouvelle valeur %d\n",*p); //Erreur
    
    return 0;
}
```
Ceci survient car la nouvelle adresse (où on essaie d'insérer `261`) n'a jamais été initialisée comme _int_. Elle contient peut-être des données de types différents (donc des blocs de mémoire de taille différente), ce qui cause un _erreur de segmentation_.

## Pointeurs et listes
On utilise souvent les pointeurs pour parcourir les tableaux. Dans l'exemple suivant, le tableau `liste` contient une séquence d'entiers, chacun occupant des blocs de mémoire contigus de 4 octets chacuns. Déplacer un pointeur dans cette liste permet donc d'accéder à chacun de ses éléments:
```c
int main() {
    int liste[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int *p = &liste[0];
   
    for (int i=0;i<10;i++) {
        printf("%d,",*p);
        p++;
    }
}
```
Dans ce programme, on déclare un tableau nommé _liste_. Ce tableau est de type _int_, ce qui signifie qu'il se compose d'une séquence de blocs mémoire de 4 octets (où chaque bloc est une donnée de type _int_).

On place ensuite le pointeur au début du tableau en lui donnant l'adresse du premier élément, `&liste[0]`. L'incrémentation du pointeur sur les blocs de mémoire permet ainsi de parcourir le tableau.

Si on déplace le pointeur à des adresses plus loin que le dernier _int_ de la séquence, par exemple avec une boucle `for (int i=0;i<100;i++)`, lire les valeurs à ces adresses ne provoquera pas l'erreur de _Segmentation fault_ vue plus haut mais les données ne sont pas significatives. Essayez!


#### Exercices
1. Modifier le programme de l'exemple précédent pour afficher les nombres 10, 8, 6, 4, 2 en utilisant uniquement le déplacement du pointeur.
2. Affichez chaque lettre du mot 'bonjour' en déplacant un pointeur. Rappelez-vous: un mot est en fait un tableau de caractères. 

<!--
{{% expand "Réponses" %}}
```c
// 1.
int main() {
    int liste[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int *p = &liste[9];
   
    for (int i=0;i<5;i++) {
        printf("%d,",*p);
        p=p-2;
    }
}
```
```c
// 2.
int main() {
    char s[] = "bonjour";
    char *p1 = &s[0];
    for (int i=0;i<7;i++) {
        printf("%c\n", *p1);
        p1++;
    }
    return 0;
}
```
{{% /expand %}}
-->


<!-- Pourquoi est-ce utile? -->


