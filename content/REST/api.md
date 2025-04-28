+++
title = 'API'
date = 2024-03-04T18:45:29-05:00
draft = false
weight = 82
+++

Puisqu'on veut avoir une API, il faut définir ses points terminaux ("endpoints"). Il s'agit donc de créer les routes correspondantes et d'y ajouter le code nécessaire pour envoyer ou recevoir des données au format JSON. 

## /info
À titre d'exemple, nous allons créer un premier point terminal pour afficher des informations sur l'hôte, soit son nom et la date et l'heure du système. On souhaite que les données retournées aient le format suivant:
```json
{
    "hote": "raspberry",
    "date": "2025-03-03, 15:39:57"
}
```
On utilisera l'objet `Datetime` de python pour récupérer la date et `socket.gethostname()` pour le nom de l'hôte. Flask dispose aussi de la fonction `jsonify()` qui permet de créer des données au format JSON à partir d'un dictionnaire. Le fichier `app.py` doit donc contenir le code suivant:
```js
from flask import Flask, jsonify, request
import socket
from datetime import datetime

app = Flask(__name__)

@app.route('/info',methods=['GET'])
def info_hote():
  d = {}
  date = datetime.now()
  d['hote'] = socket.gethostname()
  d['date'] = date.strftime("%Y-%m-%d, %H:%M:%S")
  
  return jsonify(d)

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=3000)
```
Pour tester, ouvrez une page à cet _endpoint_; par exemple si votre Pi est à l'adresse 10.10.10.100 vous devrez ouvrir une page à `http://10.10.10.100:3000/info`.

## Lire une valeur du GPIO
Dans l'exemple suivant nous utilisons un bouton sur le GPIO 16 du Pi, puis le _endpoint_ `get_btn` pour lire l'état du bouton (1 ou 0). 
```python
from flask import Flask, jsonify, request
import pigpio

BTN = 16

app = Flask(__name__)
pi = pigpio.pi()
pi.set_mode(BTN,pigpio.INPUT)

@app.route('/bouton',methods=['GET'])
def get_bouton():
  d = {}
  d['etat'] = pi.read(BTN)
  
  return jsonify(d)

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=3000)
```

## Écrire une valeur sur le GPIO
Dans cet exemple on suppose qu'un module LED est connecté sur le port GPIO 20 du Pi. 

On ajoutera le point terminal `led` à notre API. Celui-ci recevra des informations au format JSON suivant:
```json
{ 
    etat: 1
}
``` 
L'attribut `etat` peut être 1 ou 0 selon qu'on veut allumer ou éteindre la LED.

On utilisera _pigpio_ pour accéder comme d'habitude au GPIO:
```python
from flask import Flask, jsonify, request
import pigpio

LED = 20

app = Flask(__name__)

pi = pigpio.pi()
pi.set_mode(LED,pigpio.OUTPUT)

@app.route('/led', methods=['POST'])
def set_led():
    if request.method == "POST":
      json = request.get_json()
      if "etat" in json:
        if json["etat"] == 1:
            pi.write(LED,1)
        elif json["etat"] == 0:
            pi.write(LED,0)
        else:
            return jsonify({'Erreur': 'Mauvaise valeur'}),500
      else:
        return jsonify({'Erreur': 'Mauvais attribut'}),500
    else:
      return jsonify({'Erreur': 'Requetes POST seulement'}),500
    return jsonify({'Etat': json["etat"]}),200
   
if __name__ == '__main__':
    app.run(host='0.0.0.0',port=3000)
```
## Test
Pour tester votre programme, il y a plusieurs possibilités.
1. Utiliser une extension Firefox comme "postman" ou "rested" qui permet d'envoyer des appels d'API
2. L'utilitaire _curl_ pour faire un appel directement d'une ligne de commande linux:
```
curl -X POST http://10.10.10.100:3000/led -H "Content-Type: application/json" -d '{"etat": 1}'
```
3. Une page HTML simple dont le code fait les appels d'API à tester, comme l'exemple suivant:
```html
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="container">
        <button id="onBtn" onclick="commande(1)">ON</button>
        <button id="offBtn" onclick="commande(0)">OFF</button>
        <div id="status"></div>
    </div>

    <script>
        async function commande(etat) {
            const statusDiv = document.getElementById('status');
            try {
                const response = await fetch('http://10.10.10.100:3000/led', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ etat: etat })
                });

                const data = await response.json();
                
                if (!response.ok) {
                    statusDiv.textContent = `Erreur: ${data.Erreur || 'Erreur inconnue'}`;
                    statusDiv.style.color = 'red';
                } else {
                    statusDiv.textContent = `LED est a ${etat}`;
                    statusDiv.style.color = 'green';
                }
            } catch (error) {
                statusDiv.textContent = `Erreur: ${error.message}`;
                statusDiv.style.color = 'red';
            }
        }
    </script>
</body>
</html>
```
{{% notice primary "Attention" %}}
Lorsque vous utilisez une page HTML pour tester, vous devrez tenir compte de *CORS*, un mécanisme de sécurité actif dans la plupart des navigateurs. Pour ce faire, la méthode la plus simple est d'utiliser `flask-cors`. Installez-le d'abord:
```
pip install flask-cors
```
puis ajoutez les lignes suivantes à votre progrmme:
```python
from flask_cors import CORS
(...)
app = Flask(__name__)
CORS(app)
(...)
```
{{% /notice %}}

## Exercice
Connectez la LED RGB et faites un endpoint nommé `rgb` qui change la couleur de la LED ou l'éteint et peut prendre 4 valeurs ("red","green","blue" et "off") pour un attribut nommé "etat".

Codez ensuite une page HTML en vous basant sur l'exemple précédent; cette page doit comprendre 4 boutons pour changer l'état de la LED.
