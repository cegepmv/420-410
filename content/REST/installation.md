+++
title = 'Installation'
date = 2024-03-04T08:04:13-05:00
draft = false
weight = 81
+++
<!--
# AVEC FLASK
Pour serveur de dev:
- pip install flask
- Créer un répertoire pour l'app
- Créer un fichier app.py

Exercices: 
- 
Next: 
- Faire un serveur de prod
- Déployer 
- Implémenter app RGB sur serveur
- Faire une app avec 2 endpoints: 1 qui retourne la luminosité en JSON et 1 qui permet de set RGB. Aussi on affiche sur LCD le nombre de RGB allumés pour toute la classe (MQTT)
-------------------------------
-->
Pour contrôler à distance des modules connectés sur le Pi, une technique consiste à utiliser une API. À partir d'une page web, on pourra ainsi faire des appels d'API pour allumer une LED, activer des servos, lire les données d'un capteur, etc. On utilise cette méthode pour communiquer avec des objets connectés dans les cas où le contrôleur est assez puissant pour exécuter la pile de programmes requise. C'est le cas du Raspberry Pi.

Cette API doit donc s'exécuter sur le Pi. Pour cela nous utilisersons *Flask*, un cadriciel pour développer des applications en python côté serveur.

L'installation de base de Flask est très simple et rapide à déployer, mais n'utilise pas HTTPS et n'est pas conçue pour recevoir de nombreuses requêtes simultanément: c'est pourquoi on l'utilise dans des contextes de développement. En production, il est préférable d'utiliser un serveur WSGI; il en existe plusieurs, mais ici nous utiliserons un serveur web _apache_ avec le module `mod_wsgi`.

## Installation (développement)
Nous allons installer un serveur de développement Flask pour déployer une application de type "Hello World" où un des points terminaux retourne une simple chaîne de caractères.

La première étape est d'installer Flask:
```
pip install flask
```
Ensuite, créer un répertoire pour l'application:
```
mkdir app-flask
```
Puis finalement, dans ce répertoire, créez un fichier nommé `app.py` qui contient le code suivant:
```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/',methods=['GET'])
def bonjour():
    return "<h1>Bonjour le monde!</h1>"

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=3000)
```

Pour démarrer l'application, il suffit de lancer `python app.py` dans le répertoire où se trouve le fichier. Pour voir le message, ouvrez une page à l'adresse IP du Pi au port spécifié (ici, le port 3000).
