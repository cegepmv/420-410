+++
title = 'Écrans LCD'
date = 2025-04-24T20:41:58-04:00
draft = false
weight = 71
+++

L'écran *LCD 0802* du kit *KS0522* Est compatible avec plusieurs modules python pour le RaspberryPi. Ici nous utiliserons **Adafruit CharLCD**. Une référence est disponible ici: https://docs.circuitpython.org/projects/charlcd/en/latest/index.html.

Tout d'abord il faut installer le paquet logiciel correspondant avec la commande suivante sur le Pi:

```bash
pip install adafruit-circuitpython-charlcd
```

Il s'agit ensuite d'appeler le constructeur de la classe `Character_LCD` en lui passant (entre autres) les identifiants GPIO connectés aux broches de l'écran LCD.

## Pinout LCD 0802

Les broches du modules *LCD 0802* sont les suivantes:

![pinoutLCD](/420-410/images/pinoutLCD.png?width=500px)

Les connexions vers des modules externes sont en rouge. Par exemple, la broche 13 (DB6) doit être connectée à un port GPIO (n'importe quel) du Pi.

Aussi, une manière simple de contrôler le contraste de l'écran est de brancher la broche 3 sur le signal d'un potentiomètre.

Les broches 7, 8, 9 et 10 ne servent pas ici car on utilise une communication sur 4 bits et non 8.

## Exemple 
Dans l'exemple suivant on affiche le texte 'hello' une seconde sur la première ligne, puis le texte 'bye' sur la deuxième:

```python
import board
import digitalio
import adafruit_character_lcd.character_lcd as CharLCD
from time import sleep

## board.D1 = GPIO 1, board.D2 = GPIO 2, etc.
## Mettez ici les valeurs qui correspondent à vos connexions
RS = digitalio.DigitalInOut(board.D1)
E = digitalio.DigitalInOut(board.D1)
DB7 = digitalio.DigitalInOut(board.D1)
DB6 = digitalio.DigitalInOut(board.D1)
DB5 = digitalio.DigitalInOut(board.D1)
DB4 = digitalio.DigitalInOut(board.D1)

cols = 8
rows = 2

lcd = CharLCD.Character_LCD(RS,E,DB4,DB5,DB6,DB7,cols,rows)

lcd.message = "hello"
sleep(1)
lcd.cursor_position(5,1)
lcd.message = "bye"
sleep(1)
lcd.clear()
```

## Méthodes et propriétés utiles
#### `message`
Affiche à l'écran la chaîne de caractères à la position du curseur (définie par la méthode `cursor_position()` ou 0,0 par défaut). 

Si la chaîne de caractère contient des valeurs hexadécimales, un caractère correspondant au nombre sera affiché; par exemple, `lcd.message = "\c0"`. Ce nombre correspond en fait à une adresse en mémoire où ce caractère est stocké. Voir la Table 4 du document de référence https://www.sparkfun.com/datasheets/LCD/HD44780.pdf.

#### `cursor_position(int,int)`
Déplace le curseur à la position passée (colonne, rangée)

#### `move_left()`, `move_right()`
Décale vers la gauche / la droite le contenu de l'écran. 

#### `create_char(int,Sequence)`
Crée un caractère et le stocke à l'adresse spécifiée (on peut ensuite l'afficher avec `message`). L'adresse est le premier paramètre et peut avoir une valeur 0 à 7. Le caractère est une matrice de 5x8 bits représentée dans une liste.

Dans l'exemple suivant, on crée un caractère rectangulaire, on le stocke à l'adresse `1` et on l'affiche:

```python
boite = [
    0b11111,
    0b10001,
    0b10001,
    0b10001,
    0b10001,
    0b10001,
    0b10001,
    0b11111
]
lcd.create_char(1,boite)
lcd.message = "\x01"
```

## Exercice
1. Affichez l'heure au format HH:MM:SS, en mettant à jour l'affichage chaque seconde.
<!--
{{% expand "Solution 1." %}}
```python
(...)
from datetime import datetime
(...)
try:
    while True:
        t = datetime.now()
        hms = t.strftime("%H:%M:%S")
        lcd.message = hms
        sleep(1)
except KeyboardInterrupt:
    lcd.clear() 
```
{{% /expand %}}
-->
2. Créez un caractère "smiley" et affichez-le.
