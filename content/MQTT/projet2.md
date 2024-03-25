+++
title = 'Projet 2'
date = 2024-03-20T13:15:44-04:00
draft = false
weight = 72
+++

#### Préalables
Connectez un senseur de luminosité à votre Pi.

Vous devez faire 2 programmes:
+ **p2_pub** (sur le Pi): Publie chaque 10 secondes dans la rubrique `p2` la valeur de luminosité lue sur le senseur;
+ **p2_sub** (sur le conteneur): À partir des données lues sur tous les Pi, affiche le nom du Pi d'où est lue la valeur maximale, et la moyenne de toutes les valeurs.

#### Spécifications:
+ Le message publié doit avoir le format `HOTE|VALEUR`. HOTE est le _hostname_ du Pi et VALEUR est le pourcentage de luminosité (un nombre entier entre 0 et 100).
+ Les informations doivent s'afficher comme suit sur la console de votre conteneur lorsque vous exécutez le programme:
```
equipe2
42
```
