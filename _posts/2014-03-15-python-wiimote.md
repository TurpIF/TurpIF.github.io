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
setup.py install` mais je vous laisse le soin de voir par vous même.

Il reste encore une autre dépendance : le bluetooth. Soit votre ordinateur est
équipé de base, soit il vous faut un dongle bliuetooth. Il y en a des pas cher
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

Si vous ne voyez rien, essayez d'allumer la bluetooth de votre téléphone et
retentez l'expérience. Si vous voyez toujours rien alors la bluetooth a pas
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

[PyBluez]: https://code.google.com/p/pybluez/
