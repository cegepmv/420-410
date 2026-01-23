+++
type = "chapter"
pre = "1. "
title = "Préparation"
weight = 1
+++ 


## Activer le service ssh
Le service SSH doit s'exécuter sur le Pi pour que la connexion de VSCode soit possible. Aussi, il est utilise de faire en sorte qu'il démarre automatiquement chaque fois que la Pi est allumé. Pour ce faire, lancez les deux commandes suivantes:
```
sudo systemctl enable ssh
sudo systemctl start ssh
```

## Changer le nom d'hôte
Il est possible de se connecter sur le Pi en utilisant son adresse IP, mais il est souvent plus pratique d'utiliser un nom pour y référer. Pour y arriver il faut modifier deux fichiers.

Dans `/etc/hostname`, écrivez le nom que vous voulez lui donner:
```
bork
```

Dans `/etc/hosts`, remplacez "raspberrypi" par le nouveau nom:
```
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1               bork       
```
Redémarrez ensuite votre Pi.

À cette étape, il devrait être accessible sur le réseau par son nom suivi du suffixe `.local`. Testez avec la comande _ping_:
```
ping bork.local
```

## Connexion par VSCode

Dans le coin inférieur gauche de VSCode, cliquez sur l'élément suivant:

![vscodeconnect](/420-410/images/vscodeconnect.png)

Choisissez ensuite l'option `Connect to Host...`, puis entrez la commande de connexion SSH (vous pouvez ici utiliser le nom de votre Pi ou son adresse IP):

![cnstring](/420-410/images/cnstring.png)

Répondez ensuite aux questions (mot de passe etc.). Si tout se passe bien, vous devriez ensuite être connecté.