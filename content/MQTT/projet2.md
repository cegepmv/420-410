+++
title = 'Projet2'
date = 2024-03-20T13:15:44-04:00
draft = true
weight = 72
+++

#### Préalables
Connectez un senseur de luminosité et l'écran LCD à votre Pi.

Vous devez faire 2 programmes:
+ **p2_pub**: Publie chaque 10 secondes dans la rubrique `p2` la valeur de luminosité lue sur le senseur;
+ **p2_sub**: À partir des données lues sur tous les Pi, affiche le nom du Pi d'où est lue la valeur maximale, et la moyenne de toutes les valeurs.

#### Spécifications:
+ Le message publié doit avoir le format `HOTE|VALEUR` (HOTE est le _hostname_ du Pi et VALEUR est la luminosité)
+ Le nom de l'agent et de la rubrique ne doivent pas être directement dans le code mais être écrits dans un fichier sur 2 lignes. La première ligne est le nom de l'agent et la deuxième est le nom de la rubrique. Le fichier doit se nommer `mqttcl.conf` et être dans le même répertoire que l'exécutable.
+ Les informations doivent être affichées comme suit sur l'écran LCD:
```
equipe 2
21093
```
