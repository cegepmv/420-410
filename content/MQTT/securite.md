+++
title = 'Sécurité'
date = 2024-04-07T15:56:18-04:00
draft = false
+++

Pour sécuriser les communications en MQTT, on utilise les mêmes méthodes que pour n'importe quel autre protocole, c'est-à-dire: l'authentification et le chiffrement. 

Ces deux méthodes sont appliquées indépendamment l'une de l'autre: en effet, on peut avoir des communications chiffrées sans authentification, ou au contraire authentifiées mais sans chiffrement. Évidemment, il est recommandé d'utiliser les deux. 

Dans cette section, nous allons décrire le fonctionnement de chacune de ces méthodes puis nous expliquerons comment les appliquer aux clients et aux agents MQTT.

## Authentification

C'est dans la configuration de l'agent MQTT que l'authentification doit être activée.

Lorsqu'elle l'est, les clients MQTT devront fournir un nom d'utilisateur et un mot de passe chaque fois qu'ils établissent une connexion auprès de l'agent, donc:
+ Un client doit s'authentifier chaque fois qu'il publie sur une rubrique;
+ Un client doit s'authentifier une seule fois lorsqu'il s'abonne à une rubrique.

Un des avantages de l'authentification est qu'elle permet d'utiliser des _listes de contrôle_ d'accès pour limiter les rubriques sur lesquelles les clients peuvent publier ou s'abonner.

Pour activer l'authentification sur le serveur, il faut mettre `allow_anonymous` à **false**, ou encore commenter `allow_anonymous true` dans le fichier de configuration. Aussi, il faut définir dans la directive `password_file` le chemin du fichier qui contient les identifiants et mots de passe des utilisateurs:
```
# Fichier /etc/mosquitto/mosquitto.conf

#allow_anonymous true
password_file /etc/mosquitto/passwd
```

#### Fichier des utilisateurs
Pour ajouter un utilisateur la commande est `mosquitto_passwd`. La commande doit être suivie du nom du fichier des utilisateurs puis du nom de l'utilisateur que vous souhaitez créer; vous devrez ensuite entrer le mot de passe manuellement: 
```
root@broker:~# mosquitto_passwd /etc/mosquitto/users sam
Password: 
Reenter password: 
```
> Si le fichier n'existe pas, utilisez l'option **-c** (comme "create") pour le créer.

Le fichier contient les hachages de mots de passe:
```
root@broker:~# cat /etc/mosquitto/users 
sam:$7$101$Fx8sWVNUkHTOL0eY$s66bwEUeQhM5atnmd19DEKDExYnLhVQYaWQNPBWkjGqgEOOABdWdMSasLLOy2HODeF0JLpnwf58cteIPkKe5hg==
```

Malgré cela, il est recommandé que le fichier des utilisateurs soit accessible uniquement à l'utilisateur **mosquitto**:
```
root@broker:~# ls -l /etc/mosquitto/users 
-rw------- 1 mosquitto mosquitto 472 Apr  5 18:00 /etc/mosquitto/users
```

#### Authentification sur la ligne de commande
Pour les utilitaires `mosquitto_pub` et `mosquitto_sub`, il faut passer les identifiants et mots de passe avec les options **-u** et **-P** (respectivement):
```
root@pi:~# mosquitto_sub -h 10.10.22.154 -t "meteo" -u sam -P abc-123
```

#### Authentification dans un programme
Dans un programme en C utilisant `libmosquitto-dev`, la fonction [mosquitto_username_pw_set()](https://mosquitto.org/api/files/mosquitto-h.html#mosquitto_username_pw_set) permet de spécifier les identifiants de connexion. Elle doit être appelée avant la connexion à l'agent: 
```
mosquitto_username_pw_set(mosq,"sam","abc-123");
```
Dans l'exemple précédent, l'argument `mosq` correspond à l'identifiant retourné par la fonction `mosquitto_new()`.

## Chiffrement
La méthode éprouvée pour assurer la confidentialité des communications à la couche TCP est d'utiliser des certificats avec le protocole TLS ("Transport Layer Security").

Les certificats sont échangés chaque fois qu'un client et un serveur souhaitent établir une connexion TCP sur lequelle tous les messages seront chiffrés. Ces certificats contiennent des informations permettant de générer une  **clé de session**, qui est une clé _symétrique_ utilisée pour chiffrer et déchiffrer les messages. Elle est détruite dès que la connexion entre les deux hôtes est fermée.

Mais avant d'envoyer des informations confidentielles chiffrées à un hôte sur le réseau, il faut être certain que cet hôte est digne de confiance. Les certificats servent donc aussi à garantir que l'hôte qui le présente est approuvé par un tiers, et donc qu'on peut lui faire confiance.

On nomme ces tiers les _autorités de certification_ (ou CA pour "certification authority"). 

#### Fonctionnement des certificats numériques
Imaginez 2 hôtes: **A** et **B**. Les deux veulent s'envoyer des messages chiffrés, mais aussi ils veulent être certains de l'identité de l'autre. Il ont donc besoin qu'une entité en qui les deux ont confiance, **CA**, atteste que A et B sont bien qui ils prétendent.

Chacun des 3 va donc générer une paire de clés (une clé publique et une clé privée). CA distribue aux 2 autres sa clé _publique_ et cache sa clé _privée_.

CA crée son propre certificat et utilise sa clé privée pour le signer. A et B utilisent la clé publique de CA pour vérifier cette signature: si la clé publique qu'ils ont permet de déchiffrer correctement cette signature, alors cela signifie qu'elle a été créée avec la clé privée de CA, et comme CA a bien caché sa clé privée, on a l'assurance que le certificat a bien été signé par CA. À cette étape, A et B sont certains de l'authenticité du certificat de CA.

Ensuite, A demande à CA d'attester son identité. Pour ce faire, il crée son propre certificat, y ajoute sa clé publique et demande à CA de le signer. CA utilise sa propre clé privée pour signer le certificat de A. A est maintenant en possession d'un certificat qui contient:
+ Sa propre clé publique
+ La signature de CA
+ Le certificat de CA

Lorsque A transmettra son certificat à B, B utilisera le clé publique de CA pour vérifier que la signature est valide. La confiance que B a en CA lui permet de croire que A est authentique, et qu'il peut utiliser la clé publique de A en toute confiance pour chiffrer des messages. 

Par la suite, lorsque B voudra chiffrer les messages qu'il envoit à A, il génèrera une clé de session, la chiffrera avec la clé publique de A et lui enverra. A déchiffrera cette clé de session et les deux pourront l'utiliser pour s'envoyer des messages chiffrés.

#### Création et signature de certificats
L'outil `openssl` permet entre autres de créer des paires de clés, des signatures numériques et des demandes de de certificat. Nous verrons ici comment l'utiliser pour effectuer chacune des opérations requises pour déployer des certificats dans une infrastructure MQTT.

##### Certificat de la CA
On veut créer le certificat d'une CA qui sera ensuite utilisé pour valider d'autres certificats. La commande est la suivante:
```
openssl req -new -x509 -days 365 -extensions v3_ca -keyout ca.key -out ca.crt
```
Les options de la commande sont les suivantes:
+ **req**: On veut créer une demande
+ **-new**: Le fichier à créer est pour un nouveau certificat
+ **-x509**: On veut que le fichier créé soit le certificat lui-même
+ **-days**: La dureée de validité du certificat
+ **-extensions**: Ajouter les sections propres aux CA dans le certificat
+ **-keyout**: Le nom de la clé privée qu'on veut générer en même temps
+ **-out**: Le nom du fichier généré

Suite à cette commande vous devrez saisir un mot de passe ("PEM pass phrase") utilisé pour protéger la clé privée de la CA. Ensuite on vous demande d'entrer diverses informations servant à identifier la CA.

##### Certificat de l'hôte
Les étapes sont les suivantes:
+ Créer la clé privée de l'hôte
+ Créer la demande de certificat de l'hôte
+ Signer la demande de l'hôte avec le certificat de la CA
+ Déployer le certificat de l'hôte

Pour créer la clé privée:
```
openssl genrsa -out hote.key 2048
```
Cette clé privée permet de générer la clé publique qui sera incluse dans le certificat de l'hôte. 

La commande pour créer la demande de certificat de l'hôte (_hote.csr_):
```
openssl req -out hote.csr -key hote.key -new
```
Les options de la commande sont les suivantes:
+ **req**: On veut créer une demande
+ **-new**: Le fichier à créer est une nouvelle demande
+ **-key**: La clé privée utilisée pour générer la clé publique à joindre au certificat
+ **-out**: Le nom du fichier généré

> ATTENTION: Lorsque vous saisirez les informations à inclure dans le certificat de l'_agent_, assurez-vous que _Common Name_ a comme valeur le nom DNS utilisé pour vous connecter à l'agent (le nom utilisé après l'option **-h** dans la commande `mosquitto_pub`).

La commande pour signer la demande de l'hôte:
```
openssl x509 -req -in hote.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out hote.crt -days 365
```
Les options de la commande sont les suivantes:
+ **x509**: On veut créer un certificat
+ **-req**: On veut créer le certificat à partir d'une demande
+ **-in**: Le fichier de la demande
+ **-CA**: Le certificat de la CA
+ **-CAkey**: La clé privée de la CA
+ **-CAcreateserial**: Créer un numéro de série pour le certificat (stocké dans un fichier ayant l'extension `.srl`)
+ **-out**: Le nom du fichier du certificat créé
+ **-days**: La dureée de validité du certificat

###### Configuration de l'agent
Lorsque l'hôte est un "broker" MQTT, il faut que le service _mosquitto_ puisse localiser le certificat de la CA, son propre certificat et sa clé privée. Aussi, par convention on recommande que le port utilisé soit 8883 si MQTT utilise TLS. On doit donc avoir les lignes suivantes dans le fichier de configuration `/etc/mosquitto/mosquitto.conf`:
```
listener 8883 0.0.0.0

cafile /etc/mosquitto/ca_certificates/ca.crt
certfile /etc/mosquitto/certs/hote.crt
keyfile /etc/mosquitto/certs/hote.key
require_certificate true
```

###### Configuration d'un client
Le certificat de la CA, du client et sa clé privée doivent également être spécifiées au moment de la connexion à l'agent.

Les options pour `mosquitto_pub` et `mosquitto_sub` sont les suivantes:
+ **-cafile**: le fichier du certificat de la CA
+ **-cert**: le certificat du client
+ **-key**: la clé du client
+ **-p**: le port 

Par exemple:
```
mosquitto_pub -h 10.10.21.13 -t "meteo/serre" -m "17.5" -p 8883 -cafile ca.crt -cert client.crt -key client.key
```

Dans un programme en C utilisant la librairie `libmosquitto-dev`, il faut appeler la fonction [mosquitto_tls_set()](https://mosquitto.org/api/files/mosquitto-h.html#mosquitto_tls_set) avant la connexion à l'agent. Un exemple:
```C
char* ca = "/etc/mosquitto/ca.crt";
char* cert = "/etc/mosquitto/client.crt";
char* key = "/etc/mosquitto/client.key";
mosquitto_tls_set(mosq, ca, NULL, cert, key, NULL); 
```

## Exercice 1
Créez un agent MQTT sur votre hôte sur le réseau du département, et configurez-le pour que les clients doivent s'authentifier (prenez un nom et un mot de passe de votre choix). Ensuite, testez avec les commandes `mosquitto_pub` et `mosquitto_sub`.

## Exercice 2
Créez un certificat pour une nouvelle autorité de certification. Les valeurs doivent être les suivantes:
+ Pays: CA
+ Province: Qc
+ Ville: Montreal
+ Organisation: Informatique Marie-Victorin

## Exercice 3
Créez 2 certificats: 1 pour le client et 1 pour l'agent; attention au champ "Common Name" pour celui de l'agent. Signez chacun avec le certificat de la CA que vous venez de créer.

Ensuite déployez ces certificats sur vos clients et votre agent.

Comment pouvez-vous prouver que la communication est bien chiffrée?

