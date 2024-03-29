+++
title = 'ExercicesCorriges'
date = 2024-02-15T00:39:13-05:00
draft = true
+++

#### Ex 2
```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    char* p;
    int compte = 0;
    char c;
    int taille;

    if (argc == 3) {
        c = *argv[2];   // Le premier caractère de argv[2]
        p = argv[1];    // Pointeur sur le premier caractère de argv[1]
        taille = strlen(argv[1]);
        for (int i=0;i<taille;i++) {
            if (*p == c) {
                compte++;
            }
            p++;
        }
    }
    printf("Le caractère apparaît %i fois\n",compte);
}
```

#### Ex 3
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    char* p;
    int moitie;
    int taille;
    char* mot1;
    char* mot2;
    char* pm1;
    char* pm2;

    if (argc == 2) {
        p = argv[1];    // Pointeur sur le premier caractère de argv[1]
        taille = strlen(argv[1]);
        moitie = taille / 2;
        mot1 = (char *)malloc(moitie += moitie % 2);
        mot2 = (char *)malloc(moitie);
        pm1 = mot1; // Garder un pointeur au début du mot
        pm2 = mot2; // Garder un pointeur au début du mot

        for (int i=0;i<moitie+1;i++) {
            *mot1 = *p;
            mot1++;
            p++;
        }
        for (int i=0;i<moitie;i++) {
            *mot2 = *p;
            mot2++;
            p++;
        }
    }
    printf("mot 1:%s\n mot 2:%s\n",pm1,pm2);

}
```

#### Ex 4
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    char* final;
    char* p;
    int tailleChaine = 0;   // Taille de la chaîne finale
    int tailleMot;

    // Taille de la chaîne finale
    for (int i=1;i<argc;i++) { // Init à 1 car arg[0] est le nom de l'exec
        tailleChaine += strlen(argv[i]);
    }
    final = (char*)malloc(tailleChaine);

    for (int i=1;i<argc;i++) {  // Pour chaque mot
        tailleMot = strlen(argv[i]);
        p = argv[i];
        for (int j=0;j<tailleMot;j++) {  // Pour chaque lettre on la copie dans 'final'
            *final = *p;
            final++;
            p++;
        }

    }
    final = final-tailleChaine;

    printf("Final: %s\n",final);
}
```

#### Ex 5
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#define TAILLE_BUF 200

int main() {
    FILE *f;
    char buffer[TAILLE_BUF]; // Taille arbitraire pour stocker chaque ligne
    int compte = 0;

    f = fopen("monFichier.txt", "r"); 
    if (f == NULL) {
        printf("Erreur !!\n");
        fclose(f);
        return 1;
    }

    while (fgets(buffer,TAILLE_BUF,f)) {
        if (strcmp(buffer,"\n") == 0) {
            compte++;
        }
    }
    printf("%i\n",compte);
    fclose(f);

}
```

#### Ex 6
```c
#include <stdio.h>

void changerVal(int* x, int* y) {
    int* temp;

    /* // Les printf() commentés permettent de voir les adresses
    // et les valeurs de chaque variable à chaque étape
    printf("x=%p\t&x=%p\t*x=%d\n",x,&x,*x);
    printf("y=%p\t&y=%p\t*y=%d\n",y,&y,*y); */

    temp = *x;
    /* printf("\ntemp = *x\n---------\n");
    printf("temp=%p\t\t&temp=%p\t*temp=???\n",temp,&temp);
    printf("x=%p\t&x=%p\t*x=%d\n",x,&x,*x);
    printf("y=%p\t&y=%p\t*y=%d\n",y,&y,*y); */
    
    *x = *y;
    /* printf("\n*x = *y;\n---------\n");
    printf("temp=%p\t\t&temp=%p\t*temp=???\n",temp,&temp);
    printf("x=%p\t&x=%p\t*x=%d\n",x,&x,*x);
    printf("y=%p\t&y=%p\t*y=%d\n",y,&y,*y); */

    *y = temp;
    /* printf("\n*y = temp;\n---------\n");
    printf("temp=%p\t\t&temp=%p\t*temp=???\n",temp,&temp);
    printf("x=%p\t&x=%p\t*x=%d\n",x,&x,*x);
    printf("y=%p\t&y=%p\t*y=%d\n",y,&y,*y); */
}

int main() {

    int a = 11;
    int b = 22;
    
    changerVal(&a,&b);
    printf("a=%d, b=%d\n",a,b); // Doit afficher "a=22, b=11"

}
```

#### Ex 7
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main() {
    int a = 64999;
    char *p;

    p = &a;
    for (int i=0;i<4;i++) {
        printf("p=%p; *p=%x\n",p,*p);
        p++;
    }
}
```

#### Ex 8
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void lire_mem(char* p, int n) {
    for (int i=0;i<n;i++) {
        printf("p=%p; *p=%x\n",p,*p);
        p++;
    }   
}

int main() {
    int v1 = 25; 
    float v2 = 34.908; 
    long int v3 = 908445;

    printf("%d\n",v1);
    lire_mem(&v1,sizeof(v1));
    printf("\n%f\n",v2);
    lire_mem(&v2,sizeof(v2));
    printf("\n%li\n",v3);
    lire_mem(&v3,sizeof(v3));
    
}
```

