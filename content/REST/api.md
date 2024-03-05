+++
title = 'API'
date = 2024-03-04T18:45:29-05:00
draft = false
weight = 62
+++

Puisqu'on veut avoir une API, il faut définir ses points terminaux ("endpoints"). Il s'agit donc de créer les routes correspondantes et d'y ajouter le code nécessaire pour envoyer ou recevoir des données au format JSON. 

## /info
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

## Lire une valeur digitale du GPIO
Dans cet exemple nous utilisons un bouton sur le GPIO 18 du Pi, puis le _endpoint_ `getBTN` pour lire l'état du bouton (1 ou 0). 

Pour accéder au GPIO à partir de javascript on peut utiliser le module _onoff_. Il faut tout d'abord l'installer avec la commande suivante:
```
npm install onoff
```
Ensuite on doit:
+ Importer le module à l'aide de la fonction _require()_;
+ Déclarer le numéro de GPIO sur lequel on veut lire les données;
+ Utiliser la méthode _readSync()_ pour lire la valeur;

Votre programme devrait donc ressembler à ceci:
```js
const express = require('express');
const Gpio = require('onoff');  // Importer module onoff
const BTN = new Gpio(18, 'in'); // GPIO 18 en mode lecture
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Bonjour le monde!')
})

// Retourne le JSON { state: 1 } ou { state: 0 }
app.get('/getBTN', (req, res) => {
  const bouton = {
    "state": BTN.readSync() // Lire la valeur 
  };
  res.send(bouton);
})

app.listen(port, () => {
  console.log(`Application roule sur port ${port}`)
})
```

## Écrire une valeur digitale vers le GPIO
Dans cet exemple on suppose qu'un module LED est connecté sur le port GPIO 17 du Pi. 

On ajoutera le endpoint `setLED` à notre API. Celui-ci recevra des informations au format JSON suivant:
```json
{ 
    state: 1
}
``` 
L'attribut `state` peut être 1 ou 0 selon qu'on veut allumer ou éteindre la LED.

On utilisera le module `onoff` installé précédemment pour accéder au GPIO. Dans votre fichier _index.js_, il faut inclure le module `onoff` et définir la broche qui enverra le signal avec les instructions suivantes:
```js
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
  const { state } = req.body;
  if (state !== 0 && state !== 1) {
    return res.status(400).json({ message: 'Erreur: valeur différente de 0 ou 1' });
  }
  LED.writeSync(state);
});
```
#### Test
Pour tester votre programme, il y a plusieurs possibilités.
1. Utiliser une extension Firefox comme "postman" ou "rested" qui permet d'envoyer des appels d'API
2. L'utilitaire _curl_ pour faire un appel directement d'une ligne de commande linux:
```
curl -X POST http://10.10.10.100:3000/setLED -H "Content-Type: application/json" -d '{"valeur": 1}'
```
3. Une page HTML simple dont le code fait les appels d'API à tester, comme l'exemple suivant:
```html
<!DOCTYPE html>
<html>
<head>
<title>Test API</title>
</head>
<body>
  <button id="bouton">LED</button>

  <script>
    const button = document.getElementById("bouton");
    button.addEventListener("click", () => {
      fetch("http://10.10.10.100:3000/setLED", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ valeur: 1 })
      })
      .then(response => {
        if (!response.ok) {
          throw new Error("Erreur de la requête");
        }
        console.log("LED allumée");
      })
      .catch(error => {
        console.error("Erreur:", error);
      });
    });
  </script>
</body>
</html>
```

## Exercice
Connectez la LED RGB et faites 2 endpoints: `setRGB`, qui change la couleur de la LED ou l'éteint et peut prendre 4 valeurs ("red","green","blue" et "off") pour un attribut nommé "state", et `getRGB` qui retourne des données au même format JSON selon l'état de la LED.

Codez ensuite une page HTML en vous basant sur l'exemple précédent; cette page doit comprendre 4 boutons pour changer l'état de la LED et un bouton pour afficher sur la page la couleur courante de la LED.
