+++
title = 'Exercices'
date = 2026-01-22T17:44:01-05:00
weight = 13
+++

## Exercice 1 : Cycle de couleurs et gestion propre du programme

**Objectif :** Maîtriser les boucles temporelles et la capture d'interruptions système.

* **Consigne :** Développez un script permettant à une LED RGB de cycler indéfiniment entre trois états colorimétriques : **Rouge**, **Vert**, puis **Bleu**. Chaque couleur doit rester active pendant une durée d'une seconde.
* **Contrainte d'arrêt :** Le programme doit se terminer proprement lors d'une interruption clavier (`Ctrl+C`).
* **Exigence technique :** Aucun message d'erreur (type *KeyboardInterrupt*) ne doit apparaître dans la console à la fermeture. Assurez-vous également que la LED soit totalement **éteinte** avant la sortie définitive du script.

> **Indices :** Vous allez avoir besoin des librairies `time`, `pigpio` et d'utiliser une structure `try...except`.
> *Note : L'utilisation de l'intelligence artificielle est interdite. L'objectif est de forger votre propre logique de programmation.*

---

## Exercice 2 : Machine à états et gestion d'un bouton-poussoir

**Objectif :** Implémenter une machine à états simple et gérer le phénomène de rebond (ou l'état logique d'un bouton).

* **Consigne :** Créez un programme qui fait défiler les états de la LED RGB à chaque pression sur un bouton-poussoir. L'ordre de la séquence doit être le suivant :
1. Éteinte (état initial)
2. Rouge
3. Bleu
4. Vert
5. Retour à l'état "Éteinte", et ainsi de suite.


* **Contrainte de précision :** Le passage d'un état à un autre ne doit se produire qu'**une seule fois par pression**, peu importe la durée pendant laquelle le bouton reste enfoncé. Vous devez donc détecter le changement d'état (le "front") et non seulement le niveau logique haut ou bas.
> *Note : L'utilisation de l'intelligence artificielle est interdite. L'objectif est de forger votre propre logique de programmation.*

---
