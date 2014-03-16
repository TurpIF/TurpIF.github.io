---
layout: post
title: Utiliser une Wiimote avec Python
image: "/assets/python-wiimote-icone.png"
categories: [python, wiimote, bluetooth]
---

Il existe déjà des programmes pour pouvoir interfacer les Wiimotes avec son
ordinateur. J'en ai testé quelques-uns et il s'avère que pour utiliser certains
logiciels c'est correct mais si on veut communiquer avec une Wiimote depuis
notre programme ça reste plus compliqué.

Je vais donc montrer comment communiquer avec sa Wiimote. J'utilise le langage
Python pour ça (car je l'adore :) ) mais comme c'est de la communication par
*sockets*, cela peut être fait en n'importe quelle autre langage.

## Amorce de la communication

Avant de commencer à programmer, il faut déjà savoir comment la Wiimote
communique avec la Wii. Celle-ci utilise une communication **bluetooth** via
deux *sockets*. Un pour écrire des ordres à la Wiimote et un autre pour lire
des données.

### Dépendances

Avec Python, j'utiliserais la libraire **PyBluez** pour utiliser le bluetooth.
Le plus facile pour l'installer est d'avoir `pip` et de lancer la commande
suivante :

```sh
sudo pip2 install PyBluez
```

Attention, j'utilise Python 2.7 et je ne sais pas si mon code est compatible
avec la version 3. Donc si vous voulez suivre à la lettre mon code, je vous
conseille d'installer la version de PyBluez pour Python 2. C'est pourquoi j'ai
mis `pip2` au lieu de `pip`.

Sinon il est également possible d'utiliser les utilitaires `pacman` ou `yaourt`
sous Archlinux car je sais que la librairie est dans les répo :

```sh
sudo pacman -S python2-pybluez
# ou
sudo yaourt pybluez
```

Si aucune des solutions ci-dessus n'est ok pour vous, il reste l'installation
manuelle en téléchargeant les sources sur le site officiel de [PyBluez][].
Normalement, il suffit de décompresser l'archive puis de faire un `sudo python2
setup.py install` mais je vous laisse le soin de voir par vous-même.

Il reste encore une autre dépendance : le bluetooth. Soit votre ordinateur est
équipé de base, soit il vous faut un dongle bliuetooth. Il y en a des pas chers
pour à peine 2€.

### Cherchons la Wiimote

On va tester dès maintenant si le bluetooth fonctionne correctement. En ouvrant
le cache arrière de votre Wiimote, vous verrez les batteries et également un
bouton rouge. Celui-ci permet d'activer le mode *découverte* pendant 20
secondes. Normalement les LEDs de la Wiimote clignote pendant ces 20 secondes.
Pendant ce temps, ouvrez un terminal python puis écrivez les lignes suivantes :

```python
>>> import bluetooth
>>> print bluetooth.discover_devices(lookup_name=True)
[...]
```

La première ligne permet d'importer la lib PyBluez. La seconde demande
d'effectuer un scan pour découvrir les périphériques bluetooth environnant. Le
paramètre `lookup_name` indique qu'on souhaite obtenir les noms des
périphériques trouvés. On a alors en sortie une liste de couples `(address,
name)` pour chaque périphérique.

Normalement, dans la liste des périphériques, vous devriez voir un tuple avec
comme nom `Nintendo RVL-CNT-01` ou `Nintendo RVL-CNT-01-TR`.

Si vous ne voyez rien, essayez d'allumer le bluetooth de votre téléphone et
retentez l'expérience. Si vous voyez toujours rien alors le bluetooth a pas
l'air de fonctionner. Si vous voyez votre téléphone, alors réessayez avec la
Wiimote en vérifiant bien que les LEDs clignotent pendant l'opération.

L'adresse du tuple correspond à l'adresse où on doit connecter les sockets.
Donc garder la en mémoire (ou en variable). Voici une petite fonction qui
retourne la liste des adresses des Wiimotes présentes :

```python
def discover():
  devices = bluetooth.discover_devices(lookup_names=True)
  return [addr for (addr, name) in devices
      if name == 'Nintendo RVL-CNT-01' or name == 'Nintendo RVL-CNT-01-TR']
```

### Connexion au périphérique

Maintenant que nous avons l'adresse de notre Wiimote, nous pouvons nous
connecter. Pour cela, comme dit plus haut, on a besoin de deux sockets
bluetooth. Un pour l'écriture et un pour la lecture. Plus précisément, ces
sockets utilisent la couche L2CAP du bluetooth. Dans cette couche, pour se
connecter il faut l'adresse du périphérique ainsi que le numéro de port.

D'après les spécifications des Wiimotes, le port du socket de lecture est
`0x11` et celui d'écriture est `0x13`.

Voici un code d'exemple se connectant à la Wiimote. Je vous conseille de
regarder la [documentation de PyBluez][PyBluezDoc] afin de savoir ce que font
exactement les fonctions que j'utilise (bien que ça reste assez transparent).

```python
# Import à faire
from bluetooth import BluetoothSocket, L2CAP

# On récupère la liste des Wiimotes
addrs = discover()
if addrs:
  # Création et connexion des sockets
  input_sock = BluetoothSocket(L2CAP)
  output_sock = BluetoothSocket(L2CAP)

  addr = addrs[0]
  input_socket.connect((addr, 0x11))
  output_socket.connect((addr, 0x13))

  # On fait quelque chose avec les sockets

  # Fermeture des connexions
  input_socket.close()
  output_socket.close()
```

### Initialiser la bête

Bien que nous ayons pu se connecter à la Wiimote, il faut avant toutes choses
initialiser la communication et indiquer ce qu'on veut faire de la Wiimote. En
effet, c'est conçu pour fonctionner de différentes manières en fonction de ce
qu'on veut faire des données, de la précision qu'on souhaite et de quelles
données on veut.

Je ne vais pas être exhaustif sur la configuration de tous les modes de la
Wiimote. Il y en a beaucoup et je n'ai pas eu l'occasion de tous les tester. Je
vous invite à lire [cette page][Wiibrew] pour obtenir de plus amples
informations.

La séquence d'initialisation à envoyer via le socket d'écriture est sous la forme :

```
0xa2 0x12 0x00 0xMM
```

`MM` correspond au mode d'acquisition des données pour la Wiimote :

- 0x30 : Seulement les boutons (2 bytes)
- 0x31 : Boutons (2 bytes) et accéléromètre (3 bytes)
- 0x32 : Boutons (2 bytes) et manette d'extension connecté à la wiimote (8
bytes)
- 0x33 : Boutons (2 bytes), accéléromètre (3 bytes) et infrarouge (12 bytes)
- 0x34 : Boutons (2 bytes) et extension (19 bytes)
- 0x35 : Boutons (2 bytes), accéléromètre (3 bytes) et extension (16 bytes)
- 0x36 : Boutons (2 bytes), infrarouge (10 bytes) et extension (9 bytes)
- 0x37 : Boutons (2 bytes), accéléromètre (3 bytes), infrarouge (10 bytes) et
extension (6 bytes)
- 0x3d : extension (21 bytes)

Il existe un dernier mode avec 0x3e ou 0x3f qui permet d'avoir jusqu'à 36 bytes
d'informations sur le capteur infrarouge. Je n'en parlerais pas.

Je vais présenter l'utilisation du mode 0x33 qui permet d'avoir les boutons,
l'accéléromètre et le capteur infrarouge. La séquence à envoyer est donc :

```
0xa2 0x12 0x00 0x33
```

La fonction pour écrire sur un socket bluetooth est `send` et prend en
paramètre une string de données correspond à un tableau de `char` en C. Voici
le code pour transformer une liste d'octet en chaine de données et l'envoyer :

```python
def send(*data):
  data_str = ''.join(map(chr, data))
  input_sock.send(data_str)

send(0xa2, 0x12, 0x00, 0x33)
```

### Configuration du capteur IR

Pour l'utiliser, le capteur infrarouge doit être configuré. Cela permet d'avoir
un contrôle plus fin sur les données lues. Je vous conseille fortement à lire
la section sur la configuration de l'IR sur [cette page][Wiibrew] (déjà dit
plus haut) ainsi que la partie sur l'écriture des registres dans L'EEPROM de la
Wiimote.

Les étapes à suivre pour la configuration de l'IR sont les suivantes :

- Activation de la caméra IR : Envoi de `0xa2 0x13 0x04` et `0xa2 0x1a 0x04`
- Écriture de `0x08` dans le registre `0xb00030`
- Écrire le premier bloc de configuration de la sensibilité dans le registre
  `0xb00000`
- Écrire le second bloc de configuration de la sensibilité dans le registre
  `0xb0001a`
- Écrire le mode d'acquisition dans le registre `0xb00033` (1 si 10 bytes pour
  l'IR, 3 pour 12 bytes, 5 pour 36 bytes)
- Écrire à nouveau `0x08` dans le registre `0xb00030`

Voici la configuration de la sensibilité :

- Bloc 1 : `00 00 00 00 00 00 90 00 41`
- Bloc 2 : `40 00`

J'ai repris la configuration de la sensibilité sur [Wiibrew][].

Pour écrire dans la mémoire on envoie la séquence :

```
A2 16 04 AA AA AA SS DD...
```

Les `AA` représente l'adresse du registre à configurer. `SS` représente le
nombre de byte à écrire et peut aller jusqu'à 16. Les `DD` sont les données à
écrire et ont la même taille que `SS`.

Il est recommandé de faire une pause de 50ms entre chaque envoie pour s'assurer
que la Wiimote est correctement configuré.

Voici un code récapitulatif qui s'assure de la configuration :

```python
def init_ir(address, *data):
  a0 = (address & 0x0000FF) >> 000
  a1 = (address & 0x00FF00) >> 010
  a2 = (address & 0xFF0000) >> 020
  send(0xa2, 0x16, 0x04, a2, a1, a0, len(data), *data)

import time

time.sleep(0.050); send(0xa2, 0x13, 0x04)
time.sleep(0.050); send(0xa2, 0x1a, 0x04)
time.sleep(0.050); init_ir(0xb00030, 0x08)
time.sleep(0.050); init_ir(0xb00000, 0, 0, 0, 0, 0, 0, 0x90, 0, 0x41)
time.sleep(0.050); init_ir(0xb0001a, 0x40, 0x00)
time.sleep(0.050); init_ir(0xb00033, 0x03)
time.sleep(0.050); init_ir(0xb00030, 0x08)
```


## Tchatcher avec la Wiimote
Après l'initialisation de la Wiimote et il possible de communiquer avec et de
lire les données acquises. Voici Une liste d'actions qu'on peut faire avec
elle.

### La faire vibrer
La chose à plus simple à faire pour tester si la connexion s'est bien passé est
de faire vibrer sa Wiimote. Pour cela il suffit d'envoyer la séquence suivante
avec comme dernier byte `0x01` si on veut enclencher et `0x00` si on veut
l'éteindre.

```
A2 15 BB
```

Il est alors très facile de créer une fonction `rumble` :

```python
def rumble(duration):
  send(0xa2, 0x15, 0x01)
  time.sleep(duration)
  send(0xa2, 0x15, 0x00)
```

### L'allumer de mille feux

On peut contrôler l'état des 4 LEDs bleus en bas de la Wiimote. Ceux-ci se configurent via la commande suivante :

```
A2 11 LL
```

`LL` correspond à l'état des LEDs. Le bit 4 contrôle la première (de gauche à
droite) et le 7ème bit la dernière. Si ils sont à 1, la LED est allumé. Sinon
elle est éteinte.

Cela reste assez simple et permet de montrer l'état de la batterie par exemple.
Je vous laisse chercher ça par vous-même.

Voici une fonction pour gérer l'allumage des LEDs :

```python
def set_leds(str_state='0000'):
  send(0xa2, 0x11, int(str_state, 2) << 4)

def turn_off_led():
  set_leds('0000')

def turn_on_led():
  set_leds('1111')
```

### Lire des données

Pour l'instant la Wiimote ne nous servait pas à grand-chose à par allumer des
petites LEDs et faire vibrer la table. Le principal intérêt de la Wiimote,
c'est ses capteurs intégrés : les boutons, l'accéléromètre et la caméra
infrarouge.

Les données lisibles sont choisies lors de l'initialisation de la Wiimote. Avec
le mode choisi plus haut, les trois capteurs mentionnés ci-dessus sont
accessibles.

A chaque fois que la Wiimote a une nouvelle information à dire, elle émet un
paquet à lire depuis l'ordinateur. Il est également possible de faire en sorte
que la Wiimote émette en permanence des données même si il n'y a pas de
changement. Cela se fait dans la phase d'initialisation.

La taille à lire dépends également du mode choisi. Ici, un paquet fait 19
bytes. On utilise alors la fonction `recv` qui retourne une string que l'on
peut convertir en liste d'octet comme ci-dessous :

```python
def read():
  data = map(ord, output_socket.recv(19)
  # Vérification de l'intégrité du paquet
  if len(data) == 19 and data[0] == 0xa1 and data[1] == 0x33:
    return data
  return None
```

Le contenu est totalement dépendant du mode sélectionné et la suite n'est
valable uniquement dans celui choisi dans l'article. Cela permet toutefois
d'avoir un ordre d'idée de ce qu'il faut faire dans les autres modes.

#### Traiter ses boutons

Je rappelle que les paquets obtenus sont de la forme :

```
A1 33 BB BB AA AA AA II II II II II II II II II II II II
```

- `BB` : informations sur les boutons
- `AA` : mesure de l'accéléromètre
- `II` : position des points infrarouges.

Il y a donc 2 bytes d'informations pour les boutons. Sur ces 2 bytes, chaque
bit est assigné à un bouton et vaut 1 si le bouton est enfoncé, 0 sinon.

Voici la liste des masques binaires pour chaque bouton :

- `0x0001` : Haut
- `0x0002` : Droite
- `0x0004` : Bas
- `0x0008` : Gauche
- `0x0200` : Un
- `0x0100` : Deux
- `0x0800` : A
- `0x0400` : B
- `0x0010` : Plus
- `0x1000` : Moins
- `0x8000` : Home

Voici un code d'exemple qui affiche la liste des boutons appuyés :

```python
def button_pressed(data, mask):
  # On récupère les 2 bytes des boutons depuis le paquet
  bbbb = data[2] << 010 & data[3]
  return bool(bbbb & mask)

button_map = {
  0x0001: 'Haut',
  0x0002: 'Droite',
  0x0004: 'Bas',
  0x0008: 'Gauche',
  0x0010: 'Plus',
  0x0100: 'Deux',
  0x0200: 'Un',
  0x0400: 'B',
  0x0800: 'A',
  0x1000: 'Moins',
  0x8000: 'Home'
}

data = read()
pressed_buttons = [name for mask, name in button_map.iteritems()
  if button_pressed(data, mask)]
print ' '.join(pressed_button)
```

#### Connaître son accélération

Les bytes concernant l'accéléromètre sont les 4, 5, 6 (en commençant par 0).
Chacun des bytes correspondent respectivement à l'accélération sur X, Y, Z
selon le schéma ci-dessous.

<center>
![Axes de la Wiimote](/assets/python-wiimote-axes.png)
</center>

Le zéro de l'accélération est approximativement autour de `0x80`. Il est
possible d'avoir un peu plus de précision en rajoutant 2 bits sur l'axe X et 1
bit sur les axes Y et Z. En effet, quelques bits sur les bytes des boutons ne
sont pas utilisés et contiennent une partie de la mesure de l'accéléromètre.

La mesure est codée sur 10 bits sur les 3 axes. Pour Y et Z on impose que le
LSB soit à 0.

On a donc :

```python
data = read()
x_msb = data[4]
y_msb = data[5]
z_msb = data[6]
x_lsb = (data[2] & 0b01100000) >> 5
y_lsb = (data[3] & 1 << 5) >> 4
z_lsb = (data[3] & 1 << 6) >> 5

x = x_msb << 2 | x_lsb
y = y_msb << 2 | y_lsb
z = z_msb << 2 | z_lsb
```

La valeur neutre `0x80` est que purement théorique et peut varier en fonction
des conditions externes. Il est donc utile de faire une calibration du capteur
avant l'utilisation. Je reviendrais sur ça lors d'un prochain article.

#### Suivre des points par IR

La Wiimote possède une caméra infrarouge et fait un post-traitement permettant
d'identifier des points émettant de l'infrarouge. Il n'est pas possible de
récupérer l'image de la caméra pour prendre une photo par exemple.

Il peut y avoir 4 points qui sont suivis simultanément. Lorsqu'un point est
tracé, la Wiimote s'arrange pour l'identifier et toujours le placer à la même
position. Ainsi si deux points sont trouvés et que le premier disparait, alors
le deuxième reste en deuxième position. Le suivi n'a donc pas besoin d'être
fait du côté de l'ordinateur.

Chaque point est représenté par sa position *X, Y* codé sur 10 bits. Cela
signifie que les valeurs des positions vont de 0 à 1023 pour X et de 0 à 767
pour Y. Lorsqu'un slot de point est vide (*cad* qu'il n'y a pas de point
détecté), alors les bytes du point ont comme valeur `0xFF` ce qui peut se
traduire par l'obtention d'une position X, Y à 1024, 1024.

Dans le mode choisi plus haut, il y a 12 bytes pour l'infrarouge, soit 3 bytes
par point. On retrouve dans ces 3 bytes, la taille de la tache infrarouge
détecté codée sur 4bits (0 à 15).

On lit les bytes de la façon suivante :

```python
def read_ir(data):
  x_lsb = data[0]
  y_lsb = data[1]
  x_msb = (data[2] & 0b00110000) << 4
  y_msb = data[2] & 0b11000000 << 2
  s = data[2] & 0b00001111
  x = x_msb | x_lsb
  y = y_msb | y_lsb
  return x, y, z

data = read()
x1, y1, s1 = read_ir(data[7:10])
x2, y2, s2 = read_ir(data[10:13])
x3, y3, s3 = read_ir(data[13:15])
x4, y4, s4 = read_ir(data[15:19])
```

Bien sur, cela reste des données bruts et c'est au développeur de filtrer le
signal afin d'avoir quelque chose suivant les attentes de son programme. Je
reviendrais sur l'utilisation de l'infrarouge dans un prochain article et
parlerais plus en détails des filtres que j'utilise.

#### Pour en savoir plus

La Wiimote peut faire encore pas mal de choses. Par exemple, il est possible de
communiquer avec un périphérique d'extension branché à la Wiimote comme un
Nunchuk. Il est également possible de jouer du son sur le Wiimote ou encore
d'obtenir des informations sur l'état des batteries.

Pour voir les codes plus en détails je vous invite à voir mon répo
[PyWiimote][].

[PyBluez]: https://code.google.com/p/pybluez/
[PyBluezDoc]: http://pybluez.googlecode.com/svn/www/docs-0.7/index.html
[Wiibrew]: http://wiibrew.org/wiki/Wiimote
[PyWiimote]: https://github.com/TurpIF/PyWiimote
