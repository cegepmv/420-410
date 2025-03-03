+++
title = 'Exercice synthèse'
date = 2025-03-03T00:56:26-05:00
draft = false
weight = 42
+++

Vous devez faire un programme qui reçoit par UDP des messages pour contrôler une LED et un moteur _stepper_. 

Les messages se composent de 3 parties séparées par `:`
- Première partie: `LED` ou `STEP`. Désigne ce que le programme doit activer.
- Deuxième partie: 
  - Si `LED`: Nombre entier. Correspond au nombre de fois que la LED doit clignoter.
  - Si `STEP`: `H` ou `A`. Sens de rotation Horaire ou Antihoraire.
- Troisième partie: 
  - SI `LED`: Nombre entier. La vitesse de clignotement en nombre de fois par seconde.
  - Si `STEP`: Nombre entier [0-360]. L'angle de rotation.
  
#### Exemples de messages
- `LED:10:4`  -   La LED clignote 10 fois à vitesse de 4 fois par seconde
- `STEP:H:90` -   Le moteur fait une rotation de 90 degrés dans le sens horaire

Le programme doit pouvoir faire fonctionner le moteur et la LED en même temps, comme par exemple si le serveur reçoit `LED:60:1` et `STEP:A:720` immédiatement après. Vous devrez donc utiliser les _Threads_.