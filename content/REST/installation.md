+++
title = 'Installation'
date = 2024-03-04T08:04:13-05:00
draft = false
weight = 61
+++

Pour contrôler à distance des modules connectés sur le Pi, une technique consiste à utiliser une API. À partir d'une page web, on pourra ainsi faire des appels d'API pour allumer une LED, activer des servos, lire les données d'un capteur, etc.

Cette API doit donc s'exécuter sur le Pi. Pour cela nous utilisersons _Node_ et _Express_.

## Installation 
#### Node
Sur le Pi, en tant que **root**, installez _Node_ et le gestionnaire de paquets _npm_:
```
apt update
apt install nodejs npm
```
Vérifiez que l'installation est correcte avec les commandes `node -v` et `npm -v`. Elles devraient respectivement afficher `v12.22.12` et `7.5.2`.

#### Express
Nous allons créer un répertoire nommé **rest** pour les fichiers de notre API et y installer le _framework_ Express. Les commandes sont les suivantes:
```
mkdir rest
cd rest
npm init 
npm install express 
```
{{% notice primary "npm init" %}}
La commande `npm init` a pour but de créer le fichier _package.json_; celui-ci contient différentes informations sur votre application. Pour ce faire, `npm init` vous pose quelques questions. Ici, vous pouvez choisir les réponses par défaut en faisant `enter` pour chacune.
{{% /notice %}}

#### Test
Le serveur que nous avons installé est structuré de la manière la plus simple possible: nous avons simplement défini un point d'entrée, qui correspond à l'attribut "main" dans le fichier _package.json_ (`index.js` par défaut). Il est donc possible de mettre tout le code de notre API dans cet unique fichier.

On définit donc une première route pour tester notre installation. Celle-ci affiche "Bonjour le monde!" à la racine du site. Créez le fichier `index.js` et mettez-y le contenu suivant:
```js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Bonjour le monde!')
})

app.listen(port, () => {
  console.log(`Application roule sur port ${port}`)
})
```
Lancez ensuite l'application avec la commande `nodejs index.js`. Vous pourrez y accéder en ouvrant une page web au port 3000 de votre Pi.

