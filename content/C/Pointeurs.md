+++
title = 'Pointeurs'
weight = 23
date = 2024-01-21T19:49:58-05:00
draft = true
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

** EXERCICES
Valeur max de i
Quel type pour ...

Normalement lorsqu'on utilise une variable on veut seulement utiliser sa valeur:
    int a = 10;
    printf("%d\n",a);

Mais parfois c'est utile de connaitre son adresse. Dans ca cas, il faut la déclarer différemment.
    int a = 10;
    int *p = &a;  
    printf("%d\n",*p); //Affiche 10 (ce qu'il y a à l'adresse de 'a')
    printf("%p\n",p);  //Affiche 0x7fff206ab37c (l'adresse de 'a')
    printf("%p\n",&a);  //Affiche 0x7fff206ab37c (l'adresse de 'a')

&a désigne l'adresse de 'a' (pas sa valeur). On peut affecter cette adresse à une variable seulement si cette variable est préfixée de *.

On nommre Pointeurs les variables ainsi préfixées car elles contiennent une adresse (et donc pointent sur la valeur sans réellement la contenir)

** EXERCICES
Afficher l'adresse d'un long
Afficher l'adresse d'une variable non initialisée, int *p = &a; print p
Afficher l'adresse d'un pointeur non-init, int *p; print p (nil)
Afficher la valeur d'un pointeur non-init, int *p; print *p (segfault)
***

Changer la valeur d'un variable ne change pas son adresse:
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

On peut changer la valeur en passant par le pointeur (par déréférencement):
int main() {
    int a = 10;
    int *p = &a;
    printf("valeur %d\n",a);
    *p = 30;
    printf("valeur %d\n",a);
    return 0;
}

***EXERCICE
int main() {
    int a = 10;
    int b = a;
    return 0;
}
a et b ont-elles la meme adresse? Prouvez-votre reponse en affichant ces adresses avec des pointeurs.

Completer le code suivant pour multiplier la valeur de a par 10 sans utiliser 'a'
int main() {
    int a = 10;
    int *p = &a;
    *p = *p * 10; // A completer
    printf("valeur %d\n",a);
    return 0;
}
***

Incrementer un pointeur change l'adresse
int main() {
    char a;
    char *p = &a;
    for (int i=0;i<4;i++) {
        p++;
        printf("adresse %p\n",p);
    }
    return 0;
}

Si on change le type, les adresses sont incrementes de la taille du type de la variable

Mais attention: on ne peut pas mettre ce qu'on veut a une adresse simplement en dereferencant:
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
Parce que la nouvelle adresse n'a jamais ete initialisee comme int. Elle contient peut-etre autre chose.

On peut utiliser les pointeurs dans les listes
    int liste[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int *p = &liste[0];
   
    for (int i=0;i<10;i++) {
        printf("%d,",*p);
        p++;
    }

***EXERCICE
Modifier le programme precedent pour afficher les nombres 10, 8, 6, 4, 2 en utilisant uniquement le deplacement du pointeur

Affichez chaque lettre du mot 'bonjour' en deplacant un pointeur. 
int main() {
    char s[] = "bonjour";
    /* char *p1 = &s[0];
    for (int i=0;i<7;i++) {
        printf("%d\n", *p1);
        p1++;
    }*/
    return 0;
}
