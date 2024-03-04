+++
title = 'Installation'
date = 2024-03-04T08:04:13-05:00
draft = true
weight = 51
+++

Pour contrôler à distance des modules connectés sur le Pi, une technique consiste à utiliser une API. À partir d'une page web, on pourra ainsi faire des appels d'API pour allumer une LED, activer des servos, lire les données d'un capteur, etc.

Cette API doit donc s'exécuter sur le Pi. Pour cela nous utilisersons _Node_ et _Express_.

## Installation 
#### Node
Sur le Pi, en tant que **root**, installez _Node_ et le gestionnaire de paquets _npm_:
```
apt update
apt install node npm
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
Lancez ensuite l'application avec la commande `node index.js`. Vous pourrez y accéder en ouvrant une page web au port 3000 de votre Pi.

## API
Puisqu'on veut avoir une API, il faut définir ses points terminaux ("endpoints"). Il s'agit donc de créer les routes correspondantes et d'y ajouter le code nécessaire pour envoyer ou recevoir des données au format JSON. 

#### /info
À titre d'exemple, nous allons créer un premier point terminal pour afficher des informations sur l'hôte, soit son nom et la date et l'heure du système. On souhaite que les données retournées aient le format suivant:
```json
{
    "hostname": "raspberry",
    "date": "2024-03-03"
}
```
On utilisera l'objet `Date` de javascript pour récupérer la date et `os.hostname()` pour le nom de l'hôte. Attention, pour utiliser cette dernière fonction vous devez inclure le module "os" avec la commande `const os = require("os");` dans votre fichier _index.js_.

Ajoutez la route suivante à votre code:
```js
app.get('/info', (req, res) => {
  const info = {
    "hostname": os.hostname(),
    "date": new Date().toString()
  };
  res.send(info);
})
```
Pour tester, ouvrez une page à cet _endpoint_; par exemple si votre Pi est à l'adresse 10.10.10.100 vous devrez ouvrir une page à `http://10.10.10.100:3000/info`.

#### Lire une valeur digitale du GPIO
Pour accéder au GPIO à partir de javascript on peut utiliser le module _onoff_. Il faut tout d'abord l'installer avec la commande suivante:
```
npm install onoff
```

#### Écrire une valeur digitale du GPIO
Dans cet exemple on suppose qu'un module LED est connecté sur le port GPIO 17 du Pi. 

On ajoutera le endpoint `setLED` à notre API. Celui-ci recevra des informations au format JSON suivant:
```json
{ 
    valeur: 1
}
``` 
L'attribut `valeur` peut être 1 ou 0 selon qu'on veut allumer ou éteindre la LED.

On utilisera le module `onoff` installé précédemment pour accéder au GPIO. Dans votre fichier _index.js_, il faut inclure le module `onoff` et définir la broche qui enverra le signal avec les instructions suivantes:
```js
const Gpio = require('onoff').Gpio; 
const LED = new Gpio(17, 'out');
```

Le module `body-parser` sera utilisé pour récupérer les valeurs JSON. Installez-le tout d'abord avec la commande suivante:
```
npm install body-parser
```
Ensuite ajoutez cette instruction à votre fichier _index.js_:
```js
app.use(express.json());
```

Enfin, ajoutez le code suivant pour définir votre _endpoint_:
```js
app.post('/setLED', (req, res) => {
  const { valeur } = req.body;
  if (valeur !== 0 && valeur !== 1) {
    return res.status(400).json({ message: 'Erreur: valeur différente de 0 ou 1' });
  }
  LED.writeSync(valeur);
});
```

Pour tester votre programme, vous pouvez utiliser une extension Firefox comme "postman" ou "rested" qui permet d'envoyer des appels d'API, ou encore l'utilitaire _curl_ pour faire un appel directement d'une ligne de commande linux:
```
curl -X POST http://10.10.10.100:3000/setLED -H "Content-Type: application/json" -d '{"valeur": 1}'
```



